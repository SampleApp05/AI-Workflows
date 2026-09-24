---
artifact:
  id: OPLAN-0001
  type: ORCHESTRATION_PLAN
  version: 1
  status: DRAFT
workflow:
  id: WF-0001
  project: Atra
  technology: Cross-Stack
  feature: phase-foundation-evaluation
  mode: evaluation
lineage:
  parents:
    - HAND-0001@v1
    - HSUM-0001@v1
ownership:
  created_by: workflow_orchestrator
  performed_by: workflow_orchestrator
  recorded_by: workflow_orchestrator
  engine: codex
  target_id: codex-native
---

# Orchestration Plan — Atra foundational-phase evaluation

## Identity and approved-scope candidate

- **Workflow:** `WF-0001`
- **Contract:** v1.2
- **Mode:** `evaluation` (fixed if this plan is approved)
- **Subject:** read-only evaluation of Atra Phases 1, 2, and 2.5 against the authoritative intended-state material in *Current Atra Phases Summary*.
- **Recorded product scope:** `Atra-Services`, `main`; the exact baseline commit must be observed and recorded before any implementation-state conclusion.
- **Product impact:** none. Product edits, implementation, routing, execution, and a product PR are excluded.

## Ordered stage chain

1. Handoff — completed as `HAND-0001@v1` with `HSUM-0001@v1`.
2. Human plan gate — approval of this exact `OPLAN-0001@v1`.
3. Definition — express the comparison problem, boundaries, observable outcomes, evidence limits, and risks without prescribing solutions.
4. Architecture — inspect existing architecture only as needed; determine whether its boundaries satisfy the intended foundational phases and record findings without redesigning it.
5. Requirements — turn the approved evaluation scope into traceable, verifiable comparison criteria, preserving source uncertainties.
6. Read-only Evaluation Test — inspect/run existing safe tests where available; compare the verified baseline against requirements and report coverage, failures, gaps, edge cases, and uncertainty. No test or fixture changes.
7. Read-only Evaluation Review — independently review the existing baseline and Test Report for actionable defects, security/data risks, patterns, gaps, and residual risk. No edits.
8. Run Report — synthesize validated evidence; create or update exactly one artifact-repository pull request. Product pull request remains `NOT_APPLICABLE` with no-edit evidence.

## Proposed assignments and fallbacks

| Stage | Role | Preferred target | Approved fallback | Rationale |
| --- | --- | --- | --- | --- |
| Definition | problem_analyst | codex-native | claude-cli | Source-bound scope analysis and close controller coordination. |
| Architecture | solution_architect | claude-cli | codex-native | Stage preference and independent technical assessment of the existing system. |
| Requirements | requirements_engineer | codex-native | claude-cli | Traceability from the approved intended state, with explicit evidence limitations. |
| Evaluation Test | test_engineer | codex-native | claude-cli | Read-only test and acceptance-coverage analysis; target health/usage will be refreshed at dispatch. |
| Evaluation Review | code_reviewer | claude-cli | codex-native | Independent review from the preferred Test engine; preserves reviewer independence. |
| Run Report | workflow_orchestrator | codex-native | none | Controller-owned synthesis and artifact publication. |

No target is dispatched by this plan. Health, capacity, usage, and final target choice are checked and recorded immediately before each dispatch. Claude work, if selected, uses the recorded durable dispatcher path; native Codex work uses native subagents.

## Artifacts and validation

Expected artifacts: `DEF-0001`, `ARC-0001`, `REQ-0001`, `TEST-0001`, `REV-0001`, and `WRUN-0001`, each at v1 unless a bounded same-stage revision is required before validation. The controller writes validation records and appends workflow events. Every finding must distinguish source fact, verified baseline evidence, assumption, inference, and unknown.

## Evidence and baseline approach

- Resolve the exact `Atra-Services` `main` commit and checkout state before recording implementation findings.
- Inspect only the recorded product scope unless a verified, in-scope repository is added through a human-approved plan revision.
- Use the source chat as intended-state authority, never the repository as an implicit specification.
- Record detailed Phase 2.5 criteria unavailable from the source as an evidence gap; do not infer them.
- Run only existing safe, deterministic tests in evaluation mode and record environment limitations rather than changing test assets.

## Risks, dependencies, and blockers

1. **Source completeness risk (material):** the available source response is truncated, leaving detailed Phase 2.5 success criteria unavailable. The run can assess only visible Phase 2.5 chain-awareness intent and must retain this uncertainty.
2. **Scope risk (material):** `Atra-Services` is the only recorded product repository although the source describes an iOS-first product. The run will not make iOS implementation claims without a verified scope addition.
3. **Baseline risk:** no baseline commit is presently recorded. If `main` cannot be inspected or a baseline cannot be verified, later stages must report the gap rather than assume one.
4. **Read-only authority:** defects may be found but cannot be fixed in this run. Material findings become inputs to a separately approved delivery workflow.
5. **Artifact publication dependency:** one artifact PR is required at completion; remote identity and branch are verified, while PR-creation authority and URL remain to be verified at publication.

## Gates and next action

This is the sole human gate for evaluation mode. Approval fixes the mode, scope, ordered stages, and proposed target/fallback set. There is no execution gate. A material change before approval creates `OPLAN-0001@v2`; no Definition work may begin before an approval record identifies this exact version.
