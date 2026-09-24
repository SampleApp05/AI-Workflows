---
artifact:
  id: WRUN-0002
  type: WORKFLOW_RUN_REPORT
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
    - OPLAN-0002@v1
    - HAPP-0002@v1
    - DEF-0002@v1
    - ARC-0002@v2
    - REQ-0002@v1
    - DEC-0002@v1
    - ROUTE-0002@v2
    - HAPP-0003@v1
    - EXEC-0002@v1
    - TEST-0002@v2
    - REV-0002@v1
ownership:
  created_by: workflow_orchestrator
  performed_by: workflow_orchestrator
  recorded_by: workflow_orchestrator
  engine: codex
  target_id: codex-native
---

# Workflow Run Report — WF-0002

## Final status and approved scope

**COMPLETE, with recorded publication and evidence warnings.** This delivery run remediated the six verified WF-0001 findings in Atra-Services: market subscription accounting, freshness-aware direct `/prices`, session-derived authority, atomic nonce use, session-wallet lifecycle, and PostgreSQL role/owner integrity. The approved scope excluded deployment, release, production data work, consumer compatibility support, multi-instance market guarantees, and client work.

Completion means the approved delivery stages, Test, independent Review, and both required PRs have been verified. Product PR merge is separate from a deployment or release and none is claimed.

## Gates and stage record

| Stage | Primary artifact / gate | Outcome |
| --- | --- | --- |
| Handoff | `HAND-0002@v1`, `HSUM-0002@v1` | Validated PASS. |
| Plan gate | `OPLAN-0002@v1`, `HAPP-0002@v1` | Human approved. |
| Definition | `DEF-0002@v1` | Validated PASS. |
| Architecture | `ARC-0002@v2` | v1 paused for four human decisions; the bounded v2 revision validated PASS. |
| Requirements | `REQ-0002@v1` | Validated PASS. |
| Decomposition | `DEC-0002@v1` | Validated PASS. |
| Routing | `ROUTE-0002@v2` | v1 blocked for verified branch/environment facts; v2 validated PASS after user-authorized branch, disposable PostgreSQL, and no-existing-data decision. |
| Execution gate | `HAPP-0003@v1` | Human approved exact route and six-unit scope. |
| Execution | `EXEC-0002@v1`, `VAL-0019` | PASS_WITH_WARNINGS. |
| Test | `TEST-0002@v2`, `VAL-0017` | PASS_WITH_WARNINGS. Test v1 exposed a stale test expectation; product commit `f6d40d7` corrected it and v2 rechecked all suites. |
| Independent Review | `REV-0002@v1`, `VAL-0018` | PASS_WITH_WARNINGS; no actionable defect found. |

## Product and artifact publication

- Product: [SampleApp05/Atra-Services #3](https://github.com/SampleApp05/Atra-Services/pull/3), `main` ← `workflow/WF-0002-atra-evaluation-remediation`, head `f6d40d7e3e39c540fbc8f94f331db2c5baac3d70`, verified **MERGED** at `2026-09-24T18:35:08Z`.
- Artifact: [SampleApp05/AI-Workflows #4](https://github.com/SampleApp05/AI-Workflows/pull/4), `main` ← `workflow/WF-0002-atra-evaluation-remediation`, head `bdfb2e8f0f10cc1c6b82f57e778c0b6d1bf54cda`, verified **MERGED** at `2026-09-24T18:35:46Z`.
- Publication exception: PR #4 merged before this final report and closeout records were authored. This record is retained on the workflow branch; it was not included in the already merged PR. No additional artifact PR is created because Contract v1.2 permits exactly one artifact PR per run.

## Evidence and residual risk

At the reviewed product head, all workspace builds passed; market passed 89 tests, database 88 tests, and auth 202 tests with four skipped. Focused evidence passed for market membership (40 tests), auth authority/lifecycle (138 tests), four competing nonce attempts, and six PostgreSQL role/owner invariant or competing-write cases. Database proof is limited to the authorized local PostgreSQL 17.11 disposable loopback environment.

Residual warnings are the four skipped auth tests, two non-failing Vitest compatibility warnings, and the intentionally breaking `/prices` body. The approved no-consumer decision bounds that contract change, but no external consumer/client assertion is made.

## Targets, fallbacks, liveness, and timing

Native Codex produced Definition, Requirements, Routing, market work, Test, and independent Review. Claude CLI produced Decomposition and the environment/infrastructure work; Architecture used the plan-approved native fallback after the durable Claude dispatcher rejected its missing required artifact-path marker before work began. Execution used approved Codex and Claude paths; the durable record retains the actual unit outcomes rather than attributing unmeasured quality scores.

The journal records the plan and execution gates, the architecture and routing blockers/resolutions, and execution start/completion through EU-04 dispatch. Chat heartbeats continued during delivery, but a complete later durable heartbeat/event series was not recorded; no confirmed suspension or `STALLED_SUSPECTED` event is available. Measured aggregate active work, usage-wait, and gate-wait durations are **unknown** because the journal generally records checkpoints rather than duration accounting. No unsupported time estimate is substituted.

## Required human action

None to complete this governed delivery run. If the final report itself must be present on `main`, it requires a separately authorized publication policy exception or follow-up because the run's sole artifact PR is already merged.
