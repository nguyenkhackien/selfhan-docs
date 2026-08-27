# Tasks: Frontend Feature Architecture Refactor

**Work Item ID**: `005-frontend-feature-architecture-refactor`
**Input**: `spec.md` and `plan.md`
**Status**: Complete

## Phase 1: Setup

- [x] **T001** `[DONE]` Create and review the specification and implementation plan. Evidence: `spec.md` and `plan.md` record the accepted behavior-preserving scope and architecture decision.
- [x] **T002** `[DONE]` Confirm frontend verification commands. Evidence: `../frontend/package.json` defines `npm run verify` with formatting, lint, typecheck, coverage, build, and E2E checks.

## Phase 2: Tests First

- [x] **T003** `[DONE]` Record the current test baseline and move/update tests only as modules move. Evidence: baseline 18 tests passed; the refactored suite has 24 passing tests and preserves per-file coverage.

## Phase 3: Implementation

- [x] **T004** `[DONE]` Add aligned `@/` aliases and establish app providers, router, layout, shared HTTP transport, shared UI states, and remote-resource lifecycle. Evidence: `npm run typecheck` passed and production import scan found no shared-to-feature or deep feature import.
- [x] **T005** `[DONE]` Split auth into feature-local API, types, hook, page, and public entry point. Evidence: auth hook tests and login/register/logout learner-flow tests passed.
- [x] **T006** `[DONE]` Split the curriculum route pages, learning UI, and writing interaction into a curriculum feature with a public entry point. Evidence: unit, accessibility, and Playwright Level → Unit → Lesson tests passed.
- [x] **T007** `[DONE]` Remove superseded monolith/combined-client modules and align all imports and test locations. Evidence: `SelfHanApp.tsx`, `api/client.ts`, and the old auth hook were removed; lint, typecheck, and import-boundary scan passed.

## Phase 4: Verification

- [x] **T008** `[DONE]` Run focused tests, `npm run verify`, and `git diff --check` in `../frontend`. Evidence: `npm run verify` passed with 24 tests, 94.5% statements, 88.11% branches, 96.59% functions, 97.4% lines, and two Playwright flows.

## Phase 5: Docs And Review

- [x] **T009** `[DONE]` Resolve Canonical Docs state as `Not impacted` with exact checked paths and a behavior-stability rationale. Evidence: actual diff preserves all documented routes, API paths, env variable, auth-token handling, and learner flow.
- [x] **T010** `[DONE]` Write `review.md`. Evidence: primary review approved with no Critical, High, or Medium finding.
- [x] **T011** `[DONE]` Run `DOCS_IMPACT_FINAL_CHECK` from the actual diff and write `converge.md`. Evidence: convergence is complete and canonical docs remain accurate.

## Persona Lens Notes

- **Planner**: T004 precedes feature extraction because common transport and app boundaries are dependencies of both T005 and T006. T005 and T006 remain sequential to avoid concurrent edits to routing and shared test fixtures.

## Traceability

| Task | Requirement | Verification |
| --- | --- | --- |
| T003 | CR-001 | Existing focused tests establish behavior baseline. |
| T004 | CR-002, CR-003 | Typecheck, lint, and boundary review. |
| T005 | CR-001, CR-003 | Auth hook and learner flow tests. |
| T006 | CR-001, CR-003 | Flow, accessibility, and E2E tests. |
| T007 | CR-002, CR-003 | Import search, lint, and typecheck. |
| T008 | CR-004 | `npm run verify` and `git diff --check`. |
| T009 | CR-001 to CR-004 | Actual diff versus canonical docs. |
