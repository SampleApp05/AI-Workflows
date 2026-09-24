---
artifact:
  id: HAND-0002
  type: HANDOFF
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
  parents: []
  sources:
    - type: user_request
      role: explicit authorization to run the delivery workflow
      detail: "Make the needed fixes/changes discussed in the Atra Evaluation workflow."
    - type: workflow_artifact
      workflow_id: WF-0001
      artifact: WRUN-0001@v3
      path: Cross-Stack/workflows/phase-foundation-evaluation/WF-0001/workflow_orchestrator/workflow-run-report-v3.md
      role: authoritative verified-finding summary for remediation scope
    - type: workflow_artifact
      workflow_id: WF-0001
      artifact: TEST-0001@v1
      path: Cross-Stack/workflows/phase-foundation-evaluation/WF-0001/test_engineer/test-report-v1.md
      role: authoritative detailed evidence for remediation scope
ownership:
  created_by: intake_analyst
  performed_by: intake_analyst
  recorded_by: intake_analyst
  engine: codex
  target_id: codex-native
---

# Handoff — Atra evaluation remediation

## Intent

**Explicit request.** Start a **delivery-mode** workflow to make the needed fixes and changes discussed in Atra's prior evaluation workflow. The approved intake scope is limited to the six material, verified findings recorded in `WRUN-0001@v3` and evidenced in `TEST-0001@v1`.

**Evidence posture.** The prior evaluation artifacts are authoritative evidence of the six baseline findings and their limits. They are not authority to copy their unresolved assumptions, to expand this delivery run to other observations, or to select a technical solution before later workflow stages.

## Problem / Motivation

The prior read-only evaluation found six material gaps in the `Atra-Services` baseline that can undermine market-stream correctness, trustworthy cache consumers, authorization provenance, replay prevention, actor attribution, and role-invariant integrity. The user has now explicitly authorized a separate delivery workflow to remediate those findings.

## Goal

Deliver verified product changes, tests, and review evidence that remediate the following bounded findings in `Atra-Services`:

1. Correct market-stream unsubscribe reference counting.
2. Make market REST cache freshness visible to consumers.
3. Derive authorization for sensitive wallet, role, recovery, and revoke actions from the authenticated session rather than caller-supplied authority identifiers.
4. Make nonce consumption atomic.
5. Persist and authenticate the wallet that establishes a session so later actions have durable wallet attribution.
6. Enforce `OWNER`, `RECOVERY`, and duplicate-role invariants at the database boundary.

## Scope

### In

- The six findings listed in the Goal, as established by `WRUN-0001@v3` and `TEST-0001@v1` at the recorded `Atra-Services` baseline `be51242593fe6643aff11c491d4f8a6c0835aed5`.
- Product, database-schema, API-contract, and test changes demonstrably necessary to remediate and verify those six findings.
- Delivery workflow stages required by Contract v1.2, including the plan and execution human gates, implementation, Test, independent Review, and the required product and artifact pull requests.

### Out

- Package-manager migration or other package-management change.
- Any unrelated finding from WF-0001, including audit cursor pagination, multi-instance market aggregation/cache behavior, iOS client behavior, provider integration, Phase 2.5 expansion, portfolio work, or broader chain portability.
- New product capabilities not necessary for the six bounded remediations.
- Deployment, merge, release, or production-data operations.
- Treating the former evaluation's incomplete Review as completed or using its earlier requirements/architecture as a delivery specification.

## Known Constraints

- **Explicit decision:** This is a new `delivery` run, `WF-0002`, governed by Artifact Contract v1.2. It must follow its fixed forward-only delivery chain and both human gates.
- **Explicit decision:** The user authorized fixes only for the six material, verified WF-0001 findings enumerated above.
- **Explicit decision:** Do not undertake a package-manager migration or unrelated work.
- **Documented baseline evidence:** WF-0001 observed the findings at `Atra-Services` `main` commit `be51242593fe6643aff11c491d4f8a6c0835aed5`; its existing mocked suites passed 341/341 and did not establish live-provider, live-database concurrency, or client behavior.
- **Documented scope limit:** `Atra-Services` is the only recorded product repository. No iOS repository, deployment environment, or separately managed database schema is verified in this intake.
- **Contract requirement:** Delivery may not begin implementation until the human has approved the exact Routing Plan and execution scope. Required product and artifact pull requests must be verified before completion; neither may be inferred.

## Explicit Decisions

- The remediation target is the exact six-finding set in this Handoff; it is not a general cleanup of every concern noted by WF-0001.
- Market-stream behavior must retain correct upstream subscription accounting when an unsubscribe is unsolicited or repeated for a socket that did not hold that symbol subscription.
- Market REST consumers need an observable contract that distinguishes fresh data, stale-but-usable data, and unavailable/error conditions.
- Sensitive wallet, role, recovery, and revoke actions must base authority on the authenticated session trust context, not body-supplied account or wallet identity.
- Nonce use must provide atomic single-use consumption suitable for concurrent verification attempts.
- Session behavior must durably retain the wallet that authenticated the session and use that trusted attribution for subsequent authentication/authorization context.
- Database enforcement must cover exactly one `OWNER`, at most one `RECOVERY`, and prevention of duplicate role associations at the database boundary.

## Assumptions

- **Assumption:** The recorded baseline is an appropriate starting reference for the delivery analysis; the actual implementation baseline, branch, and any intervening changes must be freshly verified before product claims or edits.
- **Assumption:** The product repository and pull-request details in `WF-0002/manifest.yaml` remain usable; later controller and routing stages must verify them rather than rely on this intake.
- **Assumption:** The exact API representation for cache freshness, schema migration approach, authorization-context shape, and compatible behavior for existing callers are not yet decided. They require later architecture and requirements decisions.
- **Inference:** Remediating the six findings will likely span market, authentication, and database boundaries. This records cross-stack scope only; it does not assign components, workers, or solutions.

## Open Questions

- What is the verified current `Atra-Services` commit and working-tree state at delivery analysis time, and does it differ materially from the WF-0001 baseline?
- Which backward-compatibility commitments, if any, apply to existing market REST consumers and sensitive-action API callers when their contracts and authority sources change?
- Does the deployable database environment accept the constraints/migration mechanics that later stages determine are necessary, and is existing data known to satisfy the intended invariants?
- What precise recovery and revoke semantics are required beyond the verified rule that their authority derive from authenticated session context?

## Expected Outcome

A gated delivery result containing bounded product changes and verification evidence for the six findings, an independently reviewed product pull request, and the required artifact pull request. The result must preserve explicit decisions, assumptions, and unknowns; must not claim completion for unverified operational behavior; and must not broaden into package-manager or unrelated remediation work.

This Handoff does not approve a plan or execution gate, choose an architecture or implementation, define detailed requirements, assign workers, create product changes, or authorize a merge or deployment.
