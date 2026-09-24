---
artifact:
  id: IMPL-0002
  type: IMPLEMENTATION_RESULT
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
    - ROUTE-0002@v2
    - DEC-0002@v1
    - REQ-0002@v1
  references:
    - path: execution_router/routing-plan-v2.md
    - path: requirements_engineer/requirements-v1.md
ownership:
  created_by: implementation_worker
  performed_by: implementation_worker
  recorded_by: implementation_worker
  engine: codex
  target_id: codex-native
---

# Implementation result — EU-02 direct freshness-aware prices contract

## Scope and outcome

Implemented the approved direct breaking `/prices` response contract for `REQ-API-002`. `PriceService` now returns one result per requested symbol shaped as `{ ticker, freshness }`, where `freshness` is `fresh` or `stale`. A fresh cache entry and a successful cache-miss upstream fetch are `fresh`; a usable stale cache entry returned while its background refresh is pending is `stale`.

The REST handler returns those results directly. An upstream/cache-miss failure still rejects at the service boundary and the existing endpoint error path returns HTTP 502, so it is observably distinct from a successful stale result. No compatibility, versioned, parallel, negotiated, or legacy response was added.

## Changed product files

- `apps/market-service/src/services/PriceService.ts`
- `apps/market-service/src/rest/router.ts`
- `apps/market-service/tests/services/PriceService.test.ts`
- `apps/market-service/tests/rest/router.test.ts`

No product files outside EU-02's allowlist were edited by this unit. The product branch was observed at `workflow/WF-0002-atra-evaluation-remediation`, baseline commit `be51242593fe6643aff11c491d4f8a6c0835aed5`; this worker did not commit, push, create a branch, or open a pull request.

## Verification

Executed at 2026-09-24T11:55:33Z from the verified product workspace:

- `npm run build --workspace @atra/market-service` — PASS.
- `npm run test --workspace @atra/market-service -- tests/services/PriceService.test.ts tests/rest/router.test.ts` — PASS: 2 files, 21 tests.
- `git diff --check` for the product working tree — PASS.

The added/updated test evidence covers fresh cache results, stale cache results with asynchronous refresh, successful cache misses, unavailable upstream cache misses, mixed fresh/stale multi-symbol results, direct REST serialization, and the HTTP 502 unavailable/error response.

## Risks, limitations, and handoff

The `/prices` response body is intentionally breaking as approved because no consumers exist. The focused tests do not make live-provider, client, or multi-instance claims. This artifact is a draft implementation result for coordinator scope audit and independent Test/Review; it does not approve a gate or a pull request.
