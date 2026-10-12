# Workflow Artifact Contract — v1.3

Version 1.3 extends v1.2 for new runs. Existing runs retain their recorded contract version. Earlier repository, identity, scope, gate, independence, event, and PR rules continue except where refined below.

## Delivery correction cycle

Approved Handoff, Definition, Architecture, Requirements, Decomposition, Routing, and both human gates remain fixed after execution approval. Evaluation remains read-only. Delivery may repeat **bounded Execution correction → Test → independent Review** after a Test or Review finding. This corrects approved work without reopening upstream decisions.

During Test, the Test owner may correct assigned test files, fixtures, or test-only support and rerun affected and required aggregate checks. It must not silently change production behavior. A production defect goes to the execution coordinator, which assigns a correction against an approved requirement and route, audits the diff and commit, updates the existing product PR, and requests fresh Test and independent Review. A Review finding follows the same path. The implementer cannot review its correction.

Each execution correction is recorded in `execution_coordinator/correction-<finding-id>-vN.md` with a unique `CORR` ID, source finding/artifact, requirement and routed unit, classification, exact permitted files, worker and target, before/after product commits, validation, attempt, and disposition. The execution record indexes corrections. Earlier Test/Review versions remain intact. Their verdicts are valid only for their assessed commit; after a product change, supersede them and point the manifest at new validated versions. Record a test-only correction in the revised Test Report with its changed files, commits, and checks; a separate `CORR` artifact is optional unless the coordinator assigns work.

The coordinator may expand a correction file allowlist only through `execution_coordinator/route-amendment-<finding-id>-vN.md` (`RAMD` ID). This correction-specific overlay must show necessity for an already approved requirement, preserve the approved Routing Plan, repository and product scope, and pass orchestrator validation before dispatch. It cannot change unit purpose, dependencies, acceptance criteria, or target authority. New behavior, changed acceptance criteria, a new repository, unsafe data action, or changed human decision needs a new human decision or separate run. This cycle grants no merge, deployment, production-data, or gate-bypass authority.

## Blocker classes

Before marking a run blocked, record `BLOCKER_CLASSIFIED` with evidence, owner, approved remedy, and next action.

| Class | Response |
| --- | --- |
| `MECHANICAL` | Correct a deterministic invocation or record error after confirming no worker ran or wrote; use a unique action/job ID and audit any retry. |
| `ENVIRONMENT` | Use an already approved setup, recovery, or fallback within its guardrails; stop if authority or side effects differ. |
| `TEST_DEFECT` | Assign a bounded test-only correction and recheck. |
| `IMPLEMENTATION_DEFECT` | Use the delivery correction cycle. |
| `DECISION_REQUIRED` | Present the concrete choice and wait for an actual human decision. |
| `UNRESOLVED` | Preserve evidence and checkpoint; investigate without blind redispatch. |

Classify from observed facts, not a worker's label. A failed test or review remains failed until verified. Repeated unsuccessful corrections require a recorded diagnosis and narrower next action; never retry indefinitely. A live worker cannot be stopped, duplicated, or rerouted without the v1.2 human choice.

## Controller checkpoint and continuation

The orchestrator atomically maintains local `workflow_orchestrator/controller-state.json`, indexing run ID/contract, stage and delivery cycle, each pending action ID/state, worker target and job directory or native thread ID, last verified product commit, latest validated Test/Review versions and commits, blocker class, and next authorized action. Distinct action IDs may cover approved independent parallel units; no two actions may reserve the same job directory. It contains no credentials or raw streams and does not replace the manifest, journal, or approval records. This operational checkpoint is excluded from the artifact PR.

Reserve an action ID and deterministic job directory before dispatch. On worker completion, failure, heartbeat, or controller restart, reconcile checkpoint, manifest, events, worker sidecars/native status, Git state, and gates. Observe an existing job instead of dispatching it again. If a launch may have occurred without a verifiable identity, preserve an ambiguous checkpoint and investigate before redispatch. A heartbeat reports status and performs reconciliation; it continues safe approved work when ready without a manual continuation prompt. Usage suspension, target outage, and live-worker safeguards from v1.2 remain in force. Journal action IDs, correction cycles, blocker resolution, recovery, commit invalidation, and revalidation; report any gaps honestly.

## Commit-bound completion

Before `COMPLETE`, verify that each product PR head equals the commit assessed by the latest validated Test and independent Review and that all correction findings are disposed. Write and validate the final Run Report before publishing the artifact PR. Verify that the artifact PR head contains that report, required stage/gate records, and current manifest, then record URL, branches, commit, and state. A Git commit cannot contain its own hash: v1.3 therefore permits `PENDING_FINAL_PUBLICATION` for the artifact PR's final head in the committed report/manifest and records the observed final head in the local controller checkpoint and final chat report after publication. This refines v1.1's self-referential final-head recording requirement. An earlier PR is not evidence of final closeout publication. A PR may remain unmerged; this contract grants no merge authority.
