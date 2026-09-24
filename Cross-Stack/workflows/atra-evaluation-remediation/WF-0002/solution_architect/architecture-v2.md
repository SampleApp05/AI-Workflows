---
artifact:
  id: ARC-0002
  type: ARCHITECTURE
  version: 2
  status: DRAFT
workflow:
  id: WF-0002
  project: Atra
  technology: Cross-Stack
  feature: atra-evaluation-remediation
  mode: delivery
  contract_version: "1.2"
lineage:
  parents:
    - ARC-0002@v1
    - DEF-0002@v1
    - DEC-0001@v1
  references:
    - path: solution_architect/architecture-v1.md
    - path: problem_analyst/definition-v1.md
    - path: workflow_orchestrator/architecture-decisions-v1.md
    - path: intake_analyst/handoff-v1.md
    - path: intake_analyst/handoff-summary-v1.md
    - path: workflow_orchestrator/orchestration-plan-v1.md
    - workflow_id: WF-0001
      artifact: WRUN-0001@v3
      path: Cross-Stack/workflows/phase-foundation-evaluation/WF-0001/workflow_orchestrator/workflow-run-report-v3.md
    - workflow_id: WF-0001
      artifact: TEST-0001@v1
      path: Cross-Stack/workflows/phase-foundation-evaluation/WF-0001/test_engineer/test-report-v1.md
    - product_repository: Atra-Services
      branch: main
      commit: be51242593fe6643aff11c491d4f8a6c0835aed5
      state: clean at architecture inspection
ownership:
  created_by: solution_architect
  performed_by: solution_architect
  recorded_by: solution_architect
  engine: codex
  target_id: codex-native
---

# Architecture — Atra evaluation remediation, revised direction

## Revision purpose and authority

This bounded revision supersedes the technical direction in `ARC-0002@v1` for the same fixed six-finding delivery scope. It incorporates the binding human choices recorded in `DEC-0001@v1`; it does not alter `HAND-0002@v1`, `DEF-0002@v1`, the approved delivery mode, or the plan/execution gates.

The inspected product repository remains `Atra-Services` only. At architecture inspection, it resolved to `main` / `be51242593fe6643aff11c491d4f8a6c0835aed5` with no reported working-tree changes, matching the evaluated baseline. This evidence does not replace the later execution-stage verification of baseline, branch, remote, and PR authority.

The direction remains limited to:

1. per-process market-stream unsubscribe accounting;
2. consumer-visible market REST freshness;
3. session-derived authority for sensitive wallet, role, recovery, and revoke actions;
4. atomic nonce single use;
5. durable authenticating-wallet session attribution; and
6. database-boundary OWNER, RECOVERY, and duplicate-role invariants.

It excludes package-manager work, audit pagination, multi-instance market behavior, iOS/client/provider/portfolio/Phase 2.5 work, deployment, merge, release, production-data operations, and new product capabilities.

## Existing boundaries retained

| Concern | Observed ownership | Directional consequence |
| --- | --- | --- |
| Stream subscriptions | `MarketStreamManager` owns per-process socket/symbol membership and calls the Binance adapter; the gateway owns transport. | Fix the manager membership transition; do not change provider/gateway ownership or imply multi-instance aggregation. |
| Price retrieval | `PriceService` owns cache and stale-while-revalidate policy; `/prices` serializes `MarketTicker[]`. | Return a freshness-aware application result from the service and expose it directly at the REST boundary. |
| Authenticated action trust | Middleware validates JWT/session/account and attaches `req.auth`; named controllers also accept body authority fields. | Use the middleware context as the sole actor source and leave client data as target/proof input only. |
| Session identity | Session creation receives a wallet, but session persistence and claims do not retain it; refresh/middleware select an arbitrary role row. | Persist and read the authenticating wallet from the session record. |
| Nonce use | In-scope flows read an unused challenge and separately mark it used. | Make conditional database consumption a shared primitive, not a caller convention. |
| Role integrity | PostgreSQL/Drizzle owns roles and accounts; application checks/transactions exist but no relevant schema constraints or migrations were observed. | Add database boundary enforcement and verify it through a disposable PostgreSQL environment. |

## Binding human decisions incorporated

`DEC-0001@v1` resolves the material choices from the prior architecture version:

- **Freshness interface:** no existing consumers exist; `/prices` may change directly to the freshness-aware contract. No versioned, parallel, negotiated, or compatibility route is required.
- **Incompatible role/owner data:** if an authorized preflight finds duplicate role rows, multiple OWNER or RECOVERY rows, or a canonical-owner mismatch, stop and report the evidence. This run may not repair production data or otherwise accept incompatible existing state.
- **Session lifecycle:** refresh preserves the original authenticating wallet; a user may revoke their own session; recovery, ownership transfer, or loss of an authorizing role revokes affected sessions.
- **Verification environment:** a disposable non-production PostgreSQL environment is authorized only to apply workflow migrations and execute competing-write tests. It does not authorize a production connection, data operation, deployment, or persistent environment change.

These are recorded human decisions, not architecture recommendations. No unresolved material choice remains from `ARC-0002@v1` for Requirements to specify the selected bounded behavior.

## Recommended technical direction

### 1. Membership-gated market unsubscription

**Recommendation.** `SymbolEntry.subscribers` is the source of truth for subscription ownership. Remove a member—and alter any retained reference count—only when that socket is actually in the symbol's subscriber set. An absent symbol, an unsolicited unsubscribe, and a repeated unsubscribe are no-ops for both manager registries and upstream provider interest. Disconnect cleanup uses that same guarded membership transition.

The upstream Binance unsubscribe happens only when the final real member is removed. A retained `refCount` must be updated under precisely the successful membership transition or derived from membership size. This remedies the approved per-process accounting defect without modifying gateway protocol, provider ownership, or the excluded multi-instance model.

### 2. Direct `/prices` freshness-aware response

**Binding decision applied.** `/prices` may directly replace its current bare `MarketTicker[]` response with a per-result representation containing the ticker and a cache freshness classification. `PriceService` should produce the application-owned result from its existing cache decision: `fresh` for a current entry and `stale` for a usable entry being refreshed. An unavailable cache-miss/upstream result remains an error/unavailable outcome, distinguishable from a stale successful result through the endpoint's response contract.

Per-result freshness represents mixed multi-symbol responses accurately and keeps provider-normalized `MarketTicker` separate from cache policy. No compatibility route, response negotiation, or legacy body shape is required because `DEC-0001@v1` records that no consumers exist.

### 3. Trusted session actor context for sensitive actions

**Recommendation.** Validated `req.auth` is the sole authority source for actor `accountId`, `walletId`, `sessionId`, and `chainId` on wallet, role, recovery, and revoke actions. Controllers pass that trusted actor context to the established service boundary. Client-supplied fields remain acceptable only for operation targets or proof material, such as a new/target address, nonce, signature, or target session identifier; they cannot select the account or acting wallet.

Revoke must be mounted behind the same authentication boundary. A user may revoke only their own session, as decided in `DEC-0001@v1`; body account identity cannot broaden that authority. A body/session authority mismatch is rejected rather than ignored as a compatibility fallback. Audit attribution receives the authenticated session wallet, not an account ID or arbitrarily selected role row.

Recovery and role behavior use the authenticated session's actor context and preserve the existing domain proof/target validation boundary. Recovery, ownership transfer, and loss of an authorizing role revoke the sessions affected by that event, as decided; this makes session invalidation a deliberate domain consequence rather than an accidental side effect.

### 4. Atomic compare-and-consume nonce operation

**Recommendation.** Replace each approved find-then-`markUsed` path with one shared database-backed consume operation that reports whether it successfully consumed the challenge. Its predicate evaluates challenge identity/value, wallet, purpose, unconsumed state, and expiry at the time of consumption. A competing verification can therefore never obtain a second successful consume.

Signature verification can occur before the conditional consume, but an earlier lookup is never authoritative. When a nonce gates durable provisioning, linking, role change, or recovery, conditional consume and the related domain mutation occur in the same transaction. Two-nonce role operations require all required conditional consumes and the role mutation to succeed or fail together. This avoids both replay success and an unexplained partially completed privileged outcome.

### 5. Durable session wallet and selected lifecycle

**Binding decision applied.** Persist the successful signing wallet on every session with a relational wallet reference. Session creation writes it; middleware reads it after session validation; refresh creates the replacement session with that same wallet; and audit/revocation uses it for actor attribution. No middleware or refresh path selects a first `AUTH`/OWNER row to infer session identity.

Roles remain evaluated for the persisted session wallet. Recovery, ownership transfer, or loss of a role that authorizes the session revokes affected sessions. A user may revoke only their own session. These lifecycle rules make multi-wallet attribution, rotation, revocation, and audit behavior consistent with the approved session-derived authority outcome.

### 6. Database-owned role invariants and safe migration disposition

**Recommendation.** Enforce the three separate role guarantees at PostgreSQL's boundary:

1. unique `(account_id, wallet_id, role)` associations prohibit duplicate grants;
2. partial unique OWNER and RECOVERY constraints/indexes per account prohibit a second concurrent assignment; and
3. a database-enforced exact-owner/canonical-owner relationship ensures each account has one OWNER and, while `accounts.owner_wallet_id` remains, that it matches the OWNER association.

The first two controls establish only "at most one." In the observed split account/role model, an exact-one relationship needs a deferred PostgreSQL constraint trigger or structurally equivalent relational mechanism checked at transaction commit. Deferral allows account provisioning, owner transfer, and recovery to move through valid atomic transactions without an invalid intermediate state. Application transactions remain necessary domain orchestration; database constraints are the final protection against alternate paths and competing writes.

The human-approved migration disposition is strict: preflight first. If evidence shows incompatible role/owner data, the workflow stops and reports it. No production repair, deletion, backfill, or acceptance of incompatible state belongs to this run. The observed Drizzle PostgreSQL migration path is appropriate for workflow schema changes, while the disposable PostgreSQL environment is limited to applying those migrations and verifying them.

## Cross-boundary constraints and evidence

| Boundary | Selected behavior | Evidence constraint |
| --- | --- | --- |
| Price service → REST | Directly expose per-result ticker plus `fresh`/`stale` state; unavailable is a distinct failure outcome. | Test fresh, stale-with-background-refresh, and unavailable paths, including mixed result states where supported. |
| Gateway → stream manager → provider | Membership change controls upstream interest; non-member removal cannot reduce it. | Verify one subscriber retains updates after another/non-subscriber unsubscribes. No multi-instance claim. |
| JWT/session → controllers/services | Session actor context determines authority; body identifiers cannot override it. | Verify absent/invalid session, body/session mismatch, and self-only revoke behavior. |
| Session → refresh/recovery/roles/audit | Persisted session wallet survives refresh; selected security events revoke affected sessions. | Verify multi-wallet attribution and affected-session revocation for the three selected events. |
| Auth service → PostgreSQL | Conditional consume and role constraints are durable. | Use the authorized disposable PostgreSQL environment for migration and competing nonce/role writes; mocks alone are insufficient. |
| Schema → existing state | Incompatible current state stops with evidence. | Do not connect to or modify production data in this workflow. |

## Risks and residual limits

1. **Migration incompatibility remains a blocker, not an implementation fallback.** Constraints may expose duplicate/mismatched existing data. The decided response is to stop and report evidence; production repair is out of scope.
2. **Authorization regressions remain material.** Middleware mounting and actor propagation must be complete, especially revoke and audit paths. A partial body-field removal is insufficient.
3. **Database concurrency claims require real evidence.** The approved disposable environment permits this evidence, but successful mocked tests alone cannot establish it.
4. **Direct API change is intentional.** There are no known consumers, but Requirements/Test must still define and verify the selected endpoint shape and errors; this is no longer a compatibility blocker.
5. **Scope is unchanged.** Multi-instance aggregation, client behavior, provider integration, audit pagination, Phase 2.5, and operational release concerns remain excluded even if observed during later work.

## Consequences for subsequent stages

Requirements can now specify a direct freshness-aware `/prices` contract, trusted actor context, self-only session revoke, selected session invalidation events, persistent signing-wallet attribution, atomic nonce behavior, and the database/migration evidence criteria. The plan's later Decomposition, Routing, execution gate, implementation, Test, independent Review, and PR requirements still apply. This Architecture does not create requirements, implementation tasks, routes, code changes, a migration, worker selection, or an execution approval.

## Self-check and requested validation

`ARC-0002@v2` preserves the exact six-outcome, Contract v1.2 delivery boundary; has explicit lineage to `ARC-0002@v1`, `DEF-0002@v1`, and approved `DEC-0001@v1`; and carries each binding human decision into a bounded technical direction. It keeps human decisions distinct from recommendations, retains all exclusions, and introduces no product edit, requirement, task, route, migration, worker assignment, or gate approval. **READY_FOR_REQUIREMENTS is requested for orchestrator validation; this Architecture does not validate itself or advance the workflow.**
