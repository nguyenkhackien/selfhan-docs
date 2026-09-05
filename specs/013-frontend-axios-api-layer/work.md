# Work Item: Frontend Axios API Layer

**Work Item ID**: `013-frontend-axios-api-layer`
**Work Type**: `change-request`
**Status**: Complete

## Scope

- **Outcome**: Replace the frontend's shared `fetch` transport with a typed Axios API layer. It must retain the existing base-URL configuration, HttpOnly-cookie credential behavior, in-memory Bearer access token, feature-owned endpoint adapters, and normalized `ApiError` contract.
- **Non-goals**: Change backend endpoints, DTOs, authentication/refresh flow, retry policy, caching, server-state management, or user-visible error copy.
- **Assumptions and risks**: Axios adds a runtime dependency and a central timeout. A request interceptor will attach the memory-only token, while a response interceptor will convert every Axios failure into the existing safe `ApiError`. Automatic retry and refresh interception are excluded because they could duplicate mutations or alter the current explicit refresh lifecycle.

## Technical Plan And Tasks

| ID | Task | Verification | Status |
| -- | ---- | ------------ | ------ |
| T1 | Add Axios and replace the shared HTTP client with a configured instance, token interceptor, typed request wrapper, and safe error normalization. | Focused HTTP-client tests covering credentials, JSON bodies, token injection, no-content, response errors, network errors, and timeout configuration. | DONE |
| T2 | Update feature endpoint adapters to use Axios request configuration and query parameters without changing paths, payloads, or response types. | Typecheck and feature/API unit tests. | DONE |
| T3 | Run the frontend quality gates, perform a scoped technical review, and complete convergence. | Format, lint, typecheck, tests, build, diff check, and fetch-removal scan. | DONE |

## Test Migration Diagnosis

- **Observed failure**: The first full unit run after the Axios migration failed 24 of 50 tests. Each failed request reported the network fallback instead of the response configured by the test.
- **Cause and localization**: Existing component tests stub the global `fetch`, while the Axios client selects its own browser adapter. The shared HTTP client was the only affected boundary; feature logic and endpoint adapters were unchanged.
- **Minimal experiment**: The focused `httpClient` tests passed when an Axios adapter was supplied directly, proving the Axios instance, interceptors, and response normalization work independently of browser transport.
- **ROOT_CAUSE_GATE**: Confirmed. A test-only Axios adapter can translate the existing mocked `fetch` responses into Axios responses without changing production transport or feature tests.
- **Fix attempt**: 1. Install a test-only adapter in the Vitest setup file. Its first run retained Axios's explicit `GET` method, which differed from the pre-migration tests' omitted-method convention; the adapter now omits `GET` to mirror the previous test boundary before rerunning the unit suite.

## Docs Impact

- **Precheck**: Not impacted.
- **Canonical docs checked or to update**: `specs/001-chinese-learning-mvp/contracts/api-v1.md`, `specs/008-mvp-learning-admin/contracts/learning-admin-v1.md`, `specs/005-frontend-feature-architecture-refactor/spec.md`, and `../frontend/README.md`. They prescribe endpoint, credential, token, and feature-boundary behavior but do not require the `fetch` implementation. The README's API-base configuration remains accurate.
- **Final check**: Not impacted. The actual diff changes only the frontend transport implementation, dependency lockfile, test adapter, and regression tests. All endpoint paths, payload shapes, credential behavior, token lifecycle, and safe-error envelope handling remain aligned with the checked canonical docs.

## Completion Evidence

- **Verification**: `npm run verify` passed in `../frontend`: formatting, lint, strict typecheck, 51 Vitest tests with coverage thresholds, production build, and 5 Playwright flows. `git diff --check` passed. A production-source scan found no `fetch` call in `src/shared/api` or feature API adapters; the remaining production `fetch` only loads local HSK stroke assets and is outside the API layer. `npm ls axios --depth=0` resolved `axios@1.20.0`.
- **Technical review**: Approved; no Critical, High, Medium, or Low findings. The review confirmed one configured Axios instance, memory-only Bearer injection, cookie credentials, typed feature adapters, 204 handling, and central safe-error normalization. The fetch bridge is limited to Vitest setup and is not included in the production API client.
- **Scoped re-review**: Not used.
- **Convergence**: Converged; the Axios migration preserves the documented backend/API and auth contracts, has focused regression coverage, and passes the full frontend quality gate.
