---
artifact:
  id: REQ-0001
  type: REQUIREMENTS
  version: 1
  status: DRAFT
workflow:
  id: WF-0001
  project: Atra
  technology: Cross-Stack
  feature: phase-foundation-evaluation
  mode: evaluation
  contract_version: "1.2"
lineage:
  parents:
    - ARC-0001@v1
  references:
    - path: solution_architect/architecture-v1.md
    - path: problem_analyst/definition-v1.md
    - path: intake_analyst/handoff-v1.md
    - path: workflow_orchestrator/orchestration-plan-v1.md
ownership:
  created_by: requirements_engineer
  performed_by: requirements_engineer
  recorded_by: requirements_engineer
  engine: codex
  target_id: codex-native
---

# Evaluation requirements — Atra foundational phases

## Evaluation contract

This contract defines read-only comparison criteria for `Atra-Services` baseline `be51242593fe6643aff11c491d4f8a6c0835aed5`. The intended-state authority is the visible source preserved in `HAND-0001@v1`; repository evidence must not become the specification. For every requirement, downstream evaluation must cite the baseline evidence inspected and classify the outcome as **supported**, **divergent**, or **indeterminate**, with uncertainty and residual risk where applicable.

No criterion authorizes a product, test, configuration, documentation, or Git change; remediation, architecture redesign, test creation, decomposition, routing, and implementation are out of scope.

## Requirements

| ID | Source | Evaluation criterion / acceptance evidence |
| --- | --- | --- |
| REQ-P1-01 | HAND §Explicit Decisions: Phase 1; ARC §Phase 1 | Establish whether unauthenticated market REST access, asset search, an initial snapshot, and subsequent streaming updates are evidenced as distinct observable capabilities. Distinguish backend evidence from unavailable client behavior. |
| REQ-P1-02 | HAND §Phase 1; ARC §Observed components | Establish whether market-provider data and errors cross an Atra-owned normalization boundary and whether Binance is contained behind an adapter/replacement boundary rather than leaked as an application contract. |
| REQ-P1-03 | HAND §Phase 1; ARC §Phase 1 | Establish whether the baseline evidences upstream symbol-subscription aggregation, duplicate-subscription avoidance, update fan-out, and disconnect cleanup; bound any conclusion to the observed deployment/process scope. |
| REQ-P1-04 | HAND §Phase 1; ARC §Phase 1 | Establish whether consumers can distinguish fresh data, stale-but-usable data, and unavailable/error—not merely whether an internal cache has those paths. Record multi-instance/shared-state behavior as indeterminate unless baseline evidence supports it; Redis is not a Phase 1 MVP requirement. |
| REQ-P1-05 | HAND §Cross-phase and Phase 1; DEF §Constraints | Treat on-device MVP watchlists, preset presentation, client subscription sequencing, and client-side debouncing as iOS/client evidence only. Do not infer their conformance or absence from the service baseline; verify only that no backend persistence requirement is imposed by the visible source. |
| REQ-P2-01 | HAND §Cross-phase and Phase 2; ARC §Phase 2 | Establish whether account identity is persistent and distinct from wallet address, supports multiple verified wallet associations, and remains non-custodial and wallet-signature based rather than conventional credential based. |
| REQ-P2-02 | HAND §Phase 2; ARC §Phase 2 | Establish whether role behavior preserves exactly one `OWNER`, distinguishes `AUTH` from owner authority and `STANDARD` from implicit authentication, and permits at most one `RECOVERY`. Evidence must cover relevant alternate/concurrent paths or explicitly limit the conclusion. |
| REQ-P2-03 | HAND §Phase 2; ARC §Replay-control limitation | Establish whether signing challenges/nonces are server-generated, unpredictable, bounded in lifetime, purpose-bound, and single-use under concurrent verification. A separate lookup and later consume is not sufficient evidence of replay prevention; report the resulting risk if atomicity is unproven. |
| REQ-P2-04 | HAND §Phase 2; ARC §Identity/session boundary | Establish whether signature verification proves control of the claimed wallet in the documented Ethereum-compatible signing context, and whether the account/session outcome remains attributable to that verified wallet and its chain in multi-wallet accounts. |
| REQ-P2-05 | HAND §Phase 2; ARC §Phase 2 | Establish whether sessions provide short-lived access, longer-lived refresh/persistent-device support, server-side revocation, and non-plaintext server storage of refresh credentials. Mark iOS Keychain/persistent-device client handling indeterminate without an approved iOS evidence scope. |
| REQ-P2-06 | HAND §Phase 2; ARC §Authorization-context divergence | Establish whether wallet linking, role/recovery/ownership management, and session revocation derive authorization from the authenticated account/session and required wallet proof—not caller-controlled account or wallet identifiers. Ownership transfer must preserve the sole-owner invariant atomically. |
| REQ-P2-07 | HAND §Phase 2; ARC §Client and WebSocket limitations | Establish whether security-sensitive actions have audit evidence without unnecessary secret/raw-credential retention, and whether authenticated WebSocket context derives from the same account/session trust model as REST. If no server/client WebSocket evidence is in scope, record this as indeterminate rather than compliant. |
| REQ-P2-08 | HAND §Phase 2; ARC §Persistence and account roles | Establish whether the relational persistence boundary protects wallet/account uniqueness, challenge state, session revocation, and the role invariants under applicable alternate and concurrent writes. Application checks alone do not establish invariant durability where conflicting writes remain possible. |
| REQ-P25-01 | HAND §Phase 2.5; DEF §Constraints; ARC §Phase 2.5 | Limit Phase 2.5 conclusions to visible chain-awareness intent: assess whether chain identity is explicit and consistently carried across the in-scope identity, wallet, persistence, session, and API/provider boundaries examined. Do not assert required networks, chain families, or full portability absent source evidence. |
| REQ-P25-02 | HAND §Phase 2.5; ARC §Phase 2.5 | Record as bounded or indeterminate any claim about portfolio discovery, broader provider integration, client state, or non-EVM behavior unless it is supported by both the visible source and the verified in-scope baseline. Ethereum-family configuration/signing alone is neither proof of multi-chain sufficiency nor a defect under the available source. |
| REQ-EV-01 | DEF §Observable success conditions; OPLAN §Evidence and baseline approach | Each reported conclusion must identify its requirement ID, intended-state source, exact baseline evidence (or its absence), classification, and residual risk/uncertainty. It must not infer compliance from source silence, names, comments, or unverified repositories. |
| REQ-EV-02 | DEF §Constraints; OPLAN §Risks | Preserve the evidence limits in every relevant conclusion: only `Atra-Services` is in scope; no iOS repository is verified; detailed Phase 2.5 source content is truncated. These are evidence limitations, not product findings. |
| REQ-EV-03 | Contract v1.2 §Fixed mode; DEF §Constraints | Verify that evaluation remains read-only: no product/test/fixture/configuration/Git-history edits and no product PR. Findings, including material security gaps, are inputs to a separately approved workflow only. |

## Dependencies and scope boundaries

- Baseline: the manifest-recorded `Atra-Services` `main` commit `be51242593fe6643aff11c491d4f8a6c0835aed5`; a different commit cannot support conclusions under this contract.
- Intended-state evidence: only the visible material in `HAND-0001@v1`; detailed Phase 2.5 success criteria, non-goals, and supported networks remain unavailable.
- Product scope: no iOS or other repository is approved. Client-local watchlists, Keychain handling, persistent-device UX, and client WebSocket behavior remain outside evidentiary reach.
- Existing test evidence may be observed without modification, but absent, failing, or non-runnable evidence must be reported rather than repaired or replaced.

## Self-check and requested transition

`REQ-0001@v1` is traceable to the exact parent `ARC-0001@v1`, contains measurable read-only comparison criteria, preserves the approved scope and evidence limitations, and makes no product or workflow-stage decision. **READY_FOR_EVALUATION_TEST is requested for orchestrator validation; this artifact does not advance the workflow.**
