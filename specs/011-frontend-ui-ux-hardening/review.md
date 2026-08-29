# Review: Frontend UI/UX Hardening

**Work Item ID**: `011-frontend-ui-ux-hardening`
**Date**: `2026-08-29`
**Reviewer**: Agent (Reviewer/QA lens)

## Scope Reviewed

- Spec: `spec.md`
- Plan: `plan.md`
- Tasks: `tasks.md`
- Code: `../frontend/src/layouts/AppLayout.tsx`,
  `../frontend/src/styles/global.css`, auth, curriculum, HSK, learning, and
  admin source paths listed in `plan.md`
- Tests: changed feature tests, new
  `../frontend/src/features/admin/adminContent.test.tsx`, and
  `../frontend/tests/e2e/selfhan.spec.ts`
- Verification: focused Vitest, coverage, format, lint, typecheck, build,
  Playwright E2E, `git diff --check`, and live browser checks

## Reviewer/QA Lens Notes

- The implementation keeps API boundaries and persisted data unchanged. The
  keyboard writing inputs are explicitly local and non-scoring.
- The admin save source already has one refresh callback; a focused request
  count assertion now guards that behavior against a future duplicate reload.

## Findings

| ID    | Severity | Location                                                    | Finding                                                                                                                                                                    | Recommendation                                                                                   | Disposition                                                |
| ----- | -------- | ----------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ | ---------------------------------------------------------- |
| R-001 | High     | `../frontend/src/styles/global.css`                         | Responsive cascade exposed desktop navigation on phones and left the tablet hero too narrow.                                                                               | Make the final responsive rules authoritative and keep the tablet hero readable.                 | Fixed; live checks and E2E pass at 375/768/1024/1440px.    |
| R-002 | Medium   | `../frontend/src/layouts/AppLayout.tsx`, curriculum pages   | Deep-route anchor and hierarchy back links did not preserve the user journey.                                                                                              | Use a home-section route and validated router state with safe fallbacks.                         | Fixed; route-state tests and desktop E2E pass.             |
| R-003 | High     | `../frontend/src/layouts/AppLayout.tsx`                     | Mobile dialog Escape, focus restoration, background scroll, and skip-link focus were incomplete.                                                                           | Add dialog lifecycle cleanup, focus trap/restoration, scroll locking, and main focus.            | Fixed; axe, component, live keyboard, and E2E checks pass. |
| R-004 | Medium   | `../frontend/src/features/auth/pages/AuthPage.tsx`          | Auth forms had no associated field feedback or password visibility affordance.                                                                                             | Add local validation, field associations, focus management, autocomplete, and visibility toggle. | Fixed; blank submit and auth error tests pass.             |
| R-005 | Medium   | curriculum/HSK writing components                           | Canvas-only practice had no equivalent keyboard path.                                                                                                                      | Add a labelled, local text-entry alternative while retaining pointer drawing.                    | Fixed; component tests and live keyboard checks pass.      |
| R-006 | Medium   | learning progress/review components                         | Expected mutation failures were silent and visual state lacked semantic state.                                                                                             | Catch mutation errors and expose status, busy, pressed, and retryable feedback.                  | Fixed; success/error component tests pass.                 |
| R-007 | Low      | admin/HSK/CSS controls                                      | Resource selection, touch sizing, pressed feedback, HSK sheet balance, and one mode label needed consistency.                                                              | Reuse native button semantics, theme tokens, and stable active states.                           | Fixed; admin/HSK tests, axe, and live theme checks pass.   |
| R-008 | Low      | `../frontend/src/features/admin/pages/AdminContentPage.tsx` | A duplicate admin refresh was reported by the audit but was not reproduced in the current source; the existing callback has one reload path without a request-count guard. | Keep one refresh and add an explicit regression assertion.                                       | Guarded; admin request-count test passes.                  |

## Review Cycle

- **Primary review verdict**: Approved
- **Scoped re-review used**: Yes
- **Scoped re-review boundary**: Original findings R-001–R-008, their direct
  source corrections, and the final regression evidence only.
- **Scoped re-review result**: All in-scope findings closed; no direct
  regressions found.

## Test Gaps

- Authenticated mutation behavior is covered with mocked API/component tests;
  the public Playwright flow does not seed a persistent authenticated account.
  Backend integration and permission contracts remain outside this frontend
  work item.
- Canvas drawing remains manual for visual stroke fidelity; the keyboard
  alternative and pointer event lifecycle are covered automatically.

## Security And Data Risks

None found. No API, permission, credential-storage, or persisted stroke behavior
changed. Password input retains browser password-manager autocomplete metadata.

## Performance Risks

None found. The dialog listener and body-scroll lock are cleaned up, the
keyboard writing state is local, and no new dependency or synchronous data flow
was introduced.

## Cross-Cutting Concerns

| Concern                                  | Reviewed Evidence                                                           | Finding Or None                                          |
| ---------------------------------------- | --------------------------------------------------------------------------- | -------------------------------------------------------- |
| State, cache, and invalidation           | Router state, `useRemoteResource` reload paths, admin request-count test    | None; refresh scope remains feature-local.               |
| Validation, errors, retry, and offline   | Auth field tests; learner/admin success and failure tests; retryable states | None; failed mutations leave the current item available. |
| Security, privacy, auth, and permissions | Auth flow, protected admin render, no-persistence writing copy              | None found.                                              |
| Accessibility                            | Native controls, labels/descriptions, focus lifecycle, `aria-*`, axe checks | None found in covered routes.                            |
| Performance, reliability, and release    | Node 24 verify, build, E2E, cleanup paths, live breakpoints                 | None found.                                              |

## Docs Drift

None found. The work-item spec, plan, and tasks reflect the resulting code and
verification; canonical feature/API documents remain accurate.

## Docs Impact Verification

- **Precheck result reviewed**: Not impacted
- **Canonical docs checked**: `specs/003-frontend-learner-shell/spec.md`,
  `specs/007-hsk-learner-writing/spec.md`,
  `specs/008-mvp-learning-admin/spec.md`, `specs/010-theme-settings/work.md`,
  and `../frontend/README.md`
- **Actual diff impact**: Frontend presentation, navigation state, local form
  feedback, local writing input, mutation feedback, tests, and browser test
  typing. No API, persisted-data, permission, or route contract changed.
- **Project-wide traceability**: Not impacted with reason recorded in
  `spec.md` and `plan.md`.
- **Canonical docs state**: Not impacted
- **Verifier**: Agent / Reviewer-QA
- **Evidence or blocker**: Exact canonical paths above were checked; the fixes
  restore behaviors already required by those documents.

## Verdict

Approved. No Critical, High, or Medium finding remains open.
