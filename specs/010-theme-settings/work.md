# Work Item: Theme Settings

**Work Item ID**: `010-theme-settings`
**Work Type**: `feature`
**Status**: Complete

## Scope

- **Outcome**: Keep the current sage-green visual style as the default and add
  indigo and terracotta themes. Provide a `/settings` page and navigation entry
  where learners can select a theme; apply the choice across the app and retain
  it in browser storage.
- **Non-goals**: Account-specific server persistence, dark mode, changes to
  learning data or API contracts, and altering existing learner flows.
- **Assumptions and risks**: Theme preference is a presentation-only browser
  preference, so `localStorage` is sufficient. Existing CSS already centralizes
  most colors into semantic tokens, but the handwriting canvases need to read
  the active tokens rather than retain green-only values.

## Technical Plan And Tasks

| ID | Task | Verification | Status |
| --- | --- | --- | --- |
| T1 | Add a typed shared theme provider with safe local preference loading and token lookup. | Unit tests for default, storage recovery, and selection. | DONE |
| T2 | Add the Settings route, accessible theme selection controls, and navigation links. | Route and keyboard-accessible UI tests. | DONE |
| T3 | Define indigo and terracotta token sets; bind styles and handwriting canvases to active semantic colors. | CSS/theme-focused browser check. | DONE |
| T4 | Update the route list and run final verification, review, and convergence checks. | README, focused tests, lint, typecheck, build, and E2E where available. | DONE |

## Docs Impact

- **Precheck**: Impacted.
- **Canonical docs checked or to update**: `../frontend/README.md` documents
  the implemented routes and must include `/settings`. `specs/003-frontend-learner-shell/spec.md`
  and `specs/001-chinese-learning-mvp/spec.md` describe learner flows and
  accessibility; they do not prescribe a theme selector and are expected to
  remain accurate.
- **Final check**: Synced. `../frontend/README.md` now lists `/settings` and
  describes its local browser persistence. `specs/003-frontend-learner-shell/spec.md`,
  `specs/001-chinese-learning-mvp/spec.md`, and
  `specs/001-chinese-learning-mvp/tasks.md` were checked: they retain accurate
  learner-flow, responsive, contrast, and accessibility requirements; none
  promises a single exclusive color theme.

## Completion Evidence

- **Verification**: 20 focused unit tests passed; lint, typecheck, production
  build, formatting, and `git diff --check` passed. Playwright passed all three
  flows, including selecting and retaining Terracotta after reload, axe-checking
  the Settings page, and checking it at 375, 768, 1024, and 1440 px without
  horizontal scroll. Sage, Indigo, and Terracotta primary buttons have 4.97:1,
  5.41:1, and 4.60:1 contrast against white. `npm run test:coverage` remains
  blocked by existing unrelated per-file coverage gaps in Admin, learning, and
  HSK modules; the new shared theme module itself reports 89.47% statements and
  100% lines.
- **Technical review**: Approved; no Critical, High, or Medium findings. A
  coverage run exposed standalone canvas tests missing the new provider; root
  cause was confirmed by the context error, and the tests now render through
  `ThemeProvider` and pass.
- **Scoped re-review**: Not used.
- **Convergence**: Converged; new UI behavior is covered by unit and browser
  tests, existing learner routes remain green by default, and no API, data, or
  authentication contract changed.
