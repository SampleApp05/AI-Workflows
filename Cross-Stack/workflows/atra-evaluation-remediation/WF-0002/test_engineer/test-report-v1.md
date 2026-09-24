---
artifact:
  id: TEST-0002
  type: TEST_REPORT
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
    - REQ-0002@v1
    - ROUTE-0002@v2
  references:
    - path: requirements_engineer/requirements-v1.md
    - path: execution_router/routing-plan-v2.md
    - product_repository: Atra-Services
      pull_request: 3
      url: https://github.com/SampleApp05/Atra-Services/pull/3
      base: main
      head: workflow/WF-0002-atra-evaluation-remediation
      commit: 3dc6d42e351267390cc1afbcfbac3fe5fee3300b
ownership:
  created_by: test_engineer
  performed_by: test_engineer
  recorded_by: test_engineer
  engine: codex
  target_id: codex-native
---

# Test Report — Atra evaluation remediation

## Outcome

- **Mode:** delivery.
- **Product under test:** Atra-Services PR #3, `main` ← `workflow/WF-0002-atra-evaluation-remediation`, head `3dc6d42e351267390cc1afbcfbac3fe5fee3300b`; verified OPEN.
- **Product changes made by this stage:** none.
- **Verdict:** **FAIL**. All focused requirement evidence and both package builds pass, including live disposable-PostgreSQL evidence. The complete market workspace suite has one failure because its existing integration assertion still expects the intentionally replaced bare `/prices` contract. That stale assertion is a test defect, but an incomplete green package suite prevents Test-stage PASS.

## Executed evidence

| Command | Result | Observed evidence |
| --- | --- | --- |
| `npm run build --workspace @atra/market-service` | PASS | TypeScript build passed. |
| `npm run test --workspace @atra/market-service -- tests/services/MarketStreamManager.test.ts tests/services/PriceService.test.ts tests/rest/router.test.ts` | PASS | 3 files, 40 tests: multi-socket membership transitions and fresh/stale/unavailable/mixed `/prices` behavior. |
| `npm run test --workspace @atra/market-service` | **FAIL** | 8 files / 88 tests pass; 1 test fails: `tests/integration/integration.test.ts` expects `[ticker]`, while delivered endpoint returns `[{ ticker, freshness: "fresh" }]`. |
| `npm run build --workspace @atra/database` and `npm run build --workspace @atra/auth-service` | PASS | Both TypeScript builds passed. |
| Focused auth command over nonce/account/wallet/role/recovery/auth/session/middleware tests | PASS | 11 files, 138 tests. |
| `npm run test --workspace @atra/auth-service` | PASS | 18 files / 202 tests pass; 1 file / 4 tests skipped. Warnings: future Vite native-loader `__dirname` compatibility and hoisted nested `vi.mock("ethers")`. |
| `DATABASE_URL=postgres://atra_wf0002@127.0.0.1:55432/atra_wf0002 npm run db:migrate --workspace @atra/database` | PASS | Migration applied successfully to the authorized loopback disposable PostgreSQL environment. |
| Database nonce integration test with that `DATABASE_URL` | PASS | 1 file, 4 tests: competing attempts yield no more than one successful consumption/outcome. |
| Database role/owner invariant integration test with that `DATABASE_URL` | PASS | 1 file, 6 tests: duplicate role, second owner/recovery, owner consistency, and competing-write cases rejected; valid committed paths retained. |
| `DATABASE_URL=... npm run test --workspace @atra/database` | PASS | 8 files, 88 tests, including the above integration evidence. |

## Requirement coverage

| Requirement | Result | Evidence / limitation |
| --- | --- | --- |
| REQ-MKT-001 | PASS (focused) | 40 focused market tests cover retained membership/upstream-interest transitions. Full workspace remains red only because of the unrelated-to-membership stale `/prices` assertion below. No multi-instance claim. |
| REQ-API-002 | PASS behavior; FAIL suite maintenance | Focused price/router tests verify per-result `ticker` plus `freshness`, including unavailable distinction. The full-suite failure confirms one existing integration test has not been migrated to that approved breaking contract. |
| REQ-AUTH-003 | PASS | 138 focused auth tests and full auth suite cover session-derived controller authority, negative unauthenticated cases, and revoke handling. |
| REQ-NONCE-004 | PASS | Four database-backed concurrent nonce tests passed against the authorized disposable PostgreSQL instance. No production claim. |
| REQ-SES-005 | PASS | Focused/full auth suites cover session wallet attribution, refresh/lifecycle behavior in the routed test set. |
| REQ-DATA-006 | PASS | Migration and six live PostgreSQL invariant/competing-write tests passed. Existing-data compatibility is not claimed; the run's approved no-data decision remains its sole preflight basis. |
| REQ-EVD-007 | FAIL pending correction | Most required evidence is present, but the aggregate market suite is not green. |
| REQ-DEL-008 | NOT ASSESSED TO COMPLETION | Product PR #3 is verified OPEN; Review, artifact PR, and final completion are outside this Test report and cannot advance while this verdict fails. |

## Failure classification and required bounded correction

| Item | Classification | Required action |
| --- | --- | --- |
| `market-service/tests/integration/integration.test.ts:144` | **Test defect** | Update the old bare-array assertion to the already approved direct freshness-aware `/prices` response. No production behavior or API decision change is indicated. Rerun the full market workspace suite and regenerate/revise this report. |
| Vite/Vitest warnings in auth tests | Toolchain maintenance warning | No current failure; do not expand this run solely for future-tooling warnings. |

## Limits and risks

- Database results are limited to PostgreSQL 17.11 at `127.0.0.1:55432`, role/database `atra_wf0002`; no production, provider, multi-instance, client, or deployment behavior was tested or claimed.
- The auth full suite intentionally reports four skipped tests; this report does not represent those cases as executed evidence.
- The failed market assertion is evidence that the product's intentional contract replacement has not been propagated to every existing test. It must be corrected before formal Test can pass.

## Requested controller action

This independent Test report is limited to observed code and PR #3. It makes no product edit, approval, merge, or requirement change. **A bounded test-only execution correction is required before the Test stage can be validated PASS and Review may begin.**
