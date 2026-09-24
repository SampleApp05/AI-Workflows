---
artifact:
  id: ROUTE-0002
  type: ROUTING_PLAN
  version: 1
  status: BLOCKED
workflow:
  id: WF-0002
  project: Atra
  technology: Cross-Stack
  feature: atra-evaluation-remediation
  mode: delivery
  contract_version: "1.2"
lineage:
  parents:
    - DEC-0002@v1
    - REQ-0002@v1
    - ARC-0002@v2
    - DEF-0002@v1
    - HAND-0002@v1
    - HSUM-0002@v1
    - DEC-0001@v1
    - OPLAN-0002@v1
  references:
    - path: work_planner/decomposition-v1.md
    - path: requirements_engineer/requirements-v1.md
    - path: solution_architect/architecture-v2.md
    - path: problem_analyst/definition-v1.md
    - path: intake_analyst/handoff-v1.md
    - path: intake_analyst/handoff-summary-v1.md
    - path: workflow_orchestrator/architecture-decisions-v1.md
    - path: workflow_orchestrator/orchestration-plan-v1.md
    - path: workflow_orchestrator/validations/DEC-0002-v1.md
ownership:
  created_by: execution_router
  performed_by: execution_router
  recorded_by: execution_router
  engine: codex
  target_id: codex-native
---

# Routing Plan — Atra evaluation remediation

## Status and binding scope

`ROUTE-0002@v1` is the Routing artifact for `WF-0002`, governed by Contract v1.2. It derives only from validated `DEC-0002@v1`, validated requirements and architecture, the approved decision record `DEC-0001@v1`, and approved `OPLAN-0002@v1`. It creates no product change, branch, pull request, configuration change, database connection, test run, or worker dispatch.

The plan covers only the six approved remediations and their delivery evidence: membership-gated market unsubscribe accounting (`REQ-MKT-001`); a direct freshness-aware `/prices` contract (`REQ-API-002`); session-derived sensitive-action authority (`REQ-AUTH-003`); atomic nonce consumption (`REQ-NONCE-004`); durable session-wallet attribution and selected lifecycle revocations (`REQ-SES-005`); and database role/owner integrity (`REQ-DATA-006`). `REQ-EVD-007` and `REQ-DEL-008` are realized through the specified evidence, independent Test/Review, gate, and PR route.

The plan is **BLOCKED** and does **not** request `READY_FOR_IMPLEMENTATION`. The execution units below are bounded routing information only. The blockers in the next section must be resolved and the artifact rerouted/validated before an execution-gate summary can truthfully identify an exact approved product branch and verification environment.

Unchanged exclusions are package-manager work; audit pagination; multi-instance market behavior; iOS/client/provider/portfolio/Phase-2.5 work; deployment, merge, release, production-data operations, and every capability not necessary for these six findings.

## Fresh product and PR-route observation

Observed read-only on 2026-09-24 from `/Users/danielvelikov/Developer/Atra-Services`:

| Fact | Observation | Status |
| --- | --- | --- |
| Product Git root | `/Users/danielvelikov/Developer/Atra-Services` | verified |
| Repository / origin fetch and push URL | `https://github.com/SampleApp05/Atra-Services.git` / same | verified |
| Base branch / baseline | `main` at `be51242593fe6643aff11c491d4f8a6c0835aed5` (`be51242 chore: ignore local agent skills (#2)`) | verified; matches recorded intake baseline |
| Working tree | `main...origin/main`, clean; local and remote `main` resolve to the same commit | verified |
| PR authority | authenticated `SampleApp05` GitHub principal has `ADMIN` permission on `SampleApp05/Atra-Services`; token reports `repo` scope | verified capability, not a created PR |
| Dedicated workflow branch / head | manifest product head is `null`; no remote or local branch matching `workflow/WF-0002-atra-evaluation-remediation`, `wf/WF-0002-atra-evaluation-remediation`, or `codex/WF-0002-atra-evaluation-remediation` exists | **missing / blocker** |
| Existing WF-0002 product PR | no PR exists for the candidate `workflow/WF-0002-atra-evaluation-remediation` head | verified absent; no PR is invented |

Contract v1.1/1.2 requires Routing to record a verified dedicated product workflow branch before product-changing routes can proceed. The manifest records product-PR owner `execution_coordinator` and base `main`, but has no head. It is not Routing's authority to create a branch or select an unrecorded head name. Therefore this artifact cannot provide the required verified product head or route an implementation worker to edit the product repository.

### Verification-environment observation

The human authorization in `DEC-0001@v1` permits a disposable, non-production PostgreSQL environment for migration and competing-write evidence only. No endpoint, credential reference, environment identity, database name, or representativeness record is supplied to this Routing stage; `DATABASE_URL`, `PGHOST`, and `PGDATABASE` are absent in the observed shell. This is a second routing blocker for `EU-03`, `EU-04`, and `EU-06`: it is impossible to verify the required preflight, migration, or database-backed nonce/invariant evidence without guessing a connection or treating mock-only evidence as sufficient.

Resolving either blocker must not authorize a production connection, data repair, deletion, backfill, deployment, merge, or release. The execution coordinator must record the selected branch and the authorized disposable environment identity/guardrails before relevant dispatches; the router must then issue a corrected version and controller validation must pass before the distinct human execution gate.

## Target pools and dispatch rules

Every implementation unit is owned by `implementation_worker`; selection remains a dispatch-time decision after fresh target health, capacity, availability, workspace-host, and scope checks. Target labels below are registry labels, not locks or model names.

- `[LOCAL] mac-ollama` is eligible only for a unit that fits all hard limits (at most six files, 64,000 context bytes, 16,000 prompt characters), is independently verifiable, and has manual single-task availability.
- `[WINDOWS] windows-4070` is eligible only for a unit that fits all hard limits (at most three files, 32,000 context bytes, 8,000 prompt characters), is independently verifiable, and has manual single-task availability.
- `[CLAUDE] claude-cli` and `[CODEX] codex-native` are eligible for implementation work within their registered 100-file/800,000-byte limits. Claude requires the recorded readiness preflight and durable dispatcher; Codex requires a fresh usage check and a native subagent.
- A worker is never silently rerouted. A fallback is permitted only in the listed order after a recorded health/capacity/availability reason. A slow live worker stays live unless the human directs otherwise. All work occurs in the verified Mac workspace Git root.
- The `execution_coordinator` is `[CODEX] codex-native` only and owns one product PR for Atra-Services. Individual workers create no product branch or PR.

## Execution units

The following file allowlists are repository-relative to the verified Atra-Services Git root. A worker may edit no other product path. All commands are prospective validation commands; Routing has not run them.

### EU-01 — Market-stream membership accounting

- **Source work unit / purpose:** `WORK-001`; correct the manager's membership-removal transition and prove the three in-process upstream-interest cases.
- **Dependencies:** none. May run in parallel only after the branch blocker is resolved and the execution gate approves this exact route.
- **Allowed product files:**
  - `apps/market-service/src/services/MarketStreamManager.ts`
  - `apps/market-service/tests/services/MarketStreamManager.test.ts`
- **Acceptance mapping:** `REQ-MKT-001(a–c)`, `REQ-EVD-007(a)`; explicitly no multi-instance, gateway-protocol, or provider-ownership claim.
- **Validation:** `npm run build --workspace @atra/market-service`; `npm run test --workspace @atra/market-service -- tests/services/MarketStreamManager.test.ts`; then the formal Test stage runs the relevant suite independently.
- **Preferred pool / fallback:** `[LOCAL] mac-ollama` (two-file, independently testable targeted edit) → `[WINDOWS] windows-4070` (two files fits its limit) → `[CODEX] codex-native` → `[CLAUDE] claude-cli`.
- **Advisory effort:** low. **Checkpoint:** show the tested non-member, one-of-two-member, and final-member transitions before integration. **Escalate:** any need to change gateway, Binance adapter, transport, or anything claiming multi-instance behavior.

### EU-02 — Direct freshness-aware prices contract

- **Source work unit / purpose:** `WORK-002`; replace the bare result body with per-ticker data plus `fresh`/`stale`, while retaining unavailable/error as a non-success outcome.
- **Dependencies:** none. File-disjoint from EU-01 and may run in parallel after the branch blocker and execution gate.
- **Allowed product files:**
  - `apps/market-service/src/services/PriceService.ts`
  - `apps/market-service/src/rest/router.ts`
  - `apps/market-service/tests/services/PriceService.test.ts`
  - `apps/market-service/tests/rest/router.test.ts`
- **Acceptance mapping:** `REQ-API-002(a–d)`, `REQ-EVD-007(a)`; the deliberately breaking direct contract is authorized, but no compatibility/parallel/versioned response path may be introduced.
- **Validation:** `npm run build --workspace @atra/market-service`; `npm run test --workspace @atra/market-service -- tests/services/PriceService.test.ts tests/rest/router.test.ts`; formal Test independently reruns and observes fresh, stale, unavailable/error, and mixed-result states.
- **Preferred pool / fallback:** `[LOCAL] mac-ollama` (four-file, independently testable localized change) → `[CODEX] codex-native` → `[CLAUDE] claude-cli`. `[WINDOWS]` is ineligible because the four-file allowlist exceeds its hard limit.
- **Advisory effort:** low/medium. **Checkpoint:** agree and record the exact direct response shape in the execution record before service/router edits land. **Escalate:** a discovered consumer, required contract versioning, cache-adapter alteration, or provider behavior change.

### EU-03 — Read-only role/owner incompatibility preflight

- **Source work unit / purpose:** `WORK-006`; inspect only the approved representative non-production data for duplicate `(account, wallet, role)` rows, multiple `OWNER`/`RECOVERY` rows, and canonical owner mismatch.
- **Dependencies:** branch and disposable-environment blockers resolved; no product-file dependency. It may run before all edits and is the hard gate for EU-04's role/owner migration portion.
- **Allowed product files:** none. This unit records query identity, environment identity/classification, dataset representativeness, timestamp, and aggregate/per-account evidence in the later Execution record only; it must not write a product query script or data artifact.
- **Acceptance mapping:** `REQ-DATA-006(c)`, `REQ-EVD-007(c)`.
- **Validation:** read-only SQL transaction(s) prepared and reviewed by the execution coordinator; verify the connected environment is disposable and non-production before issuing any query; capture no credentials in artifacts. No mocked replacement is valid.
- **Preferred pool / fallback:** `[CODEX] codex-native` → `[CLAUDE] claude-cli`, due to security-sensitive environment classification and evidence handling. Local targets are not selected because the unit is not a bounded code edit.
- **Advisory effort:** medium. **Checkpoint:** `NO_INCOMPATIBILITY_FOUND` or `INCOMPATIBILITY_FOUND` with evidence. **Escalate/stop:** any incompatible record, uncertainty about representativeness, a production endpoint, unavailable environment, or request to repair/backfill/delete. `INCOMPATIBILITY_FOUND` blocks EU-04's invariant migration and EU-06's invariant evidence pending a separately authorized human decision.

### EU-04 — Coordinated session-wallet and database-invariant migration

- **Source work units / purpose:** `WORK-005` plus `WORK-007`, combined because both alter the database schema/migration stream. Persist the signing wallet on sessions, use it for authentication/refresh/audit attribution, revoke affected sessions at the selected lifecycle events, and add the role/owner database guarantees after EU-03 clears.
- **Dependencies:** EU-03 must report `NO_INCOMPATIBILITY_FOUND` before the `WORK-007` migration is generated/applied. It may not proceed past design or claim the role/owner constraints without that hard-gate result. It must complete before EU-05 can satisfy session-derived authority and before EU-06 can produce database evidence.
- **Allowed product files:**
  - `apps/auth-service/src/modules/auth/services/SessionService.ts`
  - `apps/auth-service/src/modules/auth/services/TokenService.ts`
  - `apps/auth-service/src/modules/auth/repositories/SessionRepository.ts`
  - `apps/auth-service/src/middleware/authenticate.ts`
  - `apps/auth-service/src/types/express.d.ts`
  - `apps/auth-service/src/modules/roles/services/RoleService.ts`
  - `apps/auth-service/src/modules/recovery/services/RecoveryService.ts`
  - `packages/database/src/schema/sessions.ts`
  - `packages/database/src/schema/accounts.ts`
  - `packages/database/src/schema/accountWalletRoles.ts`
  - `packages/database/migrations/0000_wf0002_session_wallet_role_invariants.sql`
  - `packages/database/migrations/meta/_journal.json`
  - `packages/database/migrations/meta/0000_snapshot.json`
  - `apps/auth-service/tests/auth/SessionService.test.ts`
  - `apps/auth-service/tests/middleware/authenticate.test.ts`
  - `apps/auth-service/tests/roles/RoleService.test.ts`
  - `apps/auth-service/tests/recovery/RecoveryService.test.ts`
  - `packages/database/tests/schema/sessions.test.ts`
  - `packages/database/tests/schema/accounts.test.ts`
  - `packages/database/tests/schema/accountWalletRoles.test.ts`
  - `packages/database/tests/role-owner-invariants.integration.test.ts`
- **Acceptance mapping:** `REQ-SES-005(a–d)`, `REQ-DATA-006(a–b)` implementation prerequisites, `REQ-EVD-007(a,c)`. Exact database proof remains EU-06; no production compatibility claim is allowed.
- **Validation:** `npm run build --workspace @atra/database`; `npm run build --workspace @atra/auth-service`; `npm run test --workspace @atra/database`; `npm run test --workspace @atra/auth-service -- tests/auth/SessionService.test.ts tests/middleware/authenticate.test.ts tests/roles/RoleService.test.ts tests/recovery/RecoveryService.test.ts`; only after EU-03 and an authorized disposable endpoint: `npm run db:migrate --workspace @atra/database` under the execution coordinator's environment guard.
- **Preferred pool / fallback:** `[CLAUDE] claude-cli` (high-risk cross-service schema, transaction, session-lifecycle, and migration coordination) → `[CODEX] codex-native`. Local targets are ineligible because this twenty-one-file allowlist exceeds their hard limits.
- **Advisory effort:** high. **Checkpoint:** first agree the migration names and role/recovery service boundary with EU-05; then separately record migration generation, successful disposable-environment application, and lifecycle tests. **Escalate:** incompatible preflight result, migration needing production data handling, an unapproved migration filename/output, inability to preserve refresh signing-wallet attribution, or failure to provide exactly-one-owner semantics at commit.

### EU-05 — Atomic nonce consumption and session-derived controller authority

- **Source work units / purpose:** `WORK-003` plus `WORK-004`, combined to own their shared Role/Recovery service signature. Introduce one conditional transactional consume primitive for all in-scope nonce flows and make the listed controllers/routes derive acting authority exclusively from the authenticated session.
- **Dependencies:** EU-04 must complete before EU-05 is accepted, because the actor context must use the persisted session wallet. EU-04 and EU-05 must agree the `RoleService`/`RecoveryService` mutation signature before either implementation begins. EU-06 depends on EU-05's consume primitive.
- **Allowed product files:**
  - `apps/auth-service/src/modules/identity/services/NonceService.ts`
  - `apps/auth-service/src/modules/identity/repositories/NonceRepository.ts`
  - `apps/auth-service/src/modules/identity/services/AccountService.ts`
  - `apps/auth-service/src/modules/wallets/services/WalletLinkingService.ts`
  - `apps/auth-service/src/modules/roles/services/RoleService.ts`
  - `apps/auth-service/src/modules/recovery/services/RecoveryService.ts`
  - `apps/auth-service/src/modules/wallets/controllers/WalletController.ts`
  - `apps/auth-service/src/modules/roles/controllers/RoleController.ts`
  - `apps/auth-service/src/modules/recovery/controllers/RecoveryController.ts`
  - `apps/auth-service/src/modules/auth/controllers/AuthController.ts`
  - `apps/auth-service/src/modules/auth/routes/authRoutes.ts`
  - `apps/auth-service/src/index.ts`
  - `apps/auth-service/tests/services/NonceService.test.ts`
  - `apps/auth-service/tests/services/AccountService.test.ts`
  - `apps/auth-service/tests/wallets/WalletLinkingService.test.ts`
  - `apps/auth-service/tests/wallets/WalletController.test.ts`
  - `apps/auth-service/tests/roles/RoleService.test.ts`
  - `apps/auth-service/tests/roles/RoleController.test.ts`
  - `apps/auth-service/tests/recovery/RecoveryService.test.ts`
  - `apps/auth-service/tests/recovery/RecoveryController.test.ts`
  - `apps/auth-service/tests/auth/AuthController.test.ts`
  - `apps/auth-service/tests/nonce-atomicity.integration.test.ts`
- **Acceptance mapping:** `REQ-AUTH-003(a–d)`, `REQ-NONCE-004(a–c)` implementation prerequisites, `REQ-SES-005(c)`, and `REQ-EVD-007(a)`. The concurrent database proof for `REQ-NONCE-004(a)` is EU-06, not a mocked test claim.
- **Validation:** `npm run build --workspace @atra/auth-service`; `npm run test --workspace @atra/auth-service -- tests/services/NonceService.test.ts tests/services/AccountService.test.ts tests/wallets/WalletLinkingService.test.ts tests/wallets/WalletController.test.ts tests/roles/RoleService.test.ts tests/roles/RoleController.test.ts tests/recovery/RecoveryService.test.ts tests/recovery/RecoveryController.test.ts tests/auth/AuthController.test.ts`; EU-06 adds actual competing-write validation.
- **Preferred pool / fallback:** `[CLAUDE] claude-cli` (security-sensitive cross-controller/service transaction changes and twenty-two-file scope) → `[CODEX] codex-native`. Local targets are ineligible by registered hard limits.
- **Advisory effort:** high. **Checkpoint:** record the agreed service signature, then demonstrate all in-scope flows use the same conditional predicate and no body field can choose the actor. **Escalate:** an endpoint/action outside the approved families, a public revoke route, any fallback to body-selected authority, a non-transactional multi-nonce path, or needed edits outside the allowlist.

### EU-06 — Disposable PostgreSQL concurrency and invariant evidence

- **Source work unit / purpose:** `WORK-008`; apply the cleared migration only to the verified disposable environment and observe concurrent nonce consumption plus alternate/competing role and owner writes.
- **Dependencies:** EU-03 must have cleared the preflight; EU-04's migration must be complete and applied in the disposable environment; EU-05's nonce primitive must be complete; the disposable-environment blocker must be resolved. No product-file edits are authorized by this unit.
- **Allowed product files:** none. It consumes the exact test/integration files produced by EU-04/EU-05 and writes results only to the Execution/Test evidence owned by later stages.
- **Acceptance mapping:** `REQ-NONCE-004(a)`, `REQ-DATA-006(a–b)`, `REQ-EVD-007(a,c)`.
- **Validation:** environment guard confirms non-production/disposable status; `npm run db:migrate --workspace @atra/database`; then run the existing allowlisted integration evidence suites against that endpoint. Record no-more-than-one nonce success/result, migration application, rejected duplicate-role/second-owner/second-recovery writes, and committed exactly-one-consistent-owner provisioning/transfer outcomes. The formal independent Test stage repeats or independently validates the evidence where environment access remains available.
- **Preferred pool / fallback:** `[CODEX] codex-native` → `[CLAUDE] claude-cli`, with an independent Test target later selected opposite to the actual implementer when feasible. Local targets are not selected because the work is environment-sensitive evidence, not a narrow edit.
- **Advisory effort:** high. **Checkpoint:** capture the target environment classification before migration and each observed concurrent outcome after it. **Escalate/stop:** unavailable, persistent, or production environment; migration failure; more than one successful nonce outcome; accepted invariant violation; or inability to reproduce the evidence independently. Mocked-only success is a limitation/blocker, not a pass.

## Dependency and safe-parallelism plan

`EU-01` and `EU-02` are file-disjoint and independently startable. `EU-03` has no product-file dependency and should run first once a safe environment is recorded. `EU-04` follows the EU-03 hard gate for all role/owner migration work. `EU-05` follows EU-04 so controller authority uses persisted session-wallet attribution and the shared service interface remains coherent. `EU-06` follows EU-03, EU-04, and EU-05. This forms an acyclic graph:

`EU-03 → EU-04 → EU-05 → EU-06`, with `EU-01` and `EU-02` independent.

The execution coordinator must serialize EU-04 and EU-05 despite source-level interface overlap; it must also serialize any migration generation/application to avoid colliding schema snapshots or journal entries. It may coordinate EU-01/EU-02 concurrently only after each target is freshly checked and resource-constrained local workers are not already busy.

## Execution coordination, Test, and Review route

- **Product PR:** after the branch blocker is resolved, `execution_coordinator` on `[CODEX]` owns exactly one Atra-Services PR. Required facts to record before opening/updating it are the verified remote/repository, `main` base, resolved dedicated head, commit, PR URL, and open/unmerged state. No worker opens a separate PR; no merge or force-push is authorized.
- **Execution checkpointing:** the coordinator records fresh target usage/health and the actual selected target before each unit, checks the post-worker diff against the unit allowlist, preserves user changes, and records unit result/fallback/elapsed time. A scope violation or failed hard gate stops progression.
- **Formal Test:** `test_engineer` is preferably `[CODEX]`, fallback `[CLAUDE]`; it must be independent of the implementation worker and inspect the actual product PR/diff. It maps each `REQ-MKT-001` through `REQ-DATA-006` to observed tests, distinguishes mocked checks from database-backed evidence, and records any unavailable disposable environment or live-provider/multi-instance/client limitation.
- **Independent Review:** `code_reviewer` is preferably `[CLAUDE]`, fallback `[CODEX]`, and must not be the implementer. Prefer the engine opposite the worker that completed EU-04/EU-05. Review examines the actual PR, Test Report, requirement mapping, migration safety, controller authority derivation, atomicity evidence, lifecycle semantics, and residual scope/operational limits; it does not edit or merge.
- **Artifact PR:** remains controller-owned under Contract v1.1/1.2 and is separate from this product route. Completion requires both verified unmerged PRs plus passed Test and independent Review.

## Routing blockers, self-check, and requested next action

Coverage is complete: EU-01 through EU-06 collectively route `WORK-001` through `WORK-008`; every product requirement has an acceptance mapping; the data preflight remains a hard gate; every product-changing unit has an exact repository-relative file allowlist, dependencies, prospective validation, checkpoint/escalation conditions, eligible target pool, and ordered fallback. The plan retains every exclusion, uses registry labels and limits, preserves execution-coordinator PR ownership, and defines independent Test/Review.

However, Contract-required product head verification is absent, and the authorized disposable PostgreSQL environment is not concretely identifiable or configured. These are factual routing blockers, not assumptions to be resolved by an implementation worker. Therefore this artifact requests **BLOCKED** routing validation, not `READY_FOR_IMPLEMENTATION`.

To unblock, the workflow controller needs to record a dedicated Atra-Services head branch under execution-coordinator ownership and a credential-safe reference to a disposable, non-production PostgreSQL environment (including confirmation of its preflight data's representativeness). After those facts are verified, issue a bounded routing revision and controller validation; only then present the required concise execution summary and request human approval of that exact route.
