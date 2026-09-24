---
artifact:
  id: HSUM-0002
  type: HANDOFF_SUMMARY
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
  references:
    - path: intake_analyst/handoff-v1.md
    - workflow_id: WF-0001
      artifact: WRUN-0001@v3
      path: Cross-Stack/workflows/phase-foundation-evaluation/WF-0001/workflow_orchestrator/workflow-run-report-v3.md
    - workflow_id: WF-0001
      artifact: TEST-0001@v1
      path: Cross-Stack/workflows/phase-foundation-evaluation/WF-0001/test_engineer/test-report-v1.md
ownership:
  created_by: intake_analyst
  performed_by: intake_analyst
  recorded_by: intake_analyst
  engine: codex
  target_id: codex-native
---

# Handoff Summary — Atra evaluation remediation

## Need

**Explicit request:** run a delivery workflow to make the fixes and changes discussed in the Atra evaluation. Scope is restricted to six material, verified WF-0001 findings.

## Context

The primary intake authority is [HAND-0002@v1](handoff-v1.md). The finding summary is `WRUN-0001@v3`; detailed evidence is `TEST-0001@v1`. They observed the gaps at `Atra-Services` baseline `be51242593fe6643aff11c491d4f8a6c0835aed5`, while also recording that mocked unit suites do not prove live database concurrency, provider, or client behavior.

## What Was Considered

- Market-stream unsubscribe accounting when a socket did not hold the subscription it attempts to remove.
- Consumer-visible fresh, stale-but-usable, and unavailable/error states for market REST cache results.
- Session-derived authority for sensitive wallet, role, recovery, and revoke actions.
- Atomic single-use nonce consumption under concurrent verification.
- Persisting and using the wallet that authenticated a session.
- Database-level enforcement of one `OWNER`, at most one `RECOVERY`, and no duplicate role associations.

## Decision

**Explicit decision:** deliver remediation only for those six findings, through the Contract v1.2 delivery workflow and its two human gates. Do not treat the prior evaluation as a delivery design or as authorization for other findings.

## Why

The evaluation identified material correctness, authorization, replay-control, attribution, and integrity risks. The user has now authorized a bounded delivery follow-up rather than leaving those verified findings at the evidence checkpoint.

## Important Constraints

- No package-manager migration or unrelated work.
- `Atra-Services` is the only recorded product repository; the current baseline must be reverified before implementation.
- No product work begins until the human approves the exact Routing Plan and execution scope.
- Completion requires verified Test, independent Review, and the required product and artifact pull requests; no merge or deployment is authorized.

## Deferred / Out of Scope

- Audit cursor pagination, multi-instance market behavior, iOS/client work, provider integration, Phase 2.5 expansion, portfolio work, and broad chain portability.
- Any product capability not demonstrably needed for the six remediations.
- Package-management changes, deployment, merge, release, and production-data operations.
- Treating unresolved assumptions from WF-0001 as decided requirements.

## Open Questions

- What current product baseline and working-tree state will later stages verify?
- What compatibility commitments govern changed API/freshness and sensitive-action contracts?
- What database migration feasibility and existing-data conditions apply?
- What recovery/revoke semantics are required beyond session-derived authority?

## Future Reference

Use `HAND-0002@v1` as the authoritative intake scope. Keep the six WF-0001 findings distinct from unverified or deferred observations, and preserve the stated decisions, assumptions, and unknowns through later stages.
