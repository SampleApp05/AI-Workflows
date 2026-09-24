---
artifact:
  id: HAND-0001
  type: HANDOFF
  version: 1
  status: DRAFT
workflow:
  id: WF-0001
  project: Atra
  technology: Cross-Stack
  feature: phase-foundation-evaluation
  mode: evaluation
lineage:
  parents: []
  sources:
    - type: chatgpt_thread
      id: 6aaebb64-ecc0-83ed-a8e7-29ed8d241dab
      title: Current Atra Phases Summary
      role: authoritative intended-state reference
ownership:
  created_by: intake_analyst
  performed_by: intake_analyst
  recorded_by: intake_analyst
  engine: codex
  target_id: codex-native
---

# Handoff — Atra foundational-phase evaluation

## Intent

**Explicit request.** Run an **evaluation-mode** workflow based on the ChatGPT thread *Current Atra Phases Summary* (`6aaebb64-ecc0-83ed-a8e7-29ed8d241dab`). The evaluation is read-only and compares Atra's existing implementation with the thread's intended state for Phase 1, Phase 2, and Phase 2.5.

**Source status.** The thread is the authoritative intended-state reference recorded in the run manifest; the repository is evidence to inspect rather than the specification. The thread was retrievable but its long assistant response was truncated through the available reader. This Handoff records only content visible in that retrieval and does not reconstruct unavailable text.

## Problem / Motivation

**Documented fact.** The source asks downstream work to determine what has been implemented, where it differs from the intended design, what is incomplete, what may be obsolete or unnecessarily complex, and what may need attention before Phase 3 Portfolio. It explicitly says not to treat the repository automatically as the specification.

**Inference.** A disciplined read-only comparison is needed to establish a trustworthy baseline before any future delivery decision; this evaluation does not authorize that future work.

## Goal

Produce traceable, evidence-based evaluation artifacts that compare the existing Atra implementation against the documented intended state for:

- Phase 1 — Market Data Foundation;
- Phase 2 — Wallet Identity & Account Security; and
- Phase 2.5 — Multi-Chain Foundation.

The outcome must distinguish verified implementation evidence, documented intended-state facts, gaps or uncertainty, and residual risk. No remediation is to be designed or implemented in this run.

## Scope

### In

- Read-only inspection of the existing Atra implementation in the verified product-repository scope recorded by the run manifest: `Atra-Services` (`https://github.com/SampleApp05/Atra-Services.git`), base branch `main`.
- Comparison against the source's visible intended-state material for the three named phases.
- Identification and reporting of observed alignment, divergence, incompleteness, edge cases, test gaps, logic gaps, uncertainty, and residual risk.
- Explicit accounting for the source-access limitation and for any unavailable baseline evidence.

### Out

- Product-code, configuration, infrastructure, documentation, test, or repository edits.
- Architecture selection, requirements design, work decomposition, execution routing, implementation, or fix proposals.
- Product pull requests, product commits, merges, or deployments.
- Phase 3 Portfolio evaluation except where the source identifies it as future context.
- Any assertion that an unobserved source detail or repository state is implemented, missing, correct, or approved.

## Known Constraints

- **Explicit decision:** Workflow mode is `evaluation`; Contract v1.2 makes evaluation read-only and excludes decomposition, routing, execution, product edits, and product pull requests.
- **Explicit decision:** The three intended-state areas are Phase 1 market data, Phase 2 wallet identity/security, and Phase 2.5 multi-chain foundation.
- **Explicit decision:** Atra is intended to be view-only and non-custodial in these phases; transaction execution is outside the current scope.
- **Explicit decision:** The source, not the repository, is the intended-state reference.
- **Known limitation:** The accessible thread response truncates before the full Phase 2.5 description. Detailed Phase 2.5 behavior and criteria not visible in the retrieved text must be recorded as unavailable, not inferred.
- **Unknown:** No verified baseline commit SHA, checkout state, or accessible iOS repository is supplied in this Handoff.

## Explicit Decisions

The following are documented intended-state decisions visible in the source; they are evaluation criteria, not implementation claims.

### Cross-phase

- Atra is an iOS-first cryptocurrency portfolio and market application built around wallet-based identity.
- Atra must not custody private keys, seed phrases, signing keys, or funds, and must not require conventional email/password/username/social-login accounts.
- Atra-owned domain models must insulate application layers from external-provider schemas; providers remain replaceable dependencies.
- MVP market watchlists are on-device, are not backend-persisted, and work without authentication.

### Phase 1 — Market Data Foundation

- Unauthenticated users can inspect market information, including preset lists, custom local watchlists, asset search, an initial REST snapshot, and subsequent WebSocket updates.
- The intended data pattern is REST snapshot followed by WebSocket subscription and incremental updates.
- The backend normalizes provider data and uses Atra-owned errors rather than leaking provider schemas/errors. Binance is the selected Phase 1 provider behind an adapter boundary.
- The backend is intended to aggregate subscriptions, deduplicate upstream symbol subscriptions, fan out updates, and clean up disconnects.
- Cache/freshness behavior is explicit: fresh, stale-but-usable, or unavailable/error. Redis was considered for later multi-instance shared state, not mandated for initial MVP operation.
- Search is distinct from watchlists and should not rely on an expensive provider call for each keystroke; client debouncing is helpful but not a correctness prerequisite.

### Phase 2 — Wallet Identity & Account Security

- An Atra account is persistent and independent of a wallet address; one account can associate multiple verified wallets.
- Wallet roles are distinct: exactly one `OWNER`, `AUTH` for account authentication without owner authority, `STANDARD` for operational/portfolio association without implicit authentication, and at most one optional `RECOVERY` wallet.
- Authentication uses server-generated, unpredictable, short-lived, single-use nonce/challenges and wallet signatures; replay must be prevented. The source identifies Ethereum-compatible message signing such as EIP-191 `personal_sign` as intended.
- Successful verification establishes a session with a short-lived JWT access token, longer-lived refresh mechanism, persistent-device support, and server-side revocation capability. Refresh credentials must not be stored plaintext server-side; iOS sensitive credentials belong in Keychain.
- Wallet linking requires proof of control by the authenticated account context and candidate wallet; privileged management is owner-authorized. Ownership transfer must preserve exactly one owner atomically.
- Security-sensitive actions require audit information without unnecessary secret or raw-credential storage. Authenticated WebSocket user context should derive from the same account/session trust model as REST.
- PostgreSQL is the expected relational persistence layer; exact schema is not decided, while role, challenge, uniqueness, and revocation invariants are important.

### Phase 2.5 — Multi-Chain Foundation

- The visible source states that the original implementation began with an Ethereum-centric assumption.
- It states Phase 2.5 is intended to make existing foundations chain-aware before blockchain data becomes a major domain layer, avoiding chain-specific assumptions spreading through authentication, wallets, database records, APIs, portfolio discovery, provider integration, and client state.
- The visible text characterizes this as primarily an architectural foundation rather than a large user-facing phase.

## Assumptions

- **Assumption:** The manifest's `Atra-Services` repository is the product baseline to inspect unless later verified run evidence expands the scope. This is not evidence that it contains all intended client layers.
- **Assumption:** The current `main` baseline will be resolved and recorded by later stages before they make repository-state claims; no commit is assumed here.
- **Inference:** Because the source is partially inaccessible, evaluation conclusions for detailed Phase 2.5 behavior can only be bounded by the visible chain-awareness intent and must retain uncertainty.

## Open Questions

- What exact baseline commit and checkout constitute the existing implementation for this evaluation?
- Is an iOS client repository or other product repository in scope? The source describes an iOS-first product, but only `Atra-Services` is recorded in the manifest.
- What are the full intended Phase 2.5 responsibilities, success criteria, non-goals, and any chain/network coverage? The accessible source text is truncated before those details.
- Does the authoritative chat remain accessible in a non-truncated form, or should later stages treat the limitation as a standing evidence gap?
- What precise behavior is intended for the optional recovery procedure? The visible source says it may need later requirements definition.

## Expected Outcome

An evaluation-only record of the verified existing baseline against documented intended state, including:

- implemented behavior supported by repository evidence;
- mismatches, omissions, test gaps, logic gaps, and edge cases;
- source or baseline uncertainty called out explicitly;
- residual risks and matters requiring a separately approved follow-up.

This Handoff does not approve a plan, select an architecture, define requirements, prescribe fixes, or authorize product changes.
