---
artifact:
  id: ROUTE-0002
  type: ROUTING_PLAN
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
    - ROUTE-0002@v1
    - DEC-0002@v1
    - REQ-0002@v1
    - ARC-0002@v2
    - DEF-0002@v1
    - HAND-0002@v1
    - HSUM-0002@v1
    - DEC-0001@v1
    - OPLAN-0002@v1
    - DEC-0003@v1
    - ENV-0001@v1
    - DEC-0004@v1
  references:
    - path: execution_router/routing-plan-v1.md
    - path: work_planner/decomposition-v1.md
    - path: requirements_engineer/requirements-v1.md
    - path: solution_architect/architecture-v2.md
    - path: problem_analyst/definition-v1.md
    - path: intake_analyst/handoff-v1.md
    - path: intake_analyst/handoff-summary-v1.md
    - path: workflow_orchestrator/architecture-decisions-v1.md
    - path: workflow_orchestrator/orchestration-plan-v1.md
    - path: workflow_orchestrator/routing-prerequisites-v1.md
    - path: workflow_orchestrator/postgresql-environment-v1.md
    - path: workflow_orchestrator/representative-data-decision-v1.md
    - path: workflow_orchestrator/validations/DEC-0002-v1.md
    - path: workflow_orchestrator/validations/ENV-0001-v1.md
ownership:
  created_by: execution_router
  performed_by: execution_router
  recorded_by: execution_router
  engine: codex
  target_id: codex-native
---

# Routing Plan — Atra evaluation remediation

## Status, authority, and scope

`ROUTE-0002@v2` is the bounded same-stage replacement for blocked `ROUTE-0002@v1`. It resolves only the formerly missing product-head and disposable-environment facts through approved `DEC-0003@v1`, validated `ENV-0001@v1`, and approved `DEC-0004@v1`. It does not alter approved requirements, architecture, decomposition, scope, or exclusions; creates no product change, migration, branch, PR, environment/configuration change, database operation, test run, or worker dispatch.

Scope remains exactly the six verified findings: market unsubscribe accounting (`REQ-MKT-001`), direct freshness-aware `/prices` results (`REQ-API-002`), session-derived sensitive-action authority (`REQ-AUTH-003`), atomic nonce consumption (`REQ-NONCE-004`), durable session-wallet attribution/lifecycle revocation (`REQ-SES-005`), and database role/owner integrity (`REQ-DATA-006`), together with `REQ-EVD-007` and `REQ-DEL-008` delivery evidence. Excluded: package-manager work, audit pagination, multi-instance market behavior, iOS/client/provider/portfolio/Phase-2.5 work, deployment, merge, release, production-data operations, and capabilities not necessary for these findings.

This is a delivery route, not execution authorization. Controller validation and the distinct Contract v1.2 human execution gate remain mandatory before any product edit, migration application, or test/evidence run.

## Verified product, PR, and environment route

| Fact | Verified observation | Route consequence |
| --- | --- | --- |
| Product Git root / repository | `/Users/danielvelikov/Developer/Atra-Services`; `https://github.com/SampleApp05/Atra-Services.git` fetch and push origin | all allowlists below are relative to this root |
| Base / starting state | `main` and `origin/main` at clean `be51242593fe6643aff11c491d4f8a6c0835aed5` | verified baseline; preserve unrelated changes if state changes before dispatch |
| Dedicated head | local and origin `workflow/WF-0002-atra-evaluation-remediation` exist at that same commit | verified execution-coordinator product head; no product PR exists yet |
| PR authority | authenticated `SampleApp05` has GitHub `ADMIN` permission and `repo` token scope | `execution_coordinator` may create/update the one required product PR, never workers |
| Disposable PostgreSQL | PostgreSQL 17.11 at loopback `127.0.0.1:55432`; dedicated role/database `atra_wf0002`; state rooted at `/tmp/atra-wf0002-postgres`; `ENV-0001@v1` liveness/isolation evidence passed | authorized only for later migration, preflight, and competing-write evidence |
| Existing data population | human-approved `DEC-0004@v1`: no applicable account, wallet, role, recovery, or session population exists | see binding preflight handling below |

The empty-data decision satisfies `WORK-006`/EU-03's **read-only incompatibility-preflight condition without a database data operation**: there are no rows to inspect, so no incompatible existing record can be found, repaired, backfilled, deleted, or accepted. The execution record must cite `DEC-0004@v1` and record `NO_EXISTING_APPLICABLE_DATA`, not claim a query result or data compatibility. This is not authorization to skip EU-04 migration application or EU-06 database-backed migration/concurrency evidence; those still must run against the verified disposable environment after the execution gate.

No production endpoint, data import/seed, repair, backfill, deletion, deployment, merge, release, shared persistent service, or credential record is authorized. The non-secret connection reference is `postgres://atra_wf0002@127.0.0.1:55432/atra_wf0002`; it is restricted to the cited workflow use and must not be replaced by an unverified endpoint.

## Dispatch rules and target pools

All implementation units have role `implementation_worker`. The actual worker is selected only immediately before dispatch after fresh health, capacity, availability, context/file-limit, and independence checks. `[LOCAL] mac-ollama` permits at most six files/64,000 context bytes/16,000 prompt characters; `[WINDOWS] windows-4070` permits at most three files/32,000 context bytes/8,000 prompt characters; `[CLAUDE] claude-cli` and `[CODEX] codex-native` each permit up to 100 files/800,000 context bytes. Claude requires the readiness preflight and durable dispatcher; Codex requires fresh native usage state. Local targets are manually serialized. A fallback is used only in the stated order with a recorded reason; a live worker is not silently killed or rerouted.

The `[CODEX] execution_coordinator` owns the single product PR and all execution dispatch/diff checks. Workers do not create branches or PRs. Every worker may edit only its unit's exact product allowlist; any other changed path is a scope failure. Prospective commands below are not evidence that Routing ran them.

## Execution units

### EU-01 — Market-stream membership accounting

- **Source / purpose:** `WORK-001`; membership-gate each unsubscribe/removal so non-member, absent-symbol, and repeated removal cannot alter retained or upstream interest.
- **Dependencies / acceptance:** none; `REQ-MKT-001(a–c)`, `REQ-EVD-007(a)`. May run in parallel with EU-02 after the execution gate; no multi-instance, gateway, or provider-protocol claim.
- **Allowed files:** `apps/market-service/src/services/MarketStreamManager.ts`; `apps/market-service/tests/services/MarketStreamManager.test.ts`.
- **Validation:** `npm run build --workspace @atra/market-service`; `npm run test --workspace @atra/market-service -- tests/services/MarketStreamManager.test.ts`.
- **Pool / effort:** `[LOCAL] mac-ollama` → `[WINDOWS] windows-4070` → `[CODEX] codex-native` → `[CLAUDE] claude-cli`; low. **Checkpoint:** evidence for non-member, one-of-two, final-member transitions. **Escalate:** need to alter gateway/adapter/transport or claim multi-instance behavior.

### EU-02 — Direct freshness-aware prices contract

- **Source / purpose:** `WORK-002`; direct `/prices` per-result ticker plus `fresh`/`stale` success representation, with unavailable/error distinct from stale success.
- **Dependencies / acceptance:** none; `REQ-API-002(a–d)`, `REQ-EVD-007(a)`. May run with EU-01 after the execution gate. Direct breaking change is approved; no compatibility, versioned, parallel, or negotiated response path.
- **Allowed files:** `apps/market-service/src/services/PriceService.ts`; `apps/market-service/src/rest/router.ts`; `apps/market-service/tests/services/PriceService.test.ts`; `apps/market-service/tests/rest/router.test.ts`.
- **Validation:** `npm run build --workspace @atra/market-service`; `npm run test --workspace @atra/market-service -- tests/services/PriceService.test.ts tests/rest/router.test.ts`.
- **Pool / effort:** `[LOCAL] mac-ollama` → `[CODEX] codex-native` → `[CLAUDE] claude-cli`; low/medium. `[WINDOWS]` is ineligible because four files exceed its limit. **Checkpoint:** record exact response shape. **Escalate:** discovered consumer, required compatibility, cache-adapter, or provider change.

### EU-03 — No-data incompatibility-preflight record

- **Source / purpose:** `WORK-006`; establish the required stop/continue basis for existing incompatible role/owner data without querying or changing data, because `DEC-0004@v1` confirms no applicable population exists.
- **Dependencies / acceptance:** none after the human decision; it is the hard-gate predecessor of EU-04's `WORK-007` invariant migration. Maps `REQ-DATA-006(c)` and `REQ-EVD-007(c)`.
- **Allowed product files:** none.
- **Validation:** execution coordinator records the human no-data decision, confirmed environment identity, and `NO_EXISTING_APPLICABLE_DATA`; it must not run a data operation or represent this as production/population compatibility. No mock substitute is relevant.
- **Pool / effort:** `[CODEX] codex-native` → `[CLAUDE] claude-cli`; medium. **Checkpoint:** hard-gate outcome recorded before EU-04 migration work. **Escalate/stop:** evidence of any population, uncertainty about the decision/environment, production endpoint, or any request to import/repair/delete/backfill.

### EU-04 — Coordinated session-wallet and role-invariant migration

- **Source / purpose:** `WORK-005` + `WORK-007`; coordinate the shared database migration stream to persist signing-wallet session attribution, preserve it through refresh/auth/audit/revocation lifecycle, and enforce role/owner invariants at PostgreSQL commit boundary.
- **Dependencies / acceptance:** EU-03's no-data preflight outcome; precedes EU-05 acceptance and EU-06 evidence. Maps `REQ-SES-005(a–d)`, `REQ-DATA-006(a–b)` implementation prerequisites, `REQ-EVD-007(a,c)`. Migration proof remains EU-06.
- **Allowed files:** `apps/auth-service/src/modules/auth/services/SessionService.ts`; `apps/auth-service/src/modules/auth/services/TokenService.ts`; `apps/auth-service/src/modules/auth/repositories/SessionRepository.ts`; `apps/auth-service/src/middleware/authenticate.ts`; `apps/auth-service/src/types/express.d.ts`; `apps/auth-service/src/modules/roles/services/RoleService.ts`; `apps/auth-service/src/modules/recovery/services/RecoveryService.ts`; `packages/database/src/schema/sessions.ts`; `packages/database/src/schema/accounts.ts`; `packages/database/src/schema/accountWalletRoles.ts`; `packages/database/migrations/0000_wf0002_session_wallet_role_invariants.sql`; `packages/database/migrations/meta/_journal.json`; `packages/database/migrations/meta/0000_snapshot.json`; `apps/auth-service/tests/auth/SessionService.test.ts`; `apps/auth-service/tests/middleware/authenticate.test.ts`; `apps/auth-service/tests/roles/RoleService.test.ts`; `apps/auth-service/tests/recovery/RecoveryService.test.ts`; `packages/database/tests/schema/sessions.test.ts`; `packages/database/tests/schema/accounts.test.ts`; `packages/database/tests/schema/accountWalletRoles.test.ts`; `packages/database/tests/role-owner-invariants.integration.test.ts`.
- **Validation:** `npm run build --workspace @atra/database`; `npm run build --workspace @atra/auth-service`; `npm run test --workspace @atra/database`; `npm run test --workspace @atra/auth-service -- tests/auth/SessionService.test.ts tests/middleware/authenticate.test.ts tests/roles/RoleService.test.ts tests/recovery/RecoveryService.test.ts`; after gate and EU-03 only, `npm run db:migrate --workspace @atra/database` against the ENV-0001 connection.
- **Pool / effort:** `[CLAUDE] claude-cli` → `[CODEX] codex-native`; high; local targets exceed hard limits. **Checkpoint:** agree migration names and Role/Recovery service boundary with EU-05; separately record migration creation/application. **Escalate:** need for production data action, migration output outside allowlist, inability to retain the signing wallet, or no exactly-one-owner commit guarantee.

### EU-05 — Atomic nonce consumption and session-derived authority

- **Source / purpose:** `WORK-003` + `WORK-004`; one conditional transactional nonce-consume primitive for all specified flows and controller/route authority exclusively from the persisted authenticated session actor.
- **Dependencies / acceptance:** EU-04 must complete; service boundary is coordinated with EU-04; EU-06 depends on this primitive. Maps `REQ-AUTH-003(a–d)`, `REQ-NONCE-004(a–c)` implementation prerequisites, `REQ-SES-005(c)`, `REQ-EVD-007(a)`; competing-write nonce proof remains EU-06.
- **Allowed files:** `apps/auth-service/src/modules/identity/services/NonceService.ts`; `apps/auth-service/src/modules/identity/repositories/NonceRepository.ts`; `apps/auth-service/src/modules/identity/services/AccountService.ts`; `apps/auth-service/src/modules/wallets/services/WalletLinkingService.ts`; `apps/auth-service/src/modules/roles/services/RoleService.ts`; `apps/auth-service/src/modules/recovery/services/RecoveryService.ts`; `apps/auth-service/src/modules/wallets/controllers/WalletController.ts`; `apps/auth-service/src/modules/roles/controllers/RoleController.ts`; `apps/auth-service/src/modules/recovery/controllers/RecoveryController.ts`; `apps/auth-service/src/modules/auth/controllers/AuthController.ts`; `apps/auth-service/src/modules/auth/routes/authRoutes.ts`; `apps/auth-service/src/index.ts`; `apps/auth-service/tests/services/NonceService.test.ts`; `apps/auth-service/tests/services/AccountService.test.ts`; `apps/auth-service/tests/wallets/WalletLinkingService.test.ts`; `apps/auth-service/tests/wallets/WalletController.test.ts`; `apps/auth-service/tests/roles/RoleService.test.ts`; `apps/auth-service/tests/roles/RoleController.test.ts`; `apps/auth-service/tests/recovery/RecoveryService.test.ts`; `apps/auth-service/tests/recovery/RecoveryController.test.ts`; `apps/auth-service/tests/auth/AuthController.test.ts`; `apps/auth-service/tests/nonce-atomicity.integration.test.ts`.
- **Validation:** `npm run build --workspace @atra/auth-service`; `npm run test --workspace @atra/auth-service -- tests/services/NonceService.test.ts tests/services/AccountService.test.ts tests/wallets/WalletLinkingService.test.ts tests/wallets/WalletController.test.ts tests/roles/RoleService.test.ts tests/roles/RoleController.test.ts tests/recovery/RecoveryService.test.ts tests/recovery/RecoveryController.test.ts tests/auth/AuthController.test.ts`.
- **Pool / effort:** `[CLAUDE] claude-cli` → `[CODEX] codex-native`; high; local targets exceed hard limits. **Checkpoint:** agreed Role/Recovery signature and evidence that body identity cannot select actor. **Escalate:** public revoke, an in-scope action outside allowlist, body-authority fallback, non-transactional multi-nonce behavior, or another required path.

### EU-06 — Disposable PostgreSQL concurrency and invariant evidence

- **Source / purpose:** `WORK-008`; apply EU-04 migration only to ENV-0001 and observe actual competing nonce and role/owner writes.
- **Dependencies / acceptance:** EU-03 no-data outcome; EU-04 migration complete/applied; EU-05 primitive complete; environment remains live/loopback/disposable. Maps `REQ-NONCE-004(a)`, `REQ-DATA-006(a–b)`, `REQ-EVD-007(a,c)`.
- **Allowed product files:** none; it uses integration evidence created in EU-04/EU-05 and writes results only to later Execution/Test evidence.
- **Validation:** confirm ENV-0001 guardrails, run `npm run db:migrate --workspace @atra/database` with its reference, then the allowlisted integration evidence. Record migration application, at most one nonce success/result, rejected duplicate/second-owner/second-recovery writes, and committed exactly-one-consistent-owner provisioning/transfer outcomes.
- **Pool / effort:** `[CODEX] codex-native` → `[CLAUDE] claude-cli`; high; local targets are not suitable. **Checkpoint:** record environment classification and each observed outcome. **Escalate/stop:** unavailable/non-disposable/production endpoint, migration failure, accepted invariant violation, more than one nonce outcome, or inability to independently reproduce. Mock-only success is not a pass.

## Dependency, PR, Test, and Review plan

The acyclic implementation graph is `EU-03 → EU-04 → EU-05 → EU-06`; EU-01 and EU-02 are independent and may execute in parallel after the gate. EU-04/EU-05 are serialized due to shared service/migration interface and migration metadata. The coordinator records fresh dispatch health/usage, actual target, allowlist audit, fallback reason, elapsed time, and checkpoint for every unit.

After approved execution and implementation/Test evidence, the `[CODEX] execution_coordinator` opens or updates exactly one unmerged product PR from verified `workflow/WF-0002-atra-evaluation-remediation` to verified `main`, recording repository, base, head, commit, URL, and state. No merge or force-push is allowed.

Formal Test uses `test_engineer` `[CODEX]` preferred, `[CLAUDE]` fallback, independent from implementation. It maps all six requirement IDs to observed product behavior and database evidence, explicitly preserving live-provider, multi-instance, and client limitations. Independent Review uses `code_reviewer` `[CLAUDE]` preferred, `[CODEX]` fallback, preferably opposite the EU-04/EU-05 implementer; it reviews the actual PR, Test Report, migration safety, authority derivation, atomicity, lifecycle behavior, and residual risks without editing. The controller separately owns the required artifact PR.

## Self-check and request

All eight `DEC-0002@v1` work units are routed through six bounded execution units; exact product allowlists, dependencies, acceptance mappings, validation commands, checkpoints, escalation conditions, eligible target pools/fallbacks, PR ownership, and independent Test/Review are recorded. The hard data-preflight gate is preserved: `DEC-0004@v1` resolves it only as a no-existing-data decision, while EU-04/EU-06 retain migration and actual database evidence. The previously blocking head and environment facts are now verified and no unresolved routing fact remains.

`ROUTE-0002@v2` requests controller validation and **READY_FOR_IMPLEMENTATION**. If validation passes, the next action is the required concise execution-gate summary and explicit human approval of this exact routing version; implementation must not begin before that approval.
