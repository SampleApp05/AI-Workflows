---
artifact:
  id: DEF-0001
  type: DEFINITION
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
    - HAND-0001@v1
    - HSUM-0001@v1
  references:
    - path: intake_analyst/handoff-v1.md
    - path: intake_analyst/handoff-summary-v1.md
    - path: workflow_orchestrator/orchestration-plan-v1.md
ownership:
  created_by: problem_analyst
  performed_by: problem_analyst
  recorded_by: problem_analyst
  engine: codex
  target_id: codex-native
---

# Definition — Atra foundational-phase evaluation

## Problem statement

The available implementation evidence must be assessed against the intended state recorded in *Current Atra Phases Summary* before any decision about later Phase 3 Portfolio work or remediation. The repository is evidence, not the specification. The evaluation must establish what can be supported by observed baseline evidence, what differs or remains uncertain, and the residual risks for the intended foundations of:

- Phase 1 — Market Data Foundation;
- Phase 2 — Wallet Identity & Account Security; and
- Phase 2.5 — Multi-Chain Foundation.

This is an evaluation-mode problem only. It does not authorize a design decision, a product change, or a remedy.

## Context and evidence boundary

The authoritative intended-state reference is the recorded ChatGPT thread *Current Atra Phases Summary* (`6aaebb64-ecc0-83ed-a8e7-29ed8d241dab`), as preserved in `HAND-0001@v1`. Its retrieved response is truncated: Phase 1 and Phase 2 are visible in detail, while only the high-level Phase 2.5 chain-awareness rationale is visible.

The recorded product evidence scope is `Atra-Services` at verified `main` baseline `be51242593fe6643aff11c491d4f8a6c0835aed5`. A read-only tree inspection identifies market-service, auth-service, shared chain, and database account/wallet areas as potentially relevant evidence locations; this is context only, not a finding about their behavior or completeness. No iOS repository is verified in scope despite the intended product being iOS-first.

## Desired outcomes

- A traceable, evidence-based account of observable alignment, divergence, incompleteness, edge cases, test gaps, logic gaps, and residual risk for the three named phases.
- A clear distinction among documented intended-state facts, verified baseline evidence, assumptions, inferences, and unknowns.
- Bounded conclusions: detailed Phase 2.5 claims remain limited to visible chain-awareness intent, and no iOS implementation claim is made without a verified iOS scope addition.
- A decision-ready baseline for a separately approved follow-up, if one is warranted, without selecting or prescribing any follow-up solution.

## In scope

- Read-only comparison of the verified `Atra-Services` baseline with the visible intended-state material for Phases 1, 2, and 2.5.
- Evaluation of visible Phase 1 intent: unauthenticated market access, local/on-device watchlists, search, REST snapshot followed by WebSocket updates, normalized provider boundaries, subscription handling, freshness states, and resilience expectations.
- Evaluation of visible Phase 2 intent: persistent wallet-based accounts, wallet-role invariants, replay-safe signed challenges, session and revocation expectations, wallet linking and ownership controls, auditability, consistent REST/WebSocket trust, and relational-persistence invariants.
- Evaluation of visible Phase 2.5 intent: whether existing foundations avoid unbounded Ethereum-centric assumptions across the named identity, wallet, persistence, API, provider, portfolio-discovery, and client-state concerns.
- Read-only examination of existing test evidence and reporting of uncertainty, gaps, and residual risk.

## Out of scope

- Product, test, configuration, infrastructure, documentation, or repository changes.
- New tests, fixtures, test-data changes, or changes to test execution behavior.
- Architecture selection or redesign; requirements design; decomposition; routing; implementation; remediation proposals; or task assignment.
- Product commits, pull requests, merges, deployments, and release decisions.
- Evaluation of Phase 3 Portfolio other than its stated dependency on these foundations.
- Assertions about unobserved chat content, an unverified repository, or behavior not supported by baseline evidence.

## Observable success conditions

The evaluation is successful when downstream evaluation artifacts can, without product edits:

1. Compare each visible intended-state concern with specific, verified baseline evidence or explicitly record that evidence is unavailable.
2. State whether a conclusion is supported, limited, or indeterminate, rather than converting repository structure or source omissions into implementation claims.
3. Identify observable mismatches, omissions, test/coverage gaps, logic/edge-case concerns, and residual risks where evidence supports them.
4. Preserve the non-custodial, view-only, wallet-centric identity, provider-boundary, and chain-awareness principles as comparison context, not implementation instructions.
5. Retain the Phase 2.5 truncation and absent iOS scope as explicit evidence limitations in final conclusions.
6. Leave the product baseline unchanged and make no product pull request; in evaluation mode, those outcomes are not applicable.

## Affected areas

Potential baseline evidence areas are limited to the recorded `Atra-Services` repository and include market-data service boundaries, wallet/account/security service boundaries, shared chain concepts, and account/wallet persistence records. These labels identify where evidence may be observed; they do not determine a design, imply coverage, or expand the approved product scope.

## Constraints

- Workflow mode is fixed as `evaluation` under Contract v1.2 and is read-only.
- The source thread, not the repository, is the intended-state authority.
- Atra's visible intended state is view-only and non-custodial: no custody of private keys, seed phrases, signing keys, or funds; conventional email/password/social-login identity is not the intended basis for these phases.
- Detailed Phase 2.5 behavior, success criteria, non-goals, and supported networks are unavailable from the accessible source and must not be inferred.
- The verified evidence scope is only `Atra-Services`; an iOS-first intended product does not establish an iOS repository as in scope.
- The run must not advance by altering source material, product code, tests, fixtures, configuration, or Git history.

## Dependencies

- The exact approved plan and plan gate: `OPLAN-0001@v1` and `HAPP-0001@v1`.
- The source-bound interpretation in `HAND-0001@v1` and `HSUM-0001@v1`.
- Read-only accessibility of the recorded baseline `be51242593fe6643aff11c491d4f8a6c0835aed5` in `Atra-Services`.
- Existing safe, deterministic test evidence where it can be observed without modification; inability to obtain it is itself evidence to record.

## Risks

- **Material source-completeness risk:** conclusions about detailed Phase 2.5 functionality can overstate certainty because the authoritative retrieval is truncated.
- **Material scope risk:** backend-only evidence can be mistaken for a whole-product conclusion although no iOS repository is verified.
- **Baseline-evidence risk:** repository location or naming may be mistaken for behavioral proof unless subsequent findings cite observable content at the verified commit.
- **Read-only risk:** significant defects or gaps may be identified but cannot be corrected in this run.
- **Security-assessment risk:** absence of visible evidence for sensitive identity or credential behavior cannot establish safety; it requires an explicitly limited conclusion.

## Assumptions

- The manifest-recorded baseline is the baseline to be evaluated unless the controller records a valid, approved scope or baseline change.
- The source content reproduced in `HAND-0001@v1` is the complete usable intended-state evidence for this run; unavailable source content remains unknown.
- Existing files and tests may be inspected as evidence only; their presence alone does not establish intended behavior, correctness, or coverage.
- A future delivery workflow, if needed, requires separate human approval and is outside this Definition.

## Open questions

### Non-blocking evidence questions

- Can the complete authoritative Phase 2.5 source be made available, including responsibilities, success criteria, non-goals, and chain/network coverage?
- Is an iOS client repository or another product repository intended to be included in a separately approved scope extension?
- What is the full intended procedure and authorization model for the optional recovery wallet?
- Which observed tests can be safely and deterministically run in the available environment, and what evidence will remain unavailable if they cannot?

No unresolved question blocks the approved bounded evaluation: the run can proceed only with explicit limitations for the unavailable source and scope evidence.

## Self-check

This draft preserves the approved evaluation-only mode, exact upstream lineage, verified baseline reference, and source/scope limitations. It states comparison outcomes and observable success conditions without choosing an architecture, defining implementation requirements, prescribing fixes, assigning work, routing execution, or advancing any workflow gate.
