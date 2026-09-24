---
artifact:
  id: DEC-0002
  type: DECOMPOSITION
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
    - REQ-0002@v1
  references:
    - path: requirements_engineer/requirements-v1.md
    - path: solution_architect/architecture-v2.md
    - path: solution_architect/architecture-v1.md
    - path: workflow_orchestrator/architecture-decisions-v1.md
    - path: problem_analyst/definition-v1.md
    - path: intake_analyst/handoff-v1.md
    - path: intake_analyst/handoff-summary-v1.md
    - path: workflow_orchestrator/orchestration-plan-v1.md
    - product_repository: Atra-Services
      branch: main
      commit: be51242593fe6643aff11c491d4f8a6c0835aed5
      state: observed read-only during decomposition inspection; not re-verified as an implementation baseline
ownership:
  created_by: work_planner
  performed_by: work_planner
  recorded_by: work_planner
  engine: claude-code
  target_id: claude-cli
---

# Decomposition — Atra evaluation remediation

## Contract, authority, and scope carried forward

`DEC-0002@v1` turns validated `REQ-0002@v1` (and its `ARC-0002@v2`/`DEC-0001@v1` lineage) into bounded, dependency-aware Work Units for the fixed six-finding delivery scope in `WF-0002`. It covers exactly: market-stream unsubscribe accounting (`REQ-MKT-001`), `/prices` cache freshness (`REQ-API-002`), session-derived sensitive-action authority (`REQ-AUTH-003`), atomic nonce single use (`REQ-NONCE-004`), durable authenticating-wallet session attribution (`REQ-SES-005`), and account-wallet-role integrity at the database boundary (`REQ-DATA-006`), plus the cross-cutting delivery-evidence requirements `REQ-EVD-007` and `REQ-DEL-008`.

This decomposition **excludes**, unchanged from every upstream artifact: package-manager work, audit cursor pagination, multi-instance market aggregation/cache behavior, iOS/client/provider/portfolio/Phase 2.5 expansion, deployment, merge, release, production-data operations, and any product capability not demonstrably necessary for the six outcomes. No Work Unit below authorizes any of these.

This artifact selects no target, model, or worker; creates no routing plan, code, test, migration file, manifest change, validation record, event, or pull request; and performs no product edit. File paths cited below are read-only observations recorded to bound scope, gathered by inspecting `Atra-Services` at `main` / `be51242593fe6643aff11c491d4f8a6c0835aed5`; they are not an implementation instruction, and the stages responsible for baseline/branch/PR authority (Routing, Execution) must independently reverify the working repository state before any edit. The authorized disposable, non-production PostgreSQL environment (`DEC-0001@v1` §4) is recorded here only as a later dependency for two units below (`WORK-006`, `WORK-008`); this artifact does not use, provision, or connect to it.

## Dependency-type legend

Each unit's dependency entries use one of these types:

- **none** — independently startable; no other unit's output is required first.
- **hard-gate** — the dependency is a blocking safety gate; the dependent unit may not proceed until the upstream unit's stop/continue outcome is known, and a "stop" outcome blocks the dependent unit entirely (human decision required to resume, per `DEC-0001@v1` §2).
- **interface** — the two units change behavior on the same call boundary (e.g., a shared service method's signature or contract) without touching the same file; safe to execute in parallel only if the interface change is agreed before either starts, otherwise sequence.
- **data** — the dependent unit's correctness relies on a fact or field the upstream unit establishes (e.g., a persisted column, a corrected derivation); not a file conflict, but the dependent unit's acceptance criteria cannot be truthfully met until the upstream unit's behavior exists.
- **resource-overlap** — both units touch the same file, directory, or generated-artifact stream (e.g., the migrations folder) and need explicit sequencing or coordinated authorship to avoid a conflicting or duplicated change.

## Work Units

### WORK-001 — Membership-gated market-stream unsubscribe accounting

- **Purpose.** Make subscription/unsubscription reference accounting in the per-process market-stream manager conditional on actual membership, so an absent-symbol, non-member, or repeated unsubscribe cannot affect another socket's retained interest or upstream subscription state.
- **Requirement references.** `REQ-MKT-001` (primary); `REQ-EVD-007` (evidence mapping).
- **Scope.** The membership-removal transition and its reuse by disconnect cleanup in the market-stream subscription manager. Upstream (Binance) unsubscription must occur only when the final real member is removed.
- **Inputs.** Current per-symbol subscriber-set and reference-count state; subscribe/unsubscribe/disconnect call sites that mutate them.
- **Outputs.** A corrected, guarded membership-removal transition used consistently by unsubscribe and disconnect cleanup; no change to upstream adapter protocol or transport.
- **Affected areas (observed, not prescriptive).** `apps/market-service/src/services/MarketStreamManager.ts` (`SymbolEntry.subscribers`, `refCount`, `_subscribeOne`/`_unsubscribeOne`/`_removeSocketFromSymbol`). No change is expected to `apps/market-service/src/ws/WebSocketGateway.ts` (transport) or `apps/market-service/src/adapters/BinanceWsAdapter.ts` (upstream ownership) beyond the existing call already made only on a successful membership transition.
- **Constraints.** No gateway protocol change; no provider-ownership change; no multi-instance or shared-aggregation claim or design.
- **Risks.** An existing counter/test may currently inspect a separately maintained `refCount`; deriving or guarding it against membership can change that internal shape and must not silently diverge from the `subscribers` set (`ARC-0002@v2` §1 trade-off).
- **Observable completion criteria.** `REQ-MKT-001` acceptance (a)–(c): non-member/repeated/absent-symbol unsubscribe leaves retained interest and upstream interest unchanged; one real member's removal while another remains does not remove upstream interest; upstream unsubscription occurs only after the final real member releases the symbol. Evidence is explicitly limited to the in-process manager (no multi-instance claim).
- **Dependencies.** None (independent).

### WORK-002 — Direct freshness-aware `/prices` response

- **Purpose.** Replace the bare `MarketTicker[]` response with a per-result representation exposing ticker data plus an independently observable `fresh`/`stale` classification, keeping an unavailable/error outcome distinguishable from a stale successful result.
- **Requirement references.** `REQ-API-002` (primary); `REQ-EVD-007` (evidence mapping).
- **Scope.** The application-owned result shape produced by the price/cache service and its direct serialization at the `/prices` REST boundary. No versioned, parallel, negotiated, or legacy-compatible response path is required or permitted (`DEC-0001@v1` §1).
- **Inputs.** Existing cache stale-while-revalidate decision and `MarketTicker` provider-normalized data.
- **Outputs.** A freshness-aware per-result response type consumed directly by the REST router; existing cache-miss/upstream-unavailable error handling remains a distinct outcome, not a stale success.
- **Affected areas (observed, not prescriptive).** `apps/market-service/src/services/PriceService.ts` (`getTicker`/`getTickers`, cache decision), `apps/market-service/src/rest/router.ts` (`GET /prices` serialization). `apps/market-service/src/cache/PriceCache.ts` / `InMemoryPriceCache.ts` are read-only context for the existing cache-state signal.
- **Constraints.** Direct breaking change is authorized; no compatibility route, response negotiation, or legacy body shape may be added.
- **Risks.** None material beyond ensuring mixed multi-symbol responses expose per-result state rather than one response-wide status (`ARC-0002@v2` §2).
- **Observable completion criteria.** `REQ-API-002` acceptance (a)–(d): current cached data returned as `fresh`; usable cached data pending refresh returned as `stale`; a cache-miss/upstream-unavailable outcome is observably distinct from a successful stale result; a response with multiple results exposes each result's own state.
- **Dependencies.** None (independent). File-disjoint from `WORK-001` within the same `market-service` app (`MarketStreamManager.ts`/`WebSocketGateway.ts` vs. `PriceService.ts`/`router.ts`); safe to run in parallel with `WORK-001`.

### WORK-003 — Session-derived actor authority enforcement at the controller/route boundary

- **Purpose.** Make every in-scope wallet, role, recovery, and revoke controller derive its acting account/wallet/session/chain authority exclusively from the validated authenticated session context, reject a body/session authority mismatch, mount revoke behind authentication, and enforce self-only revoke and session-wallet audit attribution.
- **Requirement references.** `REQ-AUTH-003` (primary); `REQ-EVD-007` (evidence mapping).
- **Scope.** Controller-level actor-source changes and route mounting for the named wallet/role/recovery/revoke action families. Caller-supplied identity fields remain usable only as operation targets or proof material (new/target address, nonce, signature, target session id), never as acting authority.
- **Inputs.** The corrected authenticated-session actor context this unit consumes as a trusted fact (see Dependencies).
- **Outputs.** Controllers and the revoke route that read/require the trusted session context, reject spoofed or mismatched body authority, and pass explicit actor identity into the domain services they call.
- **Affected areas (observed, not prescriptive).** `apps/auth-service/src/modules/wallets/controllers/WalletController.ts`, `apps/auth-service/src/modules/roles/controllers/RoleController.ts`, `apps/auth-service/src/modules/recovery/controllers/RecoveryController.ts`, `apps/auth-service/src/modules/auth/controllers/AuthController.ts` (`revoke`), and route mounting in `apps/auth-service/src/modules/auth/routes/authRoutes.ts` / `apps/auth-service/src/index.ts` (moving `revoke` off the public `/auth` mount to an authenticated mount, consistent with `refresh` remaining public per its own credential).
- **Constraints.** Revoke is self-only (`DEC-0001@v1` §3); a body/session mismatch is a denial, not a fallback; existing body authority fields must be removed or rejected consistently, not silently preferred.
- **Risks.** A route that bypasses the authentication boundary, or a service that still accepts a spoofed actor parameter, leaves the defect materially unresolved (`ARC-0002@v2` risk 2).
- **Observable completion criteria.** `REQ-AUTH-003` acceptance (a)–(d): missing/invalid session cannot authorize; body/session authority mismatch is rejected; a caller cannot broaden revoke authority via a body identifier; observed audit attribution for a successful action is the session's authenticated wallet, including for a multi-wallet account.
- **Dependencies.**
  - **data** on `WORK-005`: acceptance criterion (d) and the general "session-derived" authority claim are only truthfully met once the authenticated-session wallet used by the middleware/actor context is the durably persisted session wallet from `WORK-005`, not an arbitrarily selected role row. `WORK-003` must not be marked complete against a middleware actor context that still derives `walletId` by selecting a first `AUTH`-role row.
  - **interface** with `WORK-004`: `RoleController`→`RoleService` and `RecoveryController`→`RecoveryService` are the same call boundaries `WORK-004` changes internally (outcome-bearing, transactional nonce consumption). The two units do not share a file (controllers vs. services) but must agree on the resulting service-method signature (added explicit actor/target parameters vs. added outcome-bearing return) before either is considered final; recommend agreeing the interface first, then either unit may implement in parallel.

### WORK-004 — Atomic compare-and-consume nonce primitive

- **Purpose.** Replace every in-scope find-then-mark-used nonce path with one shared, database-conditional consume operation so a competing concurrent attempt can never obtain a second successful consumption, and so a losing/failed attempt leaves no partial privileged outcome.
- **Requirement references.** `REQ-NONCE-004` (primary); `REQ-EVD-007` (evidence mapping).
- **Scope.** A single conditional-consume operation predicated on challenge identity/value, wallet, purpose, unused state, and expiry at the decision point, used by every in-scope verification flow; transactional pairing of consumption with the durable domain mutation it gates; all-or-nothing behavior for operations needing more than one nonce.
- **Inputs.** Existing per-service find-then-update nonce logic and the domain mutations each currently guards (login/provisioning, wallet linking, role change, recovery).
- **Outputs.** One outcome-bearing consume primitive and refactored call sites that use it inside the same transaction as their durable mutation; no change to unrelated nonce purposes or expiry policy.
- **Affected areas (observed, not prescriptive).** `apps/auth-service/src/modules/identity/services/NonceService.ts` (`markUsed` → conditional consume), `apps/auth-service/src/modules/identity/repositories/NonceRepository.ts`, and the four current independent find-then-consume call sites: `apps/auth-service/src/modules/identity/services/AccountService.ts` (`verifyAndProvision`), `apps/auth-service/src/modules/wallets/services/WalletLinkingService.ts` (`verifyAndLink`), `apps/auth-service/src/modules/roles/services/RoleService.ts` (`verifyAndApply`, dual owner+target nonce), `apps/auth-service/src/modules/recovery/services/RecoveryService.ts` (`executeRecovery`). `packages/database/src/schema/nonceChallenges.ts` is read-only context (no schema change expected; the guarantee is query-conditional, not structural).
- **Constraints.** Signature/proof verification may precede consumption but must never make a prior read authoritative; a losing attempt must not create the durable outcome it was meant to gate.
- **Risks.** Implementation-only evidence is insufficient: mocked nominal-path tests cannot establish concurrent atomicity (`ARC-0002@v2` risk 3) — this unit's own execution needs no database evidence to *build* the primitive, but its acceptance is not verified until `WORK-008` (below) produces database-backed evidence.
- **Observable completion criteria.** `REQ-NONCE-004` acceptance (a)–(c) at the implementation level: the primitive's predicate and transactional pairing are in place for every in-scope flow, and multi-nonce operations are structured so partial consumption cannot occur without the paired mutation. Concurrent-attempt proof itself is `WORK-008`'s completion criterion, not this unit's.
- **Dependencies.**
  - **interface** with `WORK-003` (see `WORK-003` entry) on `RoleService`/`RecoveryService` method shape.
  - **none** otherwise; file-disjoint from `WORK-001`, `WORK-002`, `WORK-005`, `WORK-006`/`WORK-007` (no shared file).

### WORK-005 — Durable session-to-wallet attribution and lifecycle revocation

- **Purpose.** Persist the wallet that successfully authenticates a session as a durable, relational fact on the session record; make every downstream authentication/authorization/audit/revoke read that persisted wallet instead of an arbitrarily selected account-role row; and revoke the sessions affected by recovery, ownership transfer, or loss of an authorizing role.
- **Requirement references.** `REQ-SES-005` (primary); `REQ-AUTH-003` (supplies the corrected actor-context fact `WORK-003` depends on); `REQ-EVD-007` (evidence mapping).
- **Scope.** Session creation, the middleware's post-validation actor-context construction, refresh (which must carry the same wallet forward), and the selected session-lifecycle revocation events (`DEC-0001@v1` §3): recovery, ownership transfer, loss of an authorizing role. Revoke's self-only enforcement itself is `WORK-003`'s scope; this unit supplies the correct wallet identity revoke and audit attribution rely on.
- **Inputs.** The wallet id already supplied at session creation; existing session/refresh/role-lookup code paths that currently select a first matching role row instead of reading a persisted value.
- **Outputs.** A session-level persisted wallet reference; a middleware/refresh path that reads it instead of re-deriving it; explicit revocation triggers wired to the three selected lifecycle events.
- **Affected areas (observed, not prescriptive).** `apps/auth-service/src/modules/auth/services/SessionService.ts` (`create`, `refresh`, `revoke`), `packages/database/src/schema/sessions.ts` (add a relational wallet reference — this is a schema/migration change, see Constraints), `apps/auth-service/src/middleware/authenticate.ts` (replace the first-`AUTH`-role-row selection with the persisted session wallet), `apps/auth-service/src/modules/auth/services/TokenService.ts` (claims, if the persisted wallet is reflected there), `apps/auth-service/src/modules/auth/repositories/SessionRepository.ts`. Lifecycle-revocation wiring touches the recovery and role-change domain services already in scope for `WORK-004` (`RecoveryService.ts`, `RoleService.ts`) at their mutation boundary, not their nonce-consumption boundary.
- **Constraints.** Refresh preserves the original authenticating wallet (`DEC-0001@v1` §3); no middleware or refresh path may select a first `AUTH`/`OWNER` row to infer identity. Adding a relational wallet column to `sessions` is a schema change requiring a migration; this workflow authorizes no production connection or data repair, so migration compatibility for this column is verified only in the authorized disposable PostgreSQL environment (`WORK-008`), not asserted here.
- **Risks.** Persisting an immutable session wallet raises a real compatibility question for any session created before the column exists; this run's scope does not include a production migration decision, only the disposable-environment verification described in `WORK-008` (`ARC-0002@v2`/`ARC-0002@v1` §5 trade-off).
- **Observable completion criteria.** `REQ-SES-005` acceptance (a)–(d): for a multi-wallet account, session-authenticated behavior and audit attribution resolve to the session's signing wallet; a refreshed session retains that same wallet; revoke is self-only using authenticated attribution (jointly with `WORK-003`); each affected session can no longer authenticate/authorize after recovery, ownership transfer, or loss of an authorizing role.
- **Dependencies.**
  - **resource-overlap** with `WORK-007`: both add/alter schema in `packages/database` (a new `sessions` column here vs. role/owner constraints there) and both land in the same `packages/database/migrations/` output directory (currently empty but for `.gitkeep`). Coordinate migration-file ordering/authorship so the two schema changes do not collide or get generated against inconsistent intermediate schema state.
  - **none** toward `WORK-001`, `WORK-002` (file-disjoint, different app).

### WORK-006 — Incompatible existing role/owner data preflight (hard gate)

- **Purpose.** Before any role/owner invariant migration is written or applied, authoritatively detect whether currently reachable account-wallet-role data already violates the invariants the migration would enforce — a duplicate `(account, wallet, role)` association, more than one `OWNER` or `RECOVERY` per account, or a mismatch between `accounts.ownerWalletId` and the account's `OWNER` role association — and stop with reported evidence if so.
- **Requirement references.** `REQ-DATA-006` (primary, preflight clause); `DEC-0001@v1` §2 (binding stop-and-report decision).
- **Scope.** A read-only inspection query/report against the representative data available for this run; it is a detection and evidence-recording activity, not a repair, deletion, or backfill. Its only two outcomes are "no incompatibility found → `WORK-007` may proceed" or "incompatibility found → stop and report; this workflow performs no repair."
- **Inputs.** The existing `accounts` (`ownerWalletId`, no observed FK to `wallets`) and `account_wallet_roles` (`role` enum `OWNER`/`AUTH`/`STANDARD`/`RECOVERY`) data reachable in the environment used for this check.
- **Outputs.** A pass/stop determination with reported evidence (which rows/accounts, which violated invariant), consumed as a hard gate by `WORK-007`.
- **Affected areas (observed, not prescriptive).** Read-only queries against `packages/database/src/schema/accounts.ts` and `packages/database/src/schema/accountWalletRoles.ts` shapes; no schema or application file is edited by this unit.
- **Constraints.** No production connection, repair, deletion, backfill, or acceptance of incompatible data under any outcome (`DEC-0001@v1` §2). The environment this preflight actually runs against, and whether it is production-representative, must be verified by the stage that dispatches it (Routing/Execution); this unit does not select or provision that environment.
- **Risks.** A preflight that runs against a non-representative dataset could pass incorrectly; the evidence record must state what was checked so a later stage can judge representativeness.
- **Observable completion criteria.** `REQ-DATA-006` acceptance (c): an authorized preflight that detects duplicate roles, multiple owners/recovery roles, or a canonical-owner mismatch stops the workflow and reports evidence, with no repair/delete/backfill/acceptance performed. A "no incompatibility found" outcome is equally a valid, evidenced completion.
- **Dependencies.** **none** to start (independent, read-only, can run at any time relative to the other units). It is the **hard-gate upstream** of `WORK-007`.

### WORK-007 — Database-boundary role/owner invariant migration

- **Purpose.** Enforce, at the PostgreSQL boundary, that committed state prohibits a duplicate `(account, wallet, role)` association, more than one `OWNER` per account, and more than one `RECOVERY` per account, and that a retained canonical owner-wallet relationship agrees with the account's `OWNER` association at commit.
- **Requirement references.** `REQ-DATA-006` (primary, migration clause).
- **Scope.** A forward migration adding a unique constraint on `(account_id, wallet_id, role)`, partial unique constraints/indexes for one `OWNER` and one `RECOVERY` per account, and an exact-owner guarantee (architecture recommends a deferred constraint trigger, or a structurally equivalent relational mechanism, so account provisioning/ownership transfer/recovery can complete through valid atomic transactions). Selecting the exact mechanism is implementation-stage, not decomposition-stage, work; this unit records the guarantee to be established, not the SQL.
- **Inputs.** The existing `accounts`/`account_wallet_roles` schema and Drizzle migration tooling (`packages/database/drizzle.config.ts`, schema at `./src/schema/index.ts`, output at `./migrations`, currently empty but for `.gitkeep`).
- **Outputs.** A generated Drizzle migration applied only in the authorized disposable environment (verification is `WORK-008`, not this unit); existing application-level role/ownership transaction logic remains the primary domain-orchestration boundary, with the database constraint as the final guard.
- **Affected areas (observed, not prescriptive).** `packages/database/src/schema/accountWalletRoles.ts`, `packages/database/src/schema/accounts.ts`, and a new file under `packages/database/migrations/`.
- **Constraints.** No production connection, data operation, deployment, or persistent environment change (`DEC-0001@v1` §4). This unit may not proceed past design into a generated/applied migration unless `WORK-006` reports no incompatible data.
- **Risks.** The two "at most one" constraints alone cannot prove "exactly one owner"; the exact-owner guarantee needs the deferred-trigger (or equivalent) mechanism, which is itself schema/transaction-sensitive and must be evidenced, not assumed (`ARC-0002@v2` §6).
- **Observable completion criteria.** `REQ-DATA-006` acceptance (a)–(b) at the schema-design level: the migration, once applied in the disposable environment, rejects a duplicate role association, a second `OWNER`, or a second `RECOVERY`; account provisioning and ownership transfer are structured to complete only in a committed, single-consistent-owner state. Competing-write proof itself is `WORK-008`'s completion criterion.
- **Dependencies.**
  - **hard-gate** from `WORK-006`: must not be applied to any environment, and is not considered actionable past design, until `WORK-006` reports "no incompatibility found." A `WORK-006` "stop" outcome blocks this unit entirely pending a separately authorized human decision.
  - **resource-overlap** with `WORK-005` (see `WORK-005` entry) in `packages/database` schema/migrations.

### WORK-008 — Database-backed concurrency and invariant verification evidence

- **Purpose.** Produce the database-backed evidence `REQ-NONCE-004` and `REQ-DATA-006` require and that mocked nominal-path tests cannot establish: competing-write nonce-consumption behavior and role/owner constraint enforcement under alternate and competing writes, using the authorized disposable, non-production PostgreSQL environment.
- **Requirement references.** `REQ-NONCE-004` (concurrency clause), `REQ-DATA-006` (invariant/competing-write clause), `REQ-EVD-007` (evidence sufficiency).
- **Scope.** Applying `WORK-007`'s migration and exercising `WORK-004`'s conditional-consume primitive and the resulting schema constraints against real concurrent/competing writes in the disposable environment. This unit is evidence production, not new product behavior; it changes no application code beyond what `WORK-004`/`WORK-007` already produced.
- **Inputs.** `WORK-004`'s completed conditional-consume primitive; `WORK-007`'s migration, already cleared by `WORK-006`.
- **Outputs.** Recorded observations: concurrent-attempt nonce-consumption outcomes (no more than one success), and rejected duplicate-role/second-OWNER/second-RECOVERY attempts under competing writes, including migration-application evidence.
- **Affected areas (observed, not prescriptive).** No new product files beyond what `WORK-004`/`WORK-007` already touch; this unit's own output is evidence, recorded by the stages authorized to run and report it (Execution/Test), not a Decomposition-stage artifact.
- **Constraints.** The disposable PostgreSQL environment is authorized only for this migration/concurrency evidence, per `DEC-0001@v1` §4 — no production connection, data operation, deployment, or persistent environment change. This unit is a recorded future dependency here, not an operation performed by this Decomposition artifact.
- **Risks.** If the disposable environment is unavailable, the required database-backed claims for `REQ-NONCE-004` and `REQ-DATA-006` cannot pass as verified (`REQ-0002@v1` §Dependencies); this is a reportable limitation, not a substitutable mocked-only result.
- **Observable completion criteria.** `REQ-NONCE-004` acceptance (a): concurrent verification attempts for one otherwise valid nonce yield no more than one successful consumption and no more than one resulting privileged outcome. `REQ-DATA-006` acceptance (a)–(b) under actual competing writes, plus migration-application evidence.
- **Dependencies.**
  - **hard-gate** transitively via `WORK-007` ← `WORK-006` (cannot run against an unapplied or blocked migration).
  - **data** on `WORK-004` (needs the completed conditional-consume primitive to exercise).

## Cross-cutting requirements not realized as Work Units

- **`REQ-EVD-007`** (delivery evidence mapping) is not a separate Work Unit; it is a completion criterion embedded in every unit above (each cites the exact acceptance-criteria letters it must map evidence to) and is closed out by the later Test and Review stages against the actual product change, per `OPLAN-0002@v1` stage 10–11.
- **`REQ-DEL-008`** (forward-only delivery, execution gate, and required PRs) is not a Work Unit; Decomposition has no authority to create routing, an execution gate, or a pull request. It is realized by the Routing stage (execution scope and target selection), the second human execution gate, the Execution stage (product PR), and the Run Report (artifact PR), per `OPLAN-0002@v1` stages 7–12 and Contract v1.2 §Human gates.

## Parallelism, overlap, and cycle analysis

**Safe to start immediately, in parallel, with no coordination required:** `WORK-001`, `WORK-002`, `WORK-005`, `WORK-006`. These four touch disjoint files (`MarketStreamManager.ts`; `PriceService.ts`/`router.ts`; `SessionService.ts`/`sessions.ts`/`authenticate.ts`; and `WORK-006`'s read-only inspection with no file writes at all).

**Sequenced or coordinated pairs:**

| Pair | Type | Reason | Recommended handling |
| --- | --- | --- | --- |
| `WORK-006` → `WORK-007` | hard-gate | Migration must not be generated/applied past design before the preflight clears. | Do not dispatch `WORK-007`'s applied-migration step until `WORK-006` reports "no incompatibility found." |
| `WORK-005` → `WORK-003` | data | `WORK-003`'s session-derived-authority claim needs `WORK-005`'s corrected persisted-wallet fact to be true. | Complete or substantially land `WORK-005`'s middleware/session-wallet change before validating `WORK-003`'s acceptance criterion (d); the two may still be authored concurrently if `WORK-003` is not marked complete first. |
| `WORK-003` ↔ `WORK-004` | interface | Both change behavior on the `RoleService`/`RecoveryService` call boundary (controller-side actor params vs. service-side outcome-bearing transactional consume). | Agree the resulting method signature before either lands; then both may implement in parallel. |
| `WORK-005` ↔ `WORK-007` | resource-overlap | Both generate schema/migration files under `packages/database`. | Coordinate migration authorship/ordering (e.g., distinct migration files generated against a consistent intermediate schema) rather than two independently generated, conflicting migrations. |
| `WORK-004`, `WORK-007` → `WORK-008` | data / hard-gate (transitive) | Evidence needs both the completed primitive and the cleared, applied migration. | Dispatch `WORK-008` only after `WORK-004` is complete and `WORK-007` has passed its `WORK-006` gate and been applied in the disposable environment. |

**Cycle check.** The full dependency graph is: `WORK-006 → WORK-007 → WORK-008 ← WORK-004`; `WORK-005 → WORK-003`; `WORK-003 ↔ WORK-004` (interface, non-blocking); `WORK-001`, `WORK-002` isolated. No unit depends, directly or transitively, on its own output. **No cycle exists.**

## Constraints, risks, and exclusions retained workflow-wide

- Exactly the six approved findings and the two delivery-evidence requirements are covered by the units above; no unit introduces a capability beyond `REQ-MKT-001`–`REQ-DATA-006`, `REQ-EVD-007`, or `REQ-DEL-008`.
- No unit authorizes package-management work, audit pagination, multi-instance market behavior, iOS/client/provider/portfolio/Phase 2.5 expansion, deployment, merge, release, or a production-data operation.
- The disposable PostgreSQL environment referenced by `WORK-006` (inspection) and `WORK-008` (applied migration + competing writes) is a later dependency to be provisioned and used only by the stages authorized to do so; this artifact neither provisions nor connects to it.
- `WORK-006`'s "stop and report" outcome is the only approved response to incompatible existing role/owner data; no unit authorizes repair, deletion, backfill, or acceptance of that data. A stop outcome blocks `WORK-007` and, transitively, `WORK-008`'s role-invariant evidence until a separately authorized human decision is recorded.
- File paths recorded above are read-only observations from repository inspection at the recorded baseline commit and are not an implementation directive; actual file scope, exact worker assignment, and product-repository authority remain Routing's and Execution's responsibility.

## Self-check and requested validation

`DEC-0002@v1` has exact `REQ-0002@v1` lineage and traces every one of the eight requirement IDs (`REQ-MKT-001`, `REQ-API-002`, `REQ-AUTH-003`, `REQ-NONCE-004`, `REQ-SES-005`, `REQ-DATA-006`, `REQ-EVD-007`, `REQ-DEL-008`) to at least one Work Unit or an explicit cross-cutting note. It records, for each of the eight Work Units, purpose, requirement references, scope, inputs/outputs, affected areas, constraints, risks, observable completion criteria, and typed dependencies. It identifies four unit pairs needing sequencing or coordination, one explicit hard-gate dependency (`WORK-006` before `WORK-007`), and confirms no dependency cycle exists. It preserves the exact six-finding scope and every listed exclusion, treats the disposable PostgreSQL environment as an authorized future dependency rather than a current operation, and selects no target, model, or worker; creates no routing plan, code, test, migration, manifest change, validation record, event, or pull request; and makes no product edit. **READY_FOR_ROUTING is requested for orchestrator validation; this Decomposition does not validate itself, approve a gate, or advance the workflow.**
