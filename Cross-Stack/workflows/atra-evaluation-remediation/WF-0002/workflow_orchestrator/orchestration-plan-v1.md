---
artifact:
  id: OPLAN-0002
  type: ORCHESTRATION_PLAN
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
    - HAND-0002@v1
    - HSUM-0002@v1
ownership:
  created_by: workflow_orchestrator
  performed_by: workflow_orchestrator
  recorded_by: workflow_orchestrator
  engine: codex
  target_id: codex-native
---

# Orchestration Plan — Atra evaluation remediation

## Identity and proposed scope

- **Workflow:** `WF-0002`
- **Contract:** v1.2
- **Mode:** `delivery` (fixed if this plan is approved)
- **Product scope:** `Atra-Services` only; prior baseline `main` / `be51242593fe6643aff11c491d4f8a6c0835aed5` must be refreshed and reverified before implementation claims or edits.
- **Change scope:** only the six verified WF-0001 findings: stream unsubscribe accounting, cache-freshness visibility, session-derived authorization for sensitive actions, atomic nonce consumption, durable authenticating-wallet attribution, and database role invariants.
- **Exclusions:** package-manager migration; audit pagination; multi-instance behavior; iOS/client/provider/portfolio/Phase-2.5 expansion; deployment, merge, release, and production-data operations.

## Ordered stage chain

1. **Handoff** — `HAND-0002@v1` and `HSUM-0002@v1`, validated.
2. **Human plan gate** — approval of this exact `OPLAN-0002@v1`.
3. **Definition** — solution-independent desired outcomes, constraints, risks, and observable success criteria for the six remediations.
4. **Architecture** — assess current market, authentication/session, and database boundaries; recommend a bounded technical direction and surface any human decision that changes API or data compatibility.
5. **Requirements** — create traceable, testable behavioral, interface, persistence, security, and migration requirements.
6. **Decomposition** — form bounded, dependency-aware Work Units without selecting workers.
7. **Routing** — form safe Execution Units, verify product-PR branch/authority, choose eligible target pools and fallbacks, and define independent Test/Review evidence.
8. **Human execution gate** — approval of the exact Routing Plan and its execution scope.
9. **Execution** — apply only approved, verified units in `Atra-Services`; the execution coordinator owns the one product PR.
10. **Test** — independently verify requirements and actual product changes; add or run tests as authorized by the approved units.
11. **Review** — independently review the product PR, Test Report, and requirements; no self-review by an implementer.
12. **Run Report** — verify required product and artifact pull requests and publish the controller-owned artifact PR. No merge.

## Proposed assignments and approved fallbacks

| Stage | Role | Preferred target | Approved fallback | Scope / rationale |
| --- | --- | --- | --- | --- |
| Definition | problem_analyst | codex-native | claude-cli | Bounded source-to-outcome definition with controller-adjacent artifact validation. |
| Architecture | solution_architect | claude-cli | codex-native | Independent assessment of security, session, API, migration, and stream-boundary trade-offs. |
| Requirements | requirements_engineer | codex-native | claude-cli | Traceable testable contract from the validated direction. |
| Decomposition | work_planner | claude-cli | codex-native | Independent dependency and file-overlap analysis. |
| Routing | execution_router | codex-native | claude-cli | Verify product/PR facts, scope, worker pools, and safety before execution. |
| Execution coordination | execution_coordinator | codex-native | none | Controller-compatible product-PR ownership and recorded dispatch evidence. |
| Implementation units | implementation_worker | selected after Routing from healthy eligible registered targets | selected and recorded per Routing Plan | Exact worker is intentionally not fixed before file scope, dependencies, risks, and capacity are known. |
| Test | test_engineer | codex-native | claude-cli | Formal validation must remain independent from the implementation worker. |
| Review | code_reviewer | claude-cli | codex-native | Prefer an opposite-engine reviewer when practical; never an implementer. |
| Run Report | workflow_orchestrator | codex-native | none | Controller-owned state, artifact publication, and final evidence. |

No target is dispatched by this plan. Before every actual dispatch, the controller will refresh target health and capacity; Claude work, if selected, will use its required readiness preflight and durable launcher. A fallback requires a recorded reason and may be used only if this plan authorizes it.

## Expected artifacts and validation

Expected v1 artifacts unless a bounded same-stage correction is needed before validation: `DEF-0002`, `ARC-0002`, `REQ-0002`, `DEC-0002`, `ROUTE-0002`, execution records/results, `TEST-0002`, `REV-0002`, and `WRUN-0002`. The controller records each validation and event. Human approval is required for the exact plan version and, later, the exact routing-plan version; no approval is inferred from silence.

## Evidence and baseline approach

- Treat `WRUN-0001@v3` and `TEST-0001@v1` as evidence of the six findings, not as a delivery design or an expanded backlog.
- Refresh `Atra-Services/main`, record the exact commit and working-tree state, and stop for a material baseline divergence that changes the approved scope.
- Inspect code and tests only as required by the active stage. Do not modify product code until the Routing Plan passes the second human gate.
- Verify artifact repository remote/base/head and PR authority before artifact publication; Routing must independently verify the product remote/base/head/PR authority before any implementation.
- Preserve user-owned untracked files and unrelated working-tree changes; no destructive Git operation, merge, force-push, deployment, or production-data operation is in scope.

## Risks, dependencies, and authority checks

1. **Authorization/API compatibility:** deriving trusted action authority from session context can invalidate current request-body contracts. Architecture and Requirements must surface compatibility choices rather than assume them.
2. **Data integrity/migration:** role constraints may conflict with existing data or a deployed schema. No production-data operation is authorized; migration feasibility and incompatible data are blockers or explicit human decisions.
3. **Security/concurrency evidence:** atomic nonce consumption and relational invariants need database-backed verification. Mocked unit tests are insufficient evidence by themselves.
4. **Session semantic change:** persisting an authenticating wallet must be reconciled with multi-wallet account behavior and refresh/revocation semantics without inventing authority rules.
5. **Market contract:** exposing fresh/stale/unavailable state requires a stable consumer contract; compatibility belongs in Architecture/Requirements.
6. **PR dependency:** one verified product PR and one verified artifact PR are required before completion. If repository identity, branch, or authorization cannot be verified, Routing or publication blocks.
7. **Source limits:** iOS, live-provider, multi-instance, broader chain portability, and other deferred findings remain out of scope even if inspection encounters them.

## Gates and next action

This is the mandatory first delivery gate. Approval fixes the workflow mode, six-finding scope, ordered stages, target/fallback intentions, and the requirement for a second execution gate. A material pre-approval change creates `OPLAN-0002@v2`. No Definition work may start before the human decision is recorded.
