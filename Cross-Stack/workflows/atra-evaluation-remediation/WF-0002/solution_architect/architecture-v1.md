---
artifact:
  id: ARC-0002
  type: ARCHITECTURE
  version: 1
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
    - DEF-0002@v1
  references:
    - path: problem_analyst/definition-v1.md
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

# Architecture — Atra evaluation remediation

## Scope, authority, and evidence posture

This is the technical-direction stage for the fixed six-finding delivery scope in `DEF-0002@v1`. `HAND-0002@v1` is the delivery-scope authority and `OPLAN-0002@v1` is the approved plan authority. The findings in `WRUN-0001@v3` and `TEST-0001@v1` are baseline evidence, not a pre-approved design.

The inspected product repository is `Atra-Services` only. At this assessment it resolved to `main` / `be51242593fe6643aff11c491d4f8a6c0835aed5` with no reported working-tree changes, the same commit documented by the evaluation. This observation does not substitute for the later execution-stage baseline, branch, remote, and PR-authority checks.

The direction below is confined to:

1. subscription accounting for one market-service process;
2. consumer-visible freshness for market REST results;
3. session-derived authority for the named sensitive Auth Service actions;
4. atomic nonce consumption;
5. durable session-to-authenticating-wallet attribution; and
6. database-boundary wallet-role invariants.

It does not add package-management work, audit pagination, multi-instance market behavior, iOS/client/provider/portfolio/Phase 2.5 work, deployment, merge, release, production-data operations, or a new product capability.

## Existing architecture and affected boundaries

| Concern | Observed boundary | Architectural implication |
| --- | --- | --- |
| Market streaming | `MarketStreamManager` owns per-process symbol and socket registries, while `WebSocketGateway` supplies transport and the Binance adapter owns upstream subscription calls. | Correctness belongs in the manager's membership transition, without changing provider or transport ownership. The run makes no multi-instance claim. |
| Market snapshots | `PriceService` owns cache policy; `/prices` in the REST router serializes `MarketTicker[]`. The cache tracks `updatedAt`, but the public ticker model does not. | Freshness must cross the PriceService-to-REST consumer boundary deliberately; an internal stale-while-revalidate branch alone is insufficient. |
| Authenticated actions | Middleware verifies JWT/session/account and creates `req.auth`; wallet, role, recovery, and revoke controllers currently also accept authority identifiers from request bodies. | The middleware context is the trusted actor boundary. Controllers must distinguish trusted actor identity from client-supplied operation inputs. |
| Sessions and tokens | Session creation receives a wallet, but `sessions` and access-token claims retain account/session/chain data only; middleware and refresh select an arbitrary role row for a wallet. | Session ownership needs durable wallet attribution at the session persistence boundary, then downstream code should read that attribution rather than reselect one. |
| Nonces | Multiple service paths locate an unused challenge and later update `usedAt`; `NonceService.markUsed` updates by ID with no unused/expiry predicate. | Single-use is a repository/database operation, not a convention enforced separately by each caller. |
| Roles and persistence | PostgreSQL/Drizzle owns `account_wallet_roles`, `accounts`, sessions, and nonce tables. Role services have application checks and transactions, but the schema has no observed role uniqueness/partial uniqueness constraints; migrations are presently only `.gitkeep`. | Durability and concurrent correctness require schema/migration design, while existing application transactions remain the domain-operation boundary. |

## Architectural goals

- Preserve the established composition: market adapters, cache/service/REST layers; Auth middleware, controllers, services, repositories; and PostgreSQL/Drizzle persistence.
- Make each approved correctness or trust invariant enforceable at the boundary that owns the relevant state.
- Keep caller-provided values as operation inputs only; never allow them to select the account or acting wallet for a sensitive action.
- Make externally observable REST behavior explicit where consumers must make a freshness decision.
- Preserve existing behavior where compatibility permits it, but do not disguise a breaking API or migration choice as an implementation detail.
- Produce evidence strong enough for concurrent nonce and relational-invariant claims; mock-only nominal paths are not sufficient.

## Recommended technical direction

### 1. Market subscription membership as the source of reference accounting

**Recommendation.** Treat `SymbolEntry.subscribers` as the source of truth and mutate the upstream reference only after a successful membership removal. An unsubscribe for an absent symbol, a socket that is not a member, or a repeated unsubscribe is a no-op for both upstream and connection registries. Cleanup/disconnect must reuse the same guarded transition so the two registries remain consistent.

The existing `Set` already captures membership; a separately decremented `refCount` can diverge from it. The bounded direction is to make the removal transition conditional on membership (and, if the count remains, derive or keep it equal to membership size). Upstream unsubscribe occurs exactly when the final real member is removed. This fixes the approved per-process accounting defect without altering Binance ownership, gateway protocol, or multi-instance behavior.

**Trade-off.** A membership-derived count simplifies correctness but may change internal observability/tests that inspect the current counter. Keeping a counter can preserve that shape, but only if it is updated under the same successful-membership condition and tested against the set. No public protocol change is implied by this recommendation.

### 2. A first-class market-result freshness envelope

**Recommendation.** Make `PriceService` return an application-owned result that carries both ticker data and a freshness classification based on the cache decision (`fresh` or `stale`). The REST boundary should serialize a documented per-result representation rather than infer freshness from timestamps or hide it in an implementation-only header. An upstream/cache-miss failure remains an unavailable/error outcome using the established error path, but the consumer contract must make its distinction from stale data explicit.

This preserves provider normalization in `MarketTicker` and confines cache policy to `PriceService`. It also allows mixed multi-symbol responses to express freshness per symbol, which a single response-level status cannot accurately represent. The result shape is intentionally not fixed here: the compatible representation is a human decision below.

**Options and trade-offs.**

- Replacing `/prices`' bare `MarketTicker[]` with envelopes is the clearest contract, but can break callers that parse the current array.
- A versioned/parallel endpoint or an explicitly negotiated response representation preserves existing callers, but temporarily broadens the API surface and test matrix.
- Headers can signal a response-wide condition but do not naturally represent mixed fresh/stale entries and are less discoverable to typed consumers.

The recommended *semantic* direction is per-result data plus freshness. The choice between replacement and a compatibility path is not decided by this artifact.

### 3. One authenticated actor context for sensitive actions

**Recommendation.** Treat validated `req.auth` as the only source for actor `accountId`, acting `walletId`, `sessionId`, and `chainId` on the approved wallet, role, recovery, and revoke paths. Controllers should pass that trusted context to the existing domain services; body fields may remain only where they describe the operation target or proof material (for example, a new wallet address, target address, nonce, signature, or target session identifier), never the acting authority.

This changes the trust transition rather than replacing controller/service boundaries. It must also be applied to route mounting for revoke, which is presently under public `/auth`; a route cannot claim session-derived authority without authentication middleware. The domain services should receive explicit actor and target identities so audit writes use the trusted session wallet rather than an account identifier or arbitrary role lookup.

**Consequences.** Existing body authority fields should be removed, ignored, or rejected consistently; silently preferring them as a compatibility fallback would preserve the defect. A body/session mismatch should be a denial, not a mechanism to act on another account. The exact public request compatibility and session/recovery/revoke policy remain human decisions.

### 4. Atomic compare-and-consume nonce primitive

**Recommendation.** Replace every in-scope find-then-`markUsed` sequence with a common repository/service primitive that conditionally consumes the challenge at the database boundary and reports whether it won the consumption. The operation must predicate on the challenge identity/value, wallet, purpose, `usedAt IS NULL`, and expiry at consumption time, and obtain the successful row through the same operation. A second concurrent attempt must observe no successful consumption.

Signature verification may precede consumption, but it must not make a prior read authoritative. For operations that mutate roles, link wallets, provision accounts, or execute recovery, the implementation direction should place conditional consumption and the durable domain mutation within an appropriate transaction, so a failed mutation does not leave an unexplained partially completed privileged operation. Dual-nonce role operations require all required consumes to succeed as part of the same transactional outcome.

**Trade-off.** Conditional update/returning is a focused PostgreSQL-compatible mechanism and avoids an application lock. It requires callers and mock tests to adopt an outcome-bearing interface rather than a `void markUsed` convention. The exact retry/error presentation is a requirement-level concern; it must never claim a nonce remains usable after successful consumption.

### 5. Persist the authenticating wallet on every session

**Recommendation.** Make the session record the wallet that successfully established it, with a relational reference to `wallets`. Session creation writes it; middleware reads it after validating the session; refresh carries the same wallet into the rotated session; and revocation/audit receives the validated session wallet as actor attribution. Do not derive the actor by selecting the first `AUTH` or account role row.

This is compatible with the existing creation flow, which already receives `walletId`, and makes an existing trusted fact durable. Roles remain a policy input but must be evaluated for that session wallet, not for an arbitrary wallet associated with the account. Where a role change or recovery changes session validity, the chosen policy must be explicit rather than inferred from current implementation comments.

**Trade-off.** Adding an immutable session wallet reference gives accurate provenance and predictable refresh behavior, but it exposes a compatibility/migration question for sessions created before the column exists. It also forces a precise decision about whether session validity follows wallet role changes, recovery, or owner transfer.

### 6. Database-owned role invariant set, with transaction-safe role transitions

**Recommendation.** Enforce three distinct constraints in the PostgreSQL schema, supported by a forward-safe migration:

1. unique `(account_id, wallet_id, role)` associations to prohibit duplicate grants;
2. a partial unique constraint/index for `OWNER` per account and another for `RECOVERY` per account to prohibit a second concurrent assignment; and
3. a database-enforced relationship ensuring every account has exactly one owner and that the canonical `accounts.owner_wallet_id`, if retained, agrees with the OWNER association.

The first two constraints enforce "at most one". They cannot alone prove "exactly one" across the account and role tables. With the current separate `accounts.ownerWalletId` and role association model, the recommended assessment is a deferred PostgreSQL constraint trigger (or a structurally equivalent relational redesign) that checks the one-owner and canonical-owner relationship at transaction commit. Deferral permits owner transfer and recovery to make their delete/insert/update changes atomically without an invalid intermediate state. This is a recommendation for a database-boundary guarantee, not authorization to select migration SQL here.

Role transfer, recovery, and account provisioning must continue to be transactional domain operations; the database invariant is the final guard against alternate write paths and concurrent races, not a replacement for clear service behavior. Existing rows must be inspected before the migration is considered safe. This workflow authorizes no production-data repair, deletion, or backfill.

**Options and trade-offs.**

- Partial unique indexes plus a deferred constraint trigger retain the current role model and supply the requested database guarantee, at the cost of migration/test complexity.
- Using `accounts.ownerWalletId` as the sole owner representation and removing OWNER role rows would simplify one invariant but changes the existing role model and is outside this bounded remediation unless separately authorized.
- Application-only locking/checks retain simpler schema changes but do not meet the approved database-boundary outcome under alternate or concurrent writes.

The first option is recommended subject to migration compatibility evidence and a human decision on any incompatible existing state.

## Cross-boundary behavior and constraints

| Boundary | Direction | Constraint / residual risk |
| --- | --- | --- |
| Market service ↔ REST consumer | Freshness metadata crosses this boundary with the market result. | Existing consumer compatibility is unknown; do not assume an array-shape replacement is safe. |
| Gateway ↔ stream manager ↔ provider | Only a successful manager membership transition changes provider interest. | Per-process only; shared aggregation remains expressly out of scope. |
| JWT/session ↔ controller/service | The authenticated session determines actor account, wallet, session, and chain context. | Request targets and proofs still require normal validation; caller values must not regain authority through a fallback path. |
| Session ↔ refresh/revoke/audit | Session wallet is persisted and reused for actor attribution. | Recovery/role-change effects on active sessions require an explicit policy. |
| Auth service ↔ PostgreSQL | Conditional nonce consume and relational role constraints are enforced durably. | Live database concurrency and migration behavior need real database evidence; unit mocks alone are insufficient. |
| Schema ↔ migration tooling | Drizzle PostgreSQL migrations are the observed schema path (`packages/database/drizzle.config.ts`), although the migrations directory currently has no observed migration files beyond `.gitkeep`. | No deployed schema, existing data, or migration authority is evidenced here; execution must not infer them. |

## Risks and mitigations

1. **API compatibility — material.** A freshness envelope or removed authority fields can break existing callers. Mitigation: make the compatibility decision explicit before requirements fix the interface; test the chosen old/new behavior and mismatch rejection.
2. **Authorization regression — material.** A protected controller can still be vulnerable if any route bypasses middleware or services accept spoofed actor parameters. Mitigation: define one trusted actor-context path, mount revoke behind it, and add negative boundary tests for body/session conflicts and absent/invalid sessions.
3. **Recovery/session semantic risk — material.** Persisting a wallet makes the consequences of recovery, transfer, role removal, refresh, and cross-session revoke visible. Mitigation: obtain the policy decision below; test the chosen session lifecycle and audit attribution.
4. **Migration/data-integrity risk — material.** Existing role rows may violate proposed uniqueness or canonical-owner consistency. Mitigation: inspect a representative authorized database environment before migration; stop for incompatible state instead of repairing production data under this run.
5. **Concurrency-evidence risk — material.** A mocked repository can make conditional consumption and partial unique constraints appear correct without proving PostgreSQL behavior. Mitigation: test against a disposable PostgreSQL database with competing nonce and role writes, including migration application.
6. **Atomicity-policy risk.** Consuming a valid nonce outside a transaction that later fails can produce confusing retry behavior. Mitigation: make consumption and its privileged mutation one transaction where the operation mutates durable state; explicitly specify failure semantics.
7. **Scope-expansion risk.** Inspection may expose adjacent audit, provider, multi-instance, or client concerns. Mitigation: retain the six specified outcomes as the only architectural target.

## Assumptions and decisions

### Established decisions carried from approved upstream artifacts

- This is a Contract v1.2 **delivery** run for exactly the six outcomes in `DEF-0002@v1`.
- Authority for the named sensitive actions must derive from the authenticated session rather than caller-supplied account or wallet identifiers.
- A nonce must have atomic single-use behavior; session attribution must retain the authenticating wallet; and the stated role invariants must be enforced at the database boundary.
- No package-manager, unrelated evaluation, client, deployment, merge, release, or production-data work is authorized.

### Architectural recommendations, not approved decisions

- Preserve current component ownership; strengthen the affected transitions and persistence boundaries instead of replacing market, auth, or database architecture.
- Use a per-result market freshness representation, trusted actor context, conditional database nonce consumption, persistent session wallet identity, and PostgreSQL role constraints plus a deferred exact-owner guarantee.
- Verify concurrency/migration behavior against a disposable PostgreSQL environment before treating the security and integrity outcomes as established.

### Human decisions required

1. **Market API compatibility (required before interface requirements are fixed):** may `/prices` change from a bare ticker array to the freshness-aware representation, or must a versioned/parallel/negotiated compatibility path preserve existing consumers? The recommended semantics are per-result data plus freshness; this artifact does not choose the rollout contract.
2. **Existing data and migration disposition (required before execution):** if an authorized preflight finds duplicate roles, multiple owners/recovery roles, or a mismatch between `accounts.ownerWalletId` and roles, should this run stop for a separately authorized data decision, or is a specific non-production-compatible migration treatment authorized? No production-data repair is approved here.
3. **Session/recovery/revoke policy (required before requirements finalize behavior):** should refresh always preserve the original authenticating wallet (recommended), and which session-derived actor may revoke which sessions (self only versus another session of the same account)? Also decide whether recovery, ownership transfer, or loss of an authorizing role revokes existing sessions. The approved scope requires session-derived authority but does not decide these lifecycle semantics.
4. **Database-backed test environment (required before routing/execution evidence):** confirm an authorized disposable PostgreSQL environment can apply migrations and run competing-write tests. If unavailable, the run must surface the resulting verification limitation rather than claim concurrent database guarantees.

These are decision-ready questions, not an authorization to expand scope. The first and third affect external behavior; the second affects migration safety; the fourth affects what verification can truthfully establish.

## Consequences for subsequent stages

Requirements should turn the approved decisions and recommendations into traceable interface, security, persistence, migration, and evidence criteria without inventing a response shape, lifecycle rule, existing-data treatment, or test environment. Decomposition and Routing remain responsible for file ownership, implementation units, target choice, and repository/PR verification. No product changes, tasks, routes, schema migration, or worker assignment are made by this artifact.

## Self-check and requested validation

`ARC-0002@v1` has exact `DEF-0002@v1` lineage, retains the approved delivery mode and six-finding boundary, and distinguishes upstream decisions from architectural recommendations, assumptions, unresolved human choices, and verified inspection evidence. It recommends bounded directions across the existing market, authentication/session, and PostgreSQL boundaries without creating requirements, work units, routing, product changes, a migration, an approval, or a scope expansion. **READY_FOR_REQUIREMENTS is requested for orchestrator validation; this Architecture does not validate itself, advance the workflow, or approve any human decision.**
