---
artifact:
  id: DEF-0002
  type: DEFINITION
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
  references:
    - path: intake_analyst/handoff-v1.md
    - path: intake_analyst/handoff-summary-v1.md
    - path: workflow_orchestrator/orchestration-plan-v1.md
    - workflow_id: WF-0001
      artifact: WRUN-0001@v3
      path: Cross-Stack/workflows/phase-foundation-evaluation/WF-0001/workflow_orchestrator/workflow-run-report-v3.md
    - workflow_id: WF-0001
      artifact: TEST-0001@v1
      path: Cross-Stack/workflows/phase-foundation-evaluation/WF-0001/test_engineer/test-report-v1.md
ownership:
  created_by: problem_analyst
  performed_by: problem_analyst
  recorded_by: problem_analyst
  engine: codex
  target_id: codex-native
---

# Definition — Atra evaluation remediation

## Problem statement

The verified `Atra-Services` baseline contains six bounded defects that can produce incorrect market-stream subscription behavior, hide cache usability from consumers, allow sensitive-action authority to originate from caller-controlled identifiers, permit nonce replay races, lose the wallet identity that authenticated a session, and allow invalid wallet-role states to persist. The user authorized a delivery workflow to remediate exactly these six defects, with evidence, independent review, and no expansion into a general product cleanup.

## Context and evidence boundary

`HAND-0002@v1` is the authoritative delivery intake. It carries forward six findings from the read-only evaluation evidence in `WRUN-0001@v3` and `TEST-0001@v1`, observed at `Atra-Services` `main` commit `be51242593fe6643aff11c491d4f8a6c0835aed5`. The existing mocked suites passed 341/341 tests at that baseline, but did not prove live-provider, live-database concurrency, or client behavior.

This Definition describes outcomes and boundaries only. The evaluation findings establish the remediation need, but do not decide an API representation, authorization-context form, persistence approach, migration approach, compatibility policy, or implementation plan. The actual product baseline and working-tree state require fresh verification before product claims or edits.

## Desired outcomes

The delivery result must establish, for the approved six findings, that:

1. An unsubscribe affects market-stream accounting only when the requesting socket actually holds the relevant subscription; unsolicited or repeated unsubscribes do not remove another subscriber's interest.
2. A market REST consumer can observe whether a returned market result is fresh, stale but usable, or unavailable/error, rather than receiving an indistinguishable result.
3. Sensitive wallet, role, recovery, and revoke actions determine authority from the authenticated session trust context, not from caller-supplied account or wallet authority identifiers.
4. A nonce has single-use behavior under concurrent verification attempts.
5. The wallet that authenticates a session remains durably attributable throughout subsequent authentication and authorization context.
6. At the database boundary, each account has exactly one `OWNER`, at most one `RECOVERY`, and no duplicate wallet-role association.

The result must include proportionate product evidence, Test evidence, independent Review evidence, and the required product and artifact pull requests. It must not claim unverified operational behavior.

## In scope

- The exact six outcomes above, derived from the six verified WF-0001 findings.
- Changes and verification demonstrably necessary to establish those outcomes in `Atra-Services`.
- The Contract v1.2 delivery chain, including the approved plan gate, later execution gate, implementation, Test, independent Review, and required product and artifact pull requests.

## Out of scope

- Package-manager migration or any other package-management change.
- Audit cursor pagination; multi-instance market aggregation or cache behavior; iOS/client behavior; provider integration; Phase 2.5 expansion; portfolio work; and broader chain portability.
- A new product capability not demonstrably necessary for one of the six outcomes.
- Deployment, merge, release, production-data operation, or a claim about a separately managed database environment not supported by verified evidence.
- Treating WF-0001's prior architecture, requirements, or incomplete Review as a delivery specification.

## Observable success criteria

Success is observable when the final delivery evidence can show all of the following without relying solely on mocked nominal-path tests:

1. A socket that never held, or already released, a symbol subscription cannot reduce the upstream interest retained for another socket.
2. The contract visible to a market REST consumer distinguishes fresh, stale-but-usable, and unavailable/error results for the relevant market data.
3. For every sensitive wallet, role, recovery, and revoke action in scope, conflicting or caller-supplied authority identifiers cannot determine authorization over the authenticated session context.
4. Concurrent attempts to verify one otherwise valid nonce result in no more than one successful consumption.
5. Subsequent session-authenticated actions retain the wallet that established the session as their trusted wallet attribution, including where an account has more than one wallet.
6. Attempts to persist a second `OWNER`, a second `RECOVERY`, or a duplicate wallet-role association for an account are rejected at the database boundary, including under competing writes where applicable.
7. Test and independent Review map evidence to the approved scope and explicitly state any environment, migration, or compatibility limitation that remains unverified.
8. The delivery produces one verified product pull request and one verified artifact pull request, with neither merged or deployed by this workflow.

## Affected areas

The approved concern spans only these product concern areas in `Atra-Services`:

- market REST cache consumer behavior and market-stream subscriber accounting;
- session, wallet, role, recovery, revoke, and nonce trust behavior;
- persistent session-to-wallet attribution; and
- account-wallet-role integrity at the database boundary.

These areas identify affected behavior, not a component design, file list, data model, interface, or worker assignment.

## Constraints

- `WF-0002` is fixed as a Contract v1.2 `delivery` run and must move forward through its approved stage chain.
- The approved scope is limited to the six findings and excludes all listed deferred evaluation observations.
- No product implementation may start until the human approves the exact Routing Plan and execution scope.
- `Atra-Services` is the only recorded product repository. Its baseline, branch, working-tree state, remote, and product-PR authority must be verified at the stages responsible for those checks.
- Compatibility, API behavior, session/recovery/revoke semantics, and database migration feasibility are not predetermined by this Definition.
- Completion requires verified Test, independent Review, a product pull request, and an artifact pull request. No merge, deployment, release, or production-data operation is authorized.
- User-owned and unrelated working-tree changes must be preserved.

## Dependencies

- Approved `OPLAN-0002@v1` and recorded plan approval `HAPP-0002@v1`.
- Validated `HAND-0002@v1` and `HSUM-0002@v1` as the authoritative delivery scope.
- WF-0001 finding evidence limited to `WRUN-0001@v3` and `TEST-0001@v1`.
- A freshly verified `Atra-Services` baseline before implementation, and verified repository/branch/PR authority before product publication.
- Evidence capable of assessing concurrent nonce and database-invariant behavior; mocked tests alone are insufficient for those outcomes.

## Risks

- **Authorization and compatibility risk:** changing how sensitive actions establish authority may affect existing callers; compatibility commitments are unknown and must be made explicit later rather than assumed.
- **Session semantics risk:** durable wallet attribution must remain meaningful for multi-wallet accounts and through recovery/revoke behavior, whose precise semantics are not yet defined.
- **Data-integrity risk:** existing or deployable database state may be incompatible with the required role invariants. No production-data action is authorized.
- **Concurrency-evidence risk:** a non-concurrent or mocked test can miss the nonce and role-invariant failure modes this run must address.
- **Market-contract risk:** exposing cache usability may affect existing consumers; no representation or compatibility approach is selected here.
- **Scope-expansion risk:** adjacent evaluation observations can appear during inspection but remain out of scope unless separately approved.
- **Publication risk:** missing verified product- or artifact-PR facts blocks completion rather than permitting an inferred result.

## Assumptions

- The six findings remain an appropriate remediation target unless a freshly verified baseline materially changes their applicability; such a material divergence must be surfaced, not silently absorbed.
- The manifest-recorded product and artifact repository information remains a candidate for later verification, not an established publication fact.
- The approved plan's target and fallback intentions govern later stage dispatch, but this Definition assigns no implementation work.
- The required outcomes may involve connected market, session, and database concerns; that cross-concern observation does not select a design.

## Open questions

### Non-blocking questions for later stages

- What is the current verified `Atra-Services` commit and working-tree state, and does it materially differ from the evaluated baseline?
- Which compatibility commitments apply to market cache responses and sensitive-action callers?
- What recovery and revoke semantics are required beyond deriving authority from authenticated session context?
- What database migration feasibility and existing-data conditions apply in the deployable environment?
- What evidence can verify the concurrency and database-boundary outcomes in the available environment?

No unresolved question blocks this Definition. Each question requires explicit treatment by the appropriate later stage; none authorizes a scope expansion or a presumed technical choice.

## Self-check and requested validation

This draft has exact `HAND-0002@v1` and `HSUM-0002@v1` lineage, preserves the approved delivery mode and six-finding boundary, distinguishes evidence from assumptions and unknowns, and states observable outcomes without selecting architecture, APIs, schema or migration mechanics, technical requirements, tasks, workers, or routing. **READY_FOR_ARCHITECTURE is requested for orchestrator validation; this Definition does not validate itself, advance the workflow, or approve a gate.**
