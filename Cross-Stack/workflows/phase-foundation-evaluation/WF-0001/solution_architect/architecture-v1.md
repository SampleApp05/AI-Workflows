---
artifact:
  id: ARC-0001
  type: ARCHITECTURE
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
    - DEF-0001@v1
  references:
    - path: problem_analyst/definition-v1.md
    - path: intake_analyst/handoff-v1.md
    - path: workflow_orchestrator/orchestration-plan-v1.md
ownership:
  created_by: solution_architect
  performed_by: solution_architect
  recorded_by: solution_architect
  engine: codex
  target_id: codex-native
---

# Architecture assessment — Atra foundational-phase evaluation

## Scope and evidence posture

This is an architecture assessment for the read-only evaluation defined by `DEF-0001@v1`, not a proposal to alter the product. The implementation evidence inspected is the verified `Atra-Services` baseline `be51242593fe6643aff11c491d4f8a6c0835aed5` (the checked-out `main` and `origin/main` tip at inspection). The product worktree was clean against that commit. The intended-state authority remains the visible material captured in `HAND-0001@v1`; repository names, comments, and tests are evidence, not specification.

The assessment is intentionally bounded to the service repository. No iOS repository is verified in scope, so client-side behavior—including on-device watchlists, search debouncing, Keychain use, and client state—cannot be established here. Detailed Phase 2.5 criteria remain unavailable because the source retrieval is truncated.

## Architectural goals used for comparison

The visible intended state calls for:

- a replaceable, normalized market-provider boundary with REST snapshots followed by WebSocket updates, subscription aggregation, freshness-aware behavior, and unauthenticated market access;
- a non-custodial, wallet-based account boundary in which accounts persist independently of wallet addresses, roles and ownership remain controlled, signatures prove wallet control, sessions are revocable, and sensitive operations are auditable; and
- chain awareness contained in shared concepts and durable records so that Ethereum-centric assumptions do not spread through the named foundations before portfolio-domain work expands.

These are evaluation criteria, not newly approved architectural decisions.

## Observed components and boundaries

| Area | Verified baseline evidence | Architectural reading |
| --- | --- | --- |
| Market transport and application boundary | `apps/market-service/src/index.ts:13-34` composes Binance REST/WS adapters, an in-memory cache, application services, Express REST, and a WebSocket gateway. | Transport, service orchestration, cache, and provider access are separated in a single-instance service. |
| Market provider boundary and owned model | `apps/market-service/src/adapters/BinanceRestAdapter.ts:8-43` maps raw Binance shapes into `MarketTicker`; the owned types are defined in `apps/market-service/src/types/index.ts:9-32`. | The provider schema is not exposed directly through this internal model boundary. |
| Market streaming boundary | `apps/market-service/src/services/MarketStreamManager.ts:21-43, 57-89, 94-179` tracks socket/symbol registries, subscribes upstream once for a new symbol, updates cache, fans out, and removes disconnected clients; `apps/market-service/src/ws/WebSocketGateway.ts:55-60, 66-80, 112-140` supplies transport-only connection handling. | The service has a clear per-process aggregation and fan-out boundary. |
| Market snapshot/search boundary | `apps/market-service/src/rest/router.ts:18-64` exposes unauthenticated search and price routes; `SymbolCatalogService.ts:22-49` loads a catalog once and searches it in memory; `PriceService.ts:27-61` implements fresh-cache, stale-while-revalidate, and cache-miss paths. | This separates catalog search from ticker retrieval and avoids an upstream lookup per search request after initialization. |
| Identity/session boundary | `apps/auth-service/src/index.ts:23-44` composes chain configuration, nonce/signature services, public identity/auth routes, and protected wallet/role/recovery/audit routes. `AccountService.ts:60-191`, `NonceService.ts:30-90`, and `SignatureService.ts:16-40` provide challenge issuance, signature verification, and provisioning. | Authentication is organized around durable wallet/account records rather than conventional account credentials. |
| Persistence and account roles | `packages/database/src/schema/wallets.ts:6-16` identifies wallets by address plus chain; `accounts.ts:7-13` holds an account separately; `accountWalletRoles.ts:9-27` defines OWNER, AUTH, STANDARD, and RECOVERY associations; `nonceChallenges.ts:8-28`, `sessions.ts:8-19`, and `auditLogs.ts:9-16` model challenge, session, and audit data. | PostgreSQL/Drizzle is the intended durable boundary and makes several Phase 2 concepts explicit. |
| Chain-awareness boundary | `apps/auth-service/src/shared/chains/chain.types.ts:3-15`, `chain.service.ts:11-98`, and `chain.constants.ts:8-25` centralize chain metadata/configuration; wallet identity and session claims carry numeric `chainId` (`wallets.ts:9-16`, `TokenService.ts:9-20`). | Chain identity is represented in shared configuration and key persistence/auth paths, although known definitions are Ethereum and Sepolia only. |

## Sufficiency, divergence, and uncertainty

### Phase 1 — Market Data Foundation

**Supported alignment.** The Market Service has an owned market model and explicit Binance adapters, separated REST and WebSocket paths, in-memory catalog search, cache-backed price retrieval, and per-process upstream subscription deduplication/fan-out/cleanup. The observed composition supports the source's provider-replaceability, REST-plus-streaming, and search-separation intent at a single service instance.

**Bounded sufficiency.** `PriceService` retains a stale ticker while refreshing (`apps/market-service/src/services/PriceService.ts:34-54, 89-97`), but its public `MarketTicker` has no freshness/status field (`types/index.ts:9-24`), and the REST handler returns either ticker data or a generic 502 (`rest/router.ts:58-63`). Thus the presence of an internal stale-while-revalidate policy is evidence for partial resilience, not evidence that consumers can distinguish fresh, stale-but-usable, and unavailable/error states as intended.

**Scale and availability boundary.** The only observed cache is `InMemoryPriceCache` (`cache/InMemoryPriceCache.ts:8-29`), while client and subscription registries are process-local (`MarketStreamManager.ts:24-32`). This is compatible with the source's MVP position that Redis was not initially mandated, but it leaves multi-instance shared cache/subscription behavior outside evidence; no claim of multi-instance sufficiency is warranted.

**Scope-limited unknowns.** No iOS code is in the verified scope. The observed service has no watchlist endpoint or persistence schema, which is consistent with the intended absence of backend-persisted MVP watchlists, but cannot demonstrate local watchlist behavior, presets, UI subscription sequencing, or debouncing.

### Phase 2 — Wallet Identity and Account Security

**Supported alignment.** Accounts, wallets, and role associations are separate durable concepts; wallet uniqueness includes `chainId`; role values cover the four source roles. The account-provisioning transaction creates an account, OWNER and AUTH associations, and an audit entry (`AccountService.ts:162-191`). Nonces use cryptographic randomness and a five-minute default, are persisted with purpose/expiry/used state (`NonceService.ts:10-50`; `.env.example:19-22`), and EIP-191-style message verification uses `ethers.verifyMessage` (`SignatureService.ts:16-40`). Session creation stores a refresh-token hash and expiry, embeds a short-lived access token, and writes audit records (`TokenService.ts:25-32, 74-90`; `SessionService.ts:46-87`); middleware checks session revocation/expiry (`middleware/authenticate.ts:52-99`). These are material architectural evidence toward the visible Phase 2 goals.

**Authorization-context divergence.** Protected-route middleware derives `req.auth`, but several controllers still accept `accountId` and `walletId` from the request body. This is stated directly for wallet linking (`WalletController.ts:25-35, 77-86`), roles (`RoleController.ts:15-17, 47-52`), and session revocation (`AuthController.ts:58-81`). In addition, `/auth` is mounted before the protected route middleware (`apps/auth-service/src/index.ts:35-44`). Consequently, the trust boundary does not consistently bind a privileged operation to the account and wallet proven by the session. The visible source expects privileged linking/management and REST/WebSocket user context to derive from the same account/session trust model; the baseline evidence shows this boundary is materially incomplete.

**Session-to-wallet uncertainty and risk.** A session records account and chain but not its authenticated wallet (`sessions.ts:8-19`). The JWT likewise holds account, session, roles, and chain but no wallet (`TokenService.ts:9-20`). Middleware then selects the first AUTH association for the account rather than the signing wallet (`authenticate.ts:83-97`), and refresh similarly selects an arbitrary first role association (`SessionService.ts:106-127`). This makes the active-wallet attribution and role binding indeterminate in multi-wallet accounts, even though the intended role model distinguishes their authority.

**Invariant enforcement divergence.** The schema has no observed database uniqueness or partial-uniqueness constraints for exactly one OWNER, at most one RECOVERY, or a unique `(accountId, walletId, role)` association (`accountWalletRoles.ts:20-27`; `accounts.ts:7-13`). `RoleService` checks the recovery count and performs ownership transfer inside a transaction (`RoleService.ts:184-226`), which is positive application-level evidence, but it is not durable database-level enforcement under all write paths/concurrency. `accounts.recoveryWalletId` is also present but not updated by the observed recovery assignment path; semantic alignment between that column and the role relation is therefore not established.

**Replay-control limitation.** Challenge verification queries an unused, unexpired nonce and then marks it used in a separate operation (`AccountService.ts:116-143`; equivalent patterns occur in `WalletLinkingService.ts:119-145` and `RoleService.ts:108-133`). The source's single-use requirement is represented, but the examined sequence is not an observed atomic compare-and-consume operation. Concurrent verification behavior is therefore a security-critical evaluation question, not a supported guarantee.

**Client and WebSocket limitations.** Comments describe a query-token path for WebSocket upgrade (`middleware/authenticate.ts:1-8, 108-121`), but no Auth Service WebSocket server or upgrade handler is present in the inspected repository tree. Combined REST/WebSocket account-context behavior cannot be verified. Secure iOS Keychain handling and persistent-device client behavior are outside the only verified repository scope.

### Phase 2.5 — Multi-Chain Foundation

**Supported alignment.** Chain configuration is centralized, enabled chains are validated at identity challenge creation (`AccountService.ts:60-67`), and `chainId` is carried through wallet identity, sessions, and token claims. Address uniqueness is scoped by chain (`wallets.ts:6-16`). These are concrete signs that chain information has not been left solely as an implicit Ethereum global in the inspected identity and persistence paths.

**Bounded divergence/uncertainty.** The only known chain definitions are Ethereum and Ethereum Sepolia (`chain.constants.ts:8-25`), and signing uses the EVM-oriented `ethers.verifyMessage` interface (`SignatureService.ts:20-39`). That does not establish a defect: visible intended state includes Ethereum-compatible signing and the full Phase 2.5 target is unavailable. It does show that the observed abstraction has only been evidenced for Ethereum-compatible chains and has not demonstrated portability across other chain families or the source-named portfolio/provider/client-state concerns. No portfolio or iOS scope is available to assess those areas.

## Recommended evaluation direction

Retain the existing component boundaries as the comparison map rather than selecting a replacement architecture: Market transport/service/cache/provider; identity/signature/session/authorization; persistence/roles/audit; and shared-chain configuration. The next evaluation artifacts should test and independently review the boundary claims above against the exact baseline, prioritizing evidence for: externally observable freshness/error signaling; authorization provenance for wallet, role, and revoke actions; nonce compare-and-consume concurrency; role/account invariants under concurrent or alternate paths; session-to-wallet attribution; and which chain-aware paths are actually covered.

This is an evaluation recommendation only. It neither prescribes remediation nor authorizes implementation, requirements, work decomposition, routing, or a product change.

## Consequences and trade-offs

- The observed Market Service deliberately favors simple in-process state and a single provider boundary. That lowers MVP complexity and supports the visible source position on initial Redis deferral, while limiting evidence for multi-instance continuity.
- The observed Auth Service models the desired concepts in relational tables and service boundaries, enabling focused evaluation, but application-level authorization and invariants require stronger evidence than type/schema names alone provide.
- Carrying `chainId` through shared configuration and records avoids some Ethereum-global coupling, while EVM-specific signing and only Ethereum-family configured entries limit what this run can conclude about broader multi-chain readiness.
- The backend-only scope prevents a whole-product finding. Any iOS behavior, credential storage, or local-watchlist conclusion must remain explicitly unknown rather than be inferred from absence.

## Risks

1. **Material authorization risk:** request-body account/wallet identifiers and the publicly mounted revoke route create evidence that privileged actions may not consistently derive from the verified session context.
2. **Material replay/invariant risk:** non-atomic observed nonce consumption and absent observed database constraints can leave correctness dependent on service sequencing and untested concurrency behavior.
3. **Freshness-contract risk:** internal cache behavior may be mistaken for a consumer-visible freshness contract even though observed response types do not communicate freshness state.
4. **Scope risk:** service-only evidence can be overgeneralized to an iOS-first product; it cannot substantiate local storage, Keychain, client WebSocket, or UI behavior.
5. **Source-completeness risk:** the visible Phase 2.5 material is architectural intent only; detailed requirements, networks, and non-goals are unavailable.
6. **Read-only risk:** findings may identify material concerns but cannot be corrected in this evaluation run.

## Assumptions and human decisions required

### Assumptions

- The recorded baseline SHA is the only implementation state assessed here; a later checkout or remote change is outside this artifact's evidence.
- The source material reproduced in `HAND-0001@v1` is the complete usable intended-state evidence for this run.
- The lack of an observed iOS repository is a scope limitation, not evidence that the client is absent or nonconformant.

### Decisions requiring human authority

1. Whether to provide the complete authoritative Phase 2.5 source before treating conclusions beyond visible chain-awareness intent as decision-ready.
2. Whether a separately approved evaluation scope extension should include the iOS repository, particularly for on-device watchlists, Keychain credential storage, and client WebSocket behavior.
3. After the evaluation Test and Review evidence is complete, whether any material findings warrant a separately approved delivery workflow. This artifact does not choose that follow-up or its solution.

## Self-check

`ARC-0001@v1` preserves parent lineage to `DEF-0001@v1`, fixed evaluation mode, the verified baseline, and the stated source/iOS evidence limits. It records observed architecture and bounded evaluation direction only; it does not create requirements, work units, routes, implementation changes, or approvals.
