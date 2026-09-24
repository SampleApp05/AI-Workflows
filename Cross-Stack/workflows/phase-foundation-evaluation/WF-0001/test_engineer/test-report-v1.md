---
artifact:
  id: TEST-0001
  type: TEST_REPORT
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
    - REQ-0001@v1
  references:
    - path: requirements_engineer/requirements-v1.md
    - path: solution_architect/architecture-v1.md
    - path: problem_analyst/definition-v1.md
    - path: workflow_orchestrator/orchestration-plan-v1.md
    - product_repository: Atra-Services
      branch: main
      commit: be51242593fe6643aff11c491d4f8a6c0835aed5
ownership:
  created_by: test_engineer
  performed_by: test_engineer
  recorded_by: test_engineer
  engine: codex
  target_id: codex-native
---

# Test Report — Atra foundational-phase evaluation

## Mode, baseline, and outcome

- **Mode:** `evaluation` (read-only).
- **Tests added or changed:** none.
- **Product baseline:** `Atra-Services` `main` / `origin/main` both resolved to `be51242593fe6643aff11c491d4f8a6c0835aed5` before and after testing. `git status --short --branch`, `git diff --exit-code`, and `git diff --cached --exit-code` showed no product changes.
- **Verdict:** **FAIL**. All three existing deterministic suites passed, but read-only acceptance comparison found material divergences in authenticated-action provenance, session-to-wallet attribution, nonce replay atomicity, and subscription accounting. Passing mocked unit suites do not establish those requirements.

The intended-state authority is `HAND-0001@v1`; implementation names, comments, and tests are baseline evidence only. Phase 2.5 conclusions are limited to the source's visible chain-awareness intent. The only inspected product repository is `Atra-Services`; no iOS repository was in scope.

## Tests executed

The first sandboxed market command could not create Vitest's ignored, ephemeral `.vite-temp` configuration bundle (`EPERM`); it made no source or Git change. The same existing command was then run with the runtime cache permission needed by Vitest. No test, fixture, configuration, or product source was edited.

| Command | Result | Evidence / scope |
| --- | --- | --- |
| `npm test --workspace=@atra/market-service` | PASS — 9 files, 84 tests | Vitest 4.1.11; mocked adapter, REST-router, WebSocket gateway/stream-manager, cache, and integration-style in-process coverage. |
| `npm test --workspace=@atra/auth-service` | PASS — 18 files, 182 tests | Vitest 4.1.11; service/controller/middleware tests use mocked DB/service boundaries. Warnings: a future Vite `__dirname` config-loader incompatibility and a hoisted nested `vi.mock("ethers")` that will become an error. |
| `npm test --workspace=@atra/database` | PASS — 7 files, 75 tests | Vitest 4.1.11; schema/module tests. No live PostgreSQL migration or concurrent database test was run. |

No command contacted a market provider, opened an external WebSocket, ran a live service, accessed an iOS client, or exercised a live database. Build and coverage commands were not run because they generate product-repository output and are not necessary for this read-only evaluation.

## Requirement coverage and findings

Classifications are against `REQ-0001@v1`; **supported** means the bounded evidence is present, **divergent** means the inspected baseline contradicts a criterion, and **indeterminate** means the approved evidence cannot establish it.

| Requirement | Classification | Verified evidence and test assessment | Residual risk / limitation |
| --- | --- | --- | --- |
| REQ-P1-01 | SUPPORTED (backend only) | `market-service/src/rest/router.ts:22-63` exposes unauthenticated search and price endpoints; `src/index.ts:7-29` creates REST plus WebSocket paths. The 84 passing market tests cover router inputs/errors, cached initial retrieval, stream manager, and gateway messages. | No iOS client evidence establishes client snapshot-then-subscribe sequencing, presentation, or reconnect behavior. |
| REQ-P1-02 | SUPPORTED | `BinanceRestAdapter.ts:25-69` and `BinanceWsAdapter.ts:1-143` normalize provider fields to `MarketTicker`; service and router consume Atra-owned types. Adapter tests passed. | Only Binance was exercised through mocks; replacement-provider compatibility is structural, not integration-proven. |
| REQ-P1-03 | DIVERGENT | `MarketStreamManager.ts:93-119` prevents duplicate subscribe and tests cover normal multi-socket/unsubscribe/disconnect paths. But `_unsubscribeOne` always calls `_removeSocketFromSymbol` (`124-125`), and `_removeSocketFromSymbol` decrements `refCount` after an unchecked `Set.delete` (`137-148`). An unsolicited or repeated unsubscribe for an existing symbol can decrement a count for a socket that was never subscribed; a later valid unsubscribe can remove the upstream subscription while another subscriber remains. | Existing test `is a no-op for a symbol the socket was never subscribed to` covers only an absent symbol entry, not an existing entry held by another socket. This can interrupt live updates. Per-process registries also leave multi-instance aggregation indeterminate. |
| REQ-P1-04 | DIVERGENT | `PriceService.ts:34-53` has internal fresh/stale branches and returns the same `MarketTicker`; background failures are intentionally swallowed (`89-96`). `/prices` returns bare tickers or a 502 (`router.ts:58-63`). | Consumers cannot distinguish fresh, stale-but-usable, or unavailable states from a stated response contract. Cache is in-memory, so shared/multi-instance freshness is indeterminate. |
| REQ-P1-05 | INDETERMINATE | Backend has no watchlist persistence and 84 market tests contain no client implementation evidence. | This neither proves nor disproves on-device watchlists, presets, debouncing, or client subscriptions. No iOS repository is approved. |
| REQ-P2-01 | SUPPORTED (bounded) | `AccountService.ts:105-187` provisions persistent account/wallet records after signature verification; `WalletLinkingService.ts:54-165` supports additional verified wallet links as `STANDARD`. `SignatureService.ts:16-39` uses Ethereum-compatible `ethers.verifyMessage`; no conventional credential flow was observed. Auth tests (182) exercise these mocked paths. | Existing-account lookup is by `accounts.ownerWalletId` (`AccountService.ts:145-159`), so an account whose relevant association is not its owner remains insufficiently evidenced. No live relational test was run. |
| REQ-P2-02 | DIVERGENT | Role service has application-level checks and transactional role operations; 182 auth tests cover nominal role flows. The inspected schema source only defines columns/FKs in `accountWalletRoles.ts:20-27`; it declares no uniqueness/check constraint for one owner or one recovery, nor for duplicate role grants, and no migration SQL is present in the baseline. | Concurrent/alternate writes can violate sole-owner, max-recovery, or duplicate-role invariants despite application checks. Existing tests use mocks and do not exercise concurrent DB transactions; a separately managed deployed schema is outside the available evidence. |
| REQ-P2-03 | DIVERGENT | Challenges use `randomBytes(16)`, purpose, expiry, and `usedAt` (`NonceService.ts:24-90`). Verification selects an unused row then separately calls `markUsed` (`AccountService.ts:120-143`); `markUsed` updates by id without `usedAt IS NULL` (`NonceService.ts:85-90`). | Two concurrent verifications can both read the same unused nonce before either update. No atomic compare-and-consume and no concurrent replay test were observed. This is a material replay-control gap. |
| REQ-P2-04 | DIVERGENT | Signature comparison is wallet-address based, and chainId flows through wallet lookup and issued JWT. However sessions persist accountId and chainId but no walletId (`sessions.ts:8-19`); middleware later selects the first `AUTH` role for the account instead of deriving the verified signing wallet (`authenticate.ts:83-97`). | A multi-wallet session is not durably attributable to the wallet that established it; audit/authorization provenance can be wrong. Tests assert this selection behavior rather than binding a session to its signer. |
| REQ-P2-05 | SUPPORTED (server only) | Token service hashes opaque refresh values; session schema stores `refreshTokenHash`, expiry, and `revokedAt` (`sessions.ts:8-18`). `SessionRepository.ts:26-85` checks active/expiry and supports revocation; token/session tests passed. | No iOS Keychain, persistent-device UX, race-safe refresh rotation, or live database persistence evidence is in scope. |
| REQ-P2-06 | DIVERGENT | Protected routes mount middleware (`auth-service/src/index.ts:31-35`), but wallet controller explicitly accepts `accountId` and `walletId` from the request body (`WalletController.ts:30-60`, `80-118`), while auth context is only used for chainId on verify. Role and recovery controllers likewise take account/wallet authority identifiers from body. `/auth/revoke` is publicly mounted (`index.ts:28`) and accepts body accountId (`AuthController.ts:58-82`). | Sensitive authorization does not consistently derive from the authenticated session/wallet. Controller tests pass because they intentionally provide caller-controlled IDs and do not test body/session mismatch. This is a material authorization-control divergence. |
| REQ-P2-07 | INDETERMINATE | Write-side audit inserts exist in account/wallet/role/recovery/session services and `AuditController` requires `req.auth.accountId` to match its path. No application WebSocket auth implementation was observed in scope. | Audit logs can misattribute wallet actor because session wallet attribution is divergent. Cursor pagination compares UUID IDs while ordering by timestamp (`AuditLogService.ts:31-55`), an untested logic concern that can cause missing/duplicated pages. REST/WebSocket trust consistency is not evidenced. |
| REQ-P2-08 | DIVERGENT | Wallet address+chain has a unique index (`wallets.ts:8-16`), but the inspected role and nonce schema source declares no equivalent unique/conditional constraints (`accountWalletRoles.ts:20-27`, `nonceChallenges.ts:20-28`) and has no migration SQL. The application uses transactions for some role/provision operations only. | Durable ownership/recovery uniqueness and nonce single-use are not proven under alternate/concurrent writes. Schema tests passed but do not run a PostgreSQL migration or concurrent transactions; a separately managed deployed schema is outside available evidence. |
| REQ-P25-01 | SUPPORTED (visible scope only) | Chain configuration is centralized (`chain.service.ts:20-101`); address uniqueness includes chain; account, wallet, session and JWT code carry chainId. Chain service tests passed. | The observed chain definitions are Ethereum and Sepolia only (`chain.constants.ts:8-25`) and signature verification is EVM-specific. This is not a defect under the truncated source, but does not establish broader portability. |
| REQ-P25-02 | INDETERMINATE | No approved client or broader provider/portfolio evidence establishes cross-chain discovery, client state, non-EVM behavior, or full portability. | Detailed Phase 2.5 success criteria, non-goals, and network coverage were truncated from the authoritative source. Ethereum-family evidence alone is neither proof nor defect. |
| REQ-EV-01 | SUPPORTED | This report maps each criterion to exact baseline evidence or an explicit absence, with classification and uncertainty. | Conclusions are bounded by source, scope, and test-environment evidence. |
| REQ-EV-02 | SUPPORTED | All client/iOS and detailed Phase 2.5 conclusions above preserve the declared evidence boundaries. | These limitations are not product findings. |
| REQ-EV-03 | SUPPORTED | Evaluation produced this report only; tests added/changed are none; baseline Git checks before/after test execution were clean; product PR is not applicable by fixed evaluation mode. | Vitest created only ignored ephemeral runtime cache files while running existing suites with permission; no tracked or product source/test/fixture/config/Git history change occurred. |

## Failure classification

| Item | Classification | Evidence |
| --- | --- | --- |
| Test commands | No test failure | 341/341 existing tests passed across market, auth, and database packages. |
| Test-environment first attempt | Environment restriction | Sandbox denied Vitest's ephemeral `.vite-temp` config bundle. Re-running unchanged test commands with the cache permission succeeded. |
| Auth test warnings | Test/toolchain maintenance risk | Vite warns `__dirname` use will be unsupported by the future native config loader; Vitest warns nested `vi.mock("ethers")` will become an error. Neither failed this run. |
| P1 subscription accounting | Implementation defect | Unchecked removal decrement in `MarketStreamManager.ts:124-148`; existing tests miss the existing-symbol/non-subscriber case. |
| P2 trusted authorization provenance | Implementation defect | Controllers accept sensitive account/wallet IDs from request body rather than verified context; revoke route is public. |
| P2 nonce single-use and relational role invariants | Implementation defect / coverage gap | Separate read/update nonce flow and absent relevant database constraints; no real concurrent DB test. |
| P2 session-wallet attribution | Implementation defect | Session schema omits walletId and middleware resolves an arbitrary account `AUTH` wallet. |

## Coverage gaps, edge cases, and risks

- No test exercises an existing symbol held by one socket when a different/unsubscribed socket sends unsubscribe; this is the direct P1-03 defect path.
- No end-to-end protected-route test compares JWT account/wallet context with conflicting body accountId/walletId, and no test verifies `POST /auth/revoke` cannot be invoked without session-derived identity.
- No test simulates two valid nonce verifications racing, concurrent owner transfer/recovery assignment, duplicated role grants, or real PostgreSQL constraint enforcement.
- No test proves a session maps to its signing wallet through refresh, middleware, audit, role, or wallet-link actions.
- No test makes freshness state visible to a market API consumer or verifies shared/multi-instance stream/cache behavior.
- No live DB, real provider, production server, authenticated WebSocket, iOS client, portfolio-discovery, non-EVM, or full Phase 2.5 source evidence was available. These are evidence gaps, not a claim that absent systems fail.

## Risks and next-stage handoff

The most consequential findings are replay exposure, caller-controlled authorization identifiers, and missing durable wallet attribution. Subscription-accounting and audit-pagination concerns can respectively interrupt market updates and weaken audit retrieval. This evaluation does not prescribe or implement remediation; these findings are inputs to the independent read-only Review and, only if later chosen by a human, a separately approved delivery workflow.

## Self-check and requested validation

`TEST-0001@v1` is in the assigned `test_engineer` path, has exact `REQ-0001@v1` lineage, records the verified baseline, test commands/results, classification, coverage gaps, edge cases, risks, and fixed evaluation limits. It made no product/test/fixture/configuration/Git-history changes and did not approve a gate, merge, or PR. **READY_FOR_EVALUATION_REVIEW is requested for orchestrator validation; this report does not advance the workflow.**
