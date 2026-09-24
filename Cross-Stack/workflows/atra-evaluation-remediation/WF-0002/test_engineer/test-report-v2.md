---
artifact:
  id: TEST-0002
  type: TEST_REPORT
  version: 2
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
    - TEST-0002@v1
  references:
    - path: requirements_engineer/requirements-v1.md
    - path: execution_router/routing-plan-v2.md
    - product_repository: Atra-Services
      pull_request: 3
      url: https://github.com/SampleApp05/Atra-Services/pull/3
      base: main
      head: workflow/WF-0002-atra-evaluation-remediation
      commit: f6d40d7e3e39c540fbc8f94f331db2c5baac3d70
ownership:
  created_by: test_engineer
  performed_by: test_engineer
  recorded_by: test_engineer
  engine: codex
  target_id: codex-native
---

# Test Report — Atra evaluation remediation (recheck)

## Outcome

- **Mode:** delivery.
- **Product under test:** Atra-Services PR #3, OPEN, `main` ← `workflow/WF-0002-atra-evaluation-remediation`, head `f6d40d7e3e39c540fbc8f94f331db2c5baac3d70` (verified 2026-09-24).
- **Product changes made by this stage:** none.
- **Verdict:** **PASS_WITH_WARNINGS**. The only v1 failure was the stale bare-array `/prices` test expectation. It is corrected at the verified head; all three workspace suites and three builds now pass. Required disposable-PostgreSQL migration, atomic nonce, and role/owner evidence remain passed from the same delivery branch.

## Executed evidence

| Command / evidence | Result | Observed result |
| --- | --- | --- |
| `npm run build --workspace @atra/market-service`, `@atra/database`, `@atra/auth-service` | PASS | All three TypeScript builds passed at `f6d40d7`. |
| `npm run test --workspace @atra/market-service` | PASS | 9 files, 89 tests. This recheck includes the formerly failing integration assertion and confirms the direct freshness-aware `/prices` body. |
| `DATABASE_URL=postgres://atra_wf0002@127.0.0.1:55432/atra_wf0002 npm run test --workspace @atra/database` | PASS | 8 files, 88 tests. Authorized disposable PostgreSQL only. |
| `npm run test --workspace @atra/auth-service` | PASS | 18 files / 202 tests passed; 1 file / 4 tests skipped. |
| Prior focused market evidence | PASS | 40 tests covering multi-socket membership transitions and fresh/stale/unavailable/mixed `/prices` outcomes. |
| Prior focused auth evidence | PASS | 11 files, 138 tests covering session authority, nonce/session lifecycle, and negative authorization cases. |
| Prior disposable-PostgreSQL migration/evidence | PASS | Migration applied; 4 competing nonce tests and 6 role/owner invariant/competing-write tests passed. |

## Requirement coverage

| Requirement | Result | Evidence / limitation |
| --- | --- | --- |
| REQ-MKT-001 | PASS | Focused 40-test membership evidence plus full market suite 89/89. In-process only; no multi-instance claim. |
| REQ-API-002 | PASS | Focused router/price evidence and corrected full integration suite confirm `ticker` plus per-result `freshness`; unavailable remains distinct. Intentional direct breaking contract. |
| REQ-AUTH-003 | PASS | Focused 138-test evidence plus full auth suite validate session-derived authority and negative cases. |
| REQ-NONCE-004 | PASS | Four live PostgreSQL competing-consumption tests passed; no production claim. |
| REQ-SES-005 | PASS | Focused/full auth evidence covers durable signing-wallet attribution, refresh, self-only revoke, and selected invalidations. |
| REQ-DATA-006 | PASS | Migration plus six disposable-PostgreSQL invariant/competing-write tests passed. Approval-based no-existing-data preflight only; no production compatibility claim. |
| REQ-EVD-007 | PASS | Every functional requirement maps to observed green evidence, including database-backed durability/concurrency results. |
| REQ-DEL-008 | PASS for Test-stage scope | Product PR #3 is verified OPEN. Review, artifact PR, and final report remain controller stages; no merge/deploy/release performed. |

## Warnings and limits

- Auth suite retains four skipped tests. Vitest reports future compatibility warnings for `__dirname` configuration and a hoisted nested `vi.mock("ethers")`; neither caused a test failure and neither is expanded in this approved run.
- Database results are limited to PostgreSQL 17.11 on authorized loopback disposable environment `127.0.0.1:55432`, database/role `atra_wf0002`. No provider, production, client, deployment, or multi-instance behavior is claimed.

## Requested controller action

The stale test defect reported in `TEST-0002@v1` is resolved and independently rechecked. Test-stage evidence is ready for orchestrator validation and independent Review. This report makes no approval, merge, or requirement change.
