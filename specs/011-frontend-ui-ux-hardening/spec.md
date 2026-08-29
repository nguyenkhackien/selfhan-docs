# Bug Fix Specification: Frontend UI/UX Hardening

**Work Item ID**: `011-frontend-ui-ux-hardening`
**Work Type**: `bugfix`
**Created**: `2026-08-29`
**Status**: Approved
**Input**: Prior live UI/UX audit and user request to fix all reported findings.

## Clarification Gate

No blocking clarification is required. The prior audit contains repeatable
reproductions, source-level root causes, and the user has approved fixing the
complete finding set. The accepted implementation stays within the frontend;
it does not change backend contracts, persisted data, or permissions.

## Problem And Evidence

- **Observed behavior**: The SelfHan frontend has two major responsive failures,
  broken cross-route navigation, incomplete mobile dialog and skip-link focus
  behavior, missing field-level form feedback, pointer-only writing surfaces,
  silent learner save failures, an unguarded admin refresh path, incomplete
  state semantics, and smaller visual consistency issues.
- **Expected behavior**: The existing learner-shell and feature specifications
  require responsive Level → Unit → Lesson flows, keyboard-operable and
  labelled controls, accessible writing controls, explicit loading/error
  states, and predictable navigation. Fixes must preserve the existing visual
  language, theme choices, API boundaries, and deep-link routes.
- **Evidence**: Live checks at 375px, 768px, and 1440px; source inspection of
  CSS and React components; blank login submission; mobile dialog keyboard
  checks; dead-anchor check; contrast sampling across Sage, Indigo, and
  Terracotta; and the existing verification output.
- **Affected versions/environments**: Current frontend `main` at commit
  `3c1cbbd`, React/Vite, Node 24, desktop browser viewport emulating phone,
  tablet, and desktop widths.

## Reproduction

1. Open the frontend and set the viewport to 375px. The desktop navigation
   appears beside and below the mobile header.
2. Set the viewport to 768px and open `/`. The fixed sidebar leaves a narrow
   main column while the hero remains two-column, causing near word-per-line
   wrapping.
3. Open `/levels` and activate “Cách học”. The URL becomes
   `/levels#how-it-works`, but no target exists.
4. Open the mobile menu, press Escape, or close it with X. Escape leaves the
   dialog open and closing loses focus instead of restoring it to the trigger.
5. Activate “Bỏ qua điều hướng”. Focus does not move to the main content.
6. Follow Level → Unit → Lesson. The Unit and Lesson back links return to the
   catalogue root rather than the current parent context.
7. Submit the login form empty. The request reaches the backend and displays a
   global English error instead of field-local Vietnamese guidance.
8. Navigate to lesson or HSK writing practice with keyboard only. The canvas
   cannot be operated or replaced by an equivalent keyboard interaction.
9. Save an admin content record. Verify that the content list performs one
   refresh for the save rather than issuing duplicate list requests.

**Reproduction status**: Runtime findings were reproduced; source inspection
confirmed the admin save callback has one refresh path, so that behavior is
retained as a request-count regression guard rather than reported as a current
duplicate-request defect.

## Diagnosis

- **Failing boundary or component**: The responsive header and home hero are
  affected by later CSS overrides. AppLayout owns the navigation and overlay
  lifecycle. Curriculum pages use static back destinations. AuthPage disables
  browser validation without a replacement field model. Writing canvases only
  register pointer events. Learning save handlers do not catch expected API
  failures. The admin save callback has one refresh path, but it lacked an
  explicit request-count regression check. Progress and admin controls expose
  visual state without semantic state. The
  HSK sheet has an intentionally narrow maximum width inside a full-width
  panel.
- **Recent changes and environment checked**: Current frontend history, CSS
  cascade, route declarations, component tests, E2E config, and TypeScript
  project references were checked. No unrelated working-tree changes exist.
- **Working reference and material differences**: HSK band, character, and
  mode controls already use `aria-pressed`; quiz radios use native fieldset and
  radio semantics; AsyncState already provides status and retry patterns. The
  fixes reuse these existing patterns.
- **Data-flow or dependency evidence**: No API or data contract needs to
  change. Navigation context can travel through React Router link state with a
  safe catalogue fallback. Form validation is local before auth API calls.
  Learner and review error messages remain local to their feature boundaries.
- **Hypothesis and minimal experiment**: The mobile nav leak is caused by the
  later `.desktop-nav { display: grid; }` rule winning over the earlier mobile
  hide rule; removing that single cascade conflict reproduces the expected
  header. The 768px break is caused by the 760px one-column breakpoint being
  evaluated against the whole viewport rather than the remaining main-column
  width; a one-column tablet experiment restores readable measure. The other
  findings were confirmed by their component source and direct interactions.
- **ROOT_CAUSE_GATE**: `Confirmed` for runtime findings; the admin refresh
  behavior is source-confirmed and guarded by a focused request-count test.
- **Fix attempts**: `0` before implementation.

## Root Cause

- **Cause**: CSS ordering and an overly narrow breakpoint create responsive
  failures; route links lack contextual state; AppLayout lacks dialog keyboard
  lifecycle and skip-link focus management; forms lack field error state;
  canvas interactions lack an accessible alternative; async learner mutations
  lack failure handling; and several controls expose only visual state.
- **Gate evidence**: Each cause is linked to a source location and a live
  reproduction. The existing working controls provide local implementation
  patterns for the fixes.
- **Why existing checks missed it**: E2E checks only mobile width 390px and
  checks navigation reachability without asserting that desktop navigation is
  hidden. It does not cover 768px visual wrapping, Escape/focus restoration,
  skip-link focus, field-local validation, authenticated mutation failures, or
  canvas keyboard alternatives. TypeScript currently prevents the E2E suite
  from starting, and admin save coverage does not assert the number of list
  refresh requests.
- **Related-risk search**: All primary navigation links, curriculum hierarchy
  links, auth inputs, canvas components, progress/review mutations, tab-like
  controls, and final responsive CSS blocks were inspected.

## Accepted Fix Behavior

1. At widths up to 760px, only the mobile header and menu are visible; the
   desktop navigation is removed from layout and the menu trigger exposes its
   expanded state.
2. Between 761px and the desktop layout, the home hero uses a readable
   single-column arrangement whenever the sidebar leaves insufficient content
   width. At 375px, 768px, 1024px, and 1440px, core content remains readable
   without horizontal scrolling.
3. “Cách học” always navigates to the home page section, including from deep
   routes.
4. The mobile menu closes on Escape, traps keyboard focus while open, restores
   focus to its trigger on close, and prevents background scrolling.
5. Activating the skip link moves focus to the main content region.
6. Level → Unit → Lesson links preserve parent context through navigation state;
   direct deep links retain a safe catalogue fallback.
7. Login and registration validate email and password locally, show associated
   field errors, focus the first invalid field, keep password-manager
   autocomplete, provide a show/hide password control, and translate known
   backend validation messages into Vietnamese UI copy.
8. Both writing surfaces provide a keyboard-accessible text-entry alternative
   for the target character while retaining pointer/touch drawing. No strokes
   are persisted, scored, or sent to the backend.
9. Progress, review, and tag mutations expose saving/error/success feedback;
   failed mutations leave the current study item available for retry.
10. Toggle and resource-selection controls expose their current state to
    assistive technology, while existing HSK controls retain their semantics.
11. The HSK writing sheet is visually balanced within its panel and all visible
    mode labels are Vietnamese.
12. Interactive controls show a stable pressed state without layout shift.
13. An admin content save triggers one resource refresh and keeps the current
    resource selected.
14. The TypeScript browser test project includes the DOM library and the full
    frontend verification command can complete its configured gates.

## Regression Coverage

- **Failing-before evidence**: Existing live reproductions and focused
  component checks establish the baseline. New tests will first assert the
  expected accessible and responsive behavior against the current code where
  practical; manual-only visual checks will retain exact viewport and metric
  steps.
- **Passing-after evidence**: Focused Vitest tests for navigation, auth
  validation, progress/review failure feedback, canvas alternatives, and
  semantic states; Playwright checks for responsive header/hero, menu
  lifecycle, skip-link focus, cross-route anchor/back behavior, all themes,
  and axe analysis.
- **Broader checks**: format, lint, typecheck, unit tests, coverage, build,
  Playwright E2E, and a final live pass at 375px, 768px, 1024px, and 1440px.

## Docs Impact

- **Precheck result**: `Not impacted` for existing product/API docs.
- **Canonical docs checked**: `specs/003-frontend-learner-shell/spec.md`,
  `specs/007-hsk-learner-writing/spec.md`,
  `specs/008-mvp-learning-admin/spec.md`,
  `specs/010-theme-settings/work.md`, and `../frontend/README.md`.
- **Canonical docs to update**: None beyond this bugfix work item.
- **Reason**: The accepted fixes restore behavior already required by the
  existing responsive, accessibility, navigation, error-state, and writing
  requirements. No API, persisted-data, permission, or route contract changes
  are introduced.
- **Required verifier/owner**: Agent and Reviewer-QA.

## Out Of Scope

- Backend endpoints, DTOs, database schema, authentication protocol, or
  permission rules.
- Handwriting recognition, scoring, stroke persistence, or external stroke
  data.
- New visual branding, dark mode, or unrelated feature redesign.
- Rewriting the existing test architecture solely to raise coverage without
  covering the changed behavior.

## Traceability

| Accepted Behavior                                   | Implementation Area                                      | Regression Verification                                   | Canonical Doc                        |
| --------------------------------------------------- | -------------------------------------------------------- | --------------------------------------------------------- | ------------------------------------ |
| BF-001 responsive header and tablet hero            | `frontend/src/styles/global.css`                         | Playwright responsive metrics and screenshots             | `003-frontend-learner-shell/spec.md` |
| BF-002 navigation anchors and hierarchy back-path   | `frontend/src/layouts/AppLayout.tsx`, curriculum pages   | Vitest route-state checks and Playwright deep-link checks | `003-frontend-learner-shell/spec.md` |
| BF-003 dialog and skip-link focus                   | `frontend/src/layouts/AppLayout.tsx`                     | Playwright keyboard checks and axe                        | `003-frontend-learner-shell/spec.md` |
| BF-004 auth validation and feedback                 | `frontend/src/features/auth/pages/AuthPage.tsx`          | Vitest invalid-submit and backend-error tests             | `003-frontend-learner-shell/spec.md` |
| BF-005 keyboard writing alternative                 | curriculum/HSK writing components                        | Vitest accessible input checks and live keyboard pass     | `007-hsk-learner-writing/spec.md`    |
| BF-006 learner mutation feedback and semantic state | learning/admin components                                | Vitest failure/state tests and axe                        | `008-mvp-learning-admin/spec.md`     |
| BF-007 visual polish and complete verification      | CSS, config, tests                                       | full `npm run verify` and live breakpoint pass            | `../frontend/README.md`              |
| BF-008 single admin resource refresh                | `frontend/src/features/admin/pages/AdminContentPage.tsx` | Vitest save callback/request-count check                  | `008-mvp-learning-admin/spec.md`     |
