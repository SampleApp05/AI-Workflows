---
artifact:
  id: REQ-0002
  type: REQUIREMENTS
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
    - ARC-0002@v2
  references:
    - path: solution_architect/architecture-v2.md
    - path: workflow_orchestrator/architecture-decisions-v1.md
    - path: problem_analyst/definition-v1.md
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
  created_by: requirements_engineer
  performed_by: requirements_engineer
  recorded_by: requirements_engineer
  engine: codex
  target_id: codex-native
---

# Delivery requirements — Atra evaluation remediation

## Contract and scope

`REQ-0002@v1` turns validated `ARC-0002@v2` and the binding decisions in `DEC-0001@v1` into the delivery contract for `WF-0002`. It covers only the six approved remediation outcomes in `Atra-Services`: market-stream unsubscribe accounting, `/prices` cache freshness, session-derived sensitive-action authority, nonce single use, authenticating-wallet session attribution, and account-wallet-role integrity.

The direct breaking `/prices` response is intentional because the recorded human decision establishes that no consumers exist. Existing incompatible role or owner data is a stop-and-report condition, not a repair authorization. Refresh retains the signing wallet; revoke is self-only; recovery, ownership transfer, and loss of an authorizing role invalidate affected sessions. A disposable, non-production PostgreSQL environment is authorized only for workflow migration and competing-write evidence.

No requirement authorizes package-management work, audit pagination, multi-instance market behavior, iOS/client/provider/portfolio/Phase 2.5 expansion, deployment, merge, release, production-data operations, or a product capability not necessary for these outcomes.

## Product requirements

| ID | Source | Requirement | Acceptance criteria | Verification method |
| --- | --- | --- | --- | --- |
| REQ-MKT-001 | HAND-0002 §Goal/Explicit Decisions; DEF-0002 §Desired outcomes/Observable success criteria; ARC-0002@v2 §1 | Market-stream subscription interest for a symbol shall change only when the requesting socket actually gains or releases membership for that symbol. An absent-symbol unsubscribe, an unsubscribe by a non-member, and a repeated unsubscribe by an already released socket shall leave retained subscriber interest and upstream interest unchanged. Disconnect cleanup shall preserve the same behavior. | (a) With two sockets subscribed to one symbol, removal by a non-member or an already removed member does not cause upstream unsubscription or prevent the remaining socket from retaining interest. (b) Removing one real member while another remains does not remove upstream interest. (c) Upstream unsubscription occurs only after the final real member releases the symbol. | Behavior evidence exercises the stated multi-socket membership transitions and records the observed upstream-interest result. The evidence is explicitly limited to the in-process manager; it makes no multi-instance claim. |
| REQ-API-002 | HAND-0002 §Goal/Explicit Decisions; DEF-0002 §Desired outcomes/Observable success criteria; DEC-0001@v1 §1; ARC-0002@v2 §2 | `/prices` shall directly expose a freshness-aware successful-result contract instead of the former bare `MarketTicker[]` body. Each returned market result shall contain its ticker data and an independently observable cache-freshness classification of `fresh` or `stale`. An unavailable result shall remain distinguishable as an unavailable/error outcome and shall not be represented as a stale successful result. | (a) Current cached data is returned as `fresh`. (b) Usable cached data returned while refresh is pending is returned as `stale`. (c) A cache-miss/upstream-unavailable outcome is observably different from a successful stale result. (d) When a response contains multiple successful results with differing cache states, each result exposes its own state. | Interface evidence records representative fresh, stale, unavailable/error, and mixed successful-result observations against the delivered contract. No versioned, parallel, negotiated, or legacy response contract is required. |
| REQ-AUTH-003 | HAND-0002 §Goal/Explicit Decisions; DEF-0002 §Desired outcomes/Observable success criteria; ARC-0002@v2 §3 and Cross-boundary constraints | Every in-scope sensitive wallet, role, recovery, and revoke action shall derive its acting account, wallet, session, and chain authority from the validated authenticated session. Caller-supplied identity values may express an operation target or proof material only; they shall not select or override the acting authority. Revoke shall require authentication and permit a user to revoke only that user's own session. Audit attribution for these actions shall identify the authenticated session wallet. | (a) A missing or invalid authenticated session cannot authorize an in-scope action. (b) A body/session authority mismatch is rejected and cannot authorize the body-selected account or wallet. (c) A caller cannot broaden revoke authority through a body account, wallet, or session identifier. (d) Observed audit attribution for a successful in-scope action is the session's authenticated wallet, including for an account with more than one wallet. | Security-boundary evidence covers valid authenticated behavior and the stated negative cases for each applicable action family. It records any endpoint/action found outside the approved families rather than expanding the scope. |
| REQ-NONCE-004 | HAND-0002 §Goal/Explicit Decisions; DEF-0002 §Desired outcomes/Observable success criteria; ARC-0002@v2 §4 | A nonce used by an in-scope verification flow shall be consumed at most once, including when otherwise valid attempts compete concurrently. Successful consumption shall be conditional on the challenge identity/value, wallet, purpose, unused state, and expiry at the decision point. A failed or losing consumption shall not create the durable privileged outcome gated by that nonce. Where an in-scope operation needs more than one nonce, either all required consumptions and the associated durable outcome succeed together or none do. | (a) Concurrent verification attempts for one otherwise valid nonce yield no more than one successful consumption and no more than one resulting privileged outcome. (b) A used, expired, wrong-wallet, wrong-purpose, or wrong-value challenge cannot produce a successful outcome. (c) Failure in a multi-nonce operation leaves neither an unexplained partial consumption nor a partial associated outcome. | Database-backed concurrency evidence is required; mocked nominal-path evidence alone is insufficient. It records the competing-attempt outcomes and any applicable all-or-nothing result. |
| REQ-SES-005 | HAND-0002 §Goal/Explicit Decisions; DEF-0002 §Desired outcomes/Observable success criteria; DEC-0001@v1 §3; ARC-0002@v2 §5 | Every successfully authenticated session shall retain the wallet that established it as durable trusted attribution. Subsequent session authentication, authorization, audit attribution, and revoke behavior shall use that persisted wallet rather than infer identity from an arbitrary account-role association. Refresh shall preserve the original signing wallet. Recovery, ownership transfer, and loss of a role authorizing a session shall invalidate the sessions affected by that event. | (a) For an account with multiple wallets, session-authenticated behavior and audit attribution resolve to the wallet that signed into that session. (b) A refreshed session retains that same wallet. (c) Revoke is self-only and uses the authenticated session attribution. (d) After each selected lifecycle event, each affected session can no longer authenticate or authorize subsequent in-scope action. | Session lifecycle evidence demonstrates multi-wallet attribution, refresh, self-only revoke, and invalidation after recovery, ownership transfer, and loss of an authorizing role. Evidence identifies the event and affected session(s) without claiming semantics beyond the selected policy. |
| REQ-DATA-006 | HAND-0002 §Goal/Explicit Decisions; DEF-0002 §Desired outcomes/Observable success criteria; DEC-0001@v1 §§2,4; ARC-0002@v2 §6 | At the PostgreSQL boundary, committed account-wallet-role state shall prohibit duplicate `(account, wallet, role)` associations, more than one `OWNER` per account, and more than one `RECOVERY` per account; each account shall have exactly one `OWNER`. Where a canonical owner-wallet relationship is retained, it shall agree with the account's `OWNER` association at commit. These guarantees shall hold against alternate and competing writes, not only through application checks. | (a) Attempts to persist a duplicate role association, second `OWNER`, or second `RECOVERY` for an account are rejected at the database boundary, including competing writes where applicable. (b) Account provisioning and ownership transfer complete only in a committed state with exactly one consistent owner. (c) An authorized preflight that detects duplicate roles, multiple owners/recovery roles, or canonical-owner mismatch stops the workflow and reports evidence; it does not repair, delete, backfill, or accept incompatible data. | Migration and invariant evidence runs only in the authorized disposable non-production PostgreSQL environment. It includes migration application and competing-write observations. It must not claim production compatibility or connect to, alter, or repair production data. |

## Delivery evidence requirements

| ID | Source | Requirement | Acceptance criteria | Verification method |
| --- | --- | --- | --- | --- |
| REQ-EVD-007 | DEF-0002 §Observable success criteria; ARC-0002@v2 §Cross-boundary constraints/Risks; OPLAN-0002@v1 §Evidence and baseline approach | Delivery evidence shall map every product requirement above to observed behavior on the implemented product and state any remaining environment, migration, compatibility, or provider limitation. It shall not treat mocked nominal-path success as sufficient proof of nonce atomicity or database invariant durability. | (a) Test evidence maps REQ-MKT-001 through REQ-DATA-006 to results and includes the required database-backed concurrency/invariant evidence. (b) Independent Review assesses the actual product change and Test evidence against those same IDs, including residual risk. (c) Any migration incompatibility, unavailable disposable database evidence, or material baseline divergence is reported as a blocker or limitation rather than represented as success. | The Test Report and independent Review cite requirement IDs, exact observed evidence, and limitations. Live-provider, multi-instance, and client behavior remain unclaimed unless separately authorized and evidenced. |
| REQ-DEL-008 | HAND-0002 §Expected Outcome/Known Constraints; DEF-0002 §Desired outcomes/Constraints; OPLAN-0002@v1 §Ordered stage chain; Contract v1.2 §§Fixed mode/Human gates/Assignment and evidence | The delivery run shall remain forward-only and shall not begin product implementation before approval of the exact Routing Plan and execution scope. Completion requires one verified, unmerged product pull request for the changed `Atra-Services` repository and one verified, unmerged artifact-repository pull request containing the run evidence, together with validated Test and independent Review evidence. | (a) The separate execution approval identifies the exact Routing Plan before implementation starts. (b) Product and artifact PR records contain verified repository, base branch, head branch, commit, URL, and state. (c) No merge, deployment, release, or production-data operation is represented as part of this run. | Controller and execution records provide the gate, validation, and PR facts required by Contract v1.2; unknown or unavailable facts remain explicitly unknown or block completion. |

## Dependencies, assumptions, and stop conditions

- The actual product baseline, branch, working-tree state, remote, and product-PR authority must be freshly verified by the stages responsible for them. The prior observed baseline is evidence, not an implementation authorization.
- The approved disposable PostgreSQL environment is a dependency for `REQ-NONCE-004` and `REQ-DATA-006` verification. If it is unavailable, the required database-backed claims cannot pass as verified.
- The only approved incompatible-data response is stop and report. No preflight result authorizes production connection, repair, deletion, backfill, deployment, or acceptance of incompatible data.
- `REQ-API-002` is deliberately a direct API contract change. There is no compatibility requirement for former `/prices` consumers because the recorded human decision establishes that none exist.

## Traceability and self-check

The six functional requirements map one-to-one to the six approved findings and retain all binding human decisions. `REQ-EVD-007` and `REQ-DEL-008` add delivery evidence and Contract v1.2 completion criteria only; they do not introduce product capabilities. Each requirement has observable acceptance criteria and an evidence method, preserves the approved exclusions, and does not prescribe implementation tasks, files, workers, routes beyond the already approved `/prices` interface, code structure, migration mechanics, or test code.

**READY_FOR_DECOMPOSITION is requested for orchestrator validation.** This artifact does not validate itself, approve a gate, select workers, or advance the workflow.
