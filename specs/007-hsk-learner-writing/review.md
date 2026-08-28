# Review: HSK Learner UI And Writing Practice

**Work Item ID**: 007-hsk-learner-writing
**Date**: 2026-08-28
**Reviewer**: Agent

## Scope Reviewed

- Spec: `spec.md`
- Plan: `plan.md`
- Tasks: `tasks.md`
- Code: HSK data module/API, HSK frontend feature, local stroke asset builder,
  router/layout, and related styles.
- Tests: HSK backend use-case/controller/DTO tests and frontend HSK component
  tests.
- Verification: frontend/backend format, lint, typecheck, tests, build, local
  API smoke test, and desktop/mobile browser smoke test.

## Reviewer/QA Lens Notes

- The test double exposed a synchronous HanziWriter data callback that could
  have used the writer before assignment. The implementation now defers replay
  to a microtask and guards stale callbacks, so real asynchronous behavior and
  a synchronous adapter are both safe.

## Findings

| ID    | Severity | Location                                                       | Finding                                                                       | Recommendation                                                          | Disposition |
| ----- | -------- | -------------------------------------------------------------- | ----------------------------------------------------------------------------- | ----------------------------------------------------------------------- | ----------- |
| R-001 | Medium   | `frontend/src/features/hsk/components/WordWritingPractice.tsx` | A synchronous character-data callback could animate before writer assignment. | Defer the replay until assignment completes and ignore stale callbacks. | Fixed       |

## Review Cycle

- **Primary review verdict**: Approved.
- **Scoped re-review used**: Yes.
- **Scoped re-review boundary**: R-001, frontend writing component and focused
  test evidence.
- **Scoped re-review result**: All in scope closed; frontend format, lint,
  tests, and production build pass after the correction.

## Test Gaps

- Pointer stroke pixels are manually inspected only; handwriting recognition,
  scoring, and persistence are intentionally out of scope.

## Security And Data Risks

- None found. The HSK routes are read-only and public by design; stroke data
  uses the application origin with no remote fallback.

## Performance Risks

- The selected local stroke assets add static weight. The build includes only
  characters in the imported HSK snapshot and loads individual character JSON
  on demand.

## Cross-Cutting Concerns

| Concern                                  | Reviewed Evidence                                                                                     | Finding Or None |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------- | --------------- |
| State, cache, and invalidation           | Cursor history resets on band/query change; stale writer callbacks are ignored.                       | None.           |
| Validation, errors, retry, and offline   | DTO validation, API error UI, local loader test, and unavailable-data state.                          | None.           |
| Security, privacy, auth, and permissions | Public decorator, no writes, no learner-state storage, no remote loader.                              | None.           |
| Accessibility                            | Labelled search/canvas, status feedback, pressed state, visible focus styles, and 44px mode controls. | None.           |
| Performance, reliability, and release    | Full builds pass; 375px check has no horizontal overflow.                                             | None.           |

## Docs Drift

- None found after the API contract, data model, traceability, and application
  README updates.

## Docs Impact Verification

- **Precheck result reviewed**: Impacted.
- **Canonical docs checked**: `specs/001-chinese-learning-mvp/contracts/api-v1.md`,
  `specs/001-chinese-learning-mvp/data-model.md`,
  `specs/001-chinese-learning-mvp/tasks.md`, `spec.md`, `plan.md`, `tasks.md`,
  backend `README.md`, frontend `README.md`, and the local data notices.
- **Actual diff impact**: Public HSK read contract, learner browse/detail flow,
  local writing interaction, offline behavior, and operational guidance.
- **Project-wide traceability**: Synced in
  `specs/001-chinese-learning-mvp/tasks.md`.
- **Canonical docs state**: Synced.
- **Verifier**: Agent.
- **Evidence or blocker**: The listed canonical documents describe the final
  public API, local assets, attribution, and verified learner behavior.

## Verdict

Approved. No Critical, High, or unresolved Medium finding remains.
