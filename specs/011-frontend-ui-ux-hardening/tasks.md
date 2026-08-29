# Tasks: Frontend UI/UX Hardening

**Work Item ID**: `011-frontend-ui-ux-hardening`
**Input**: `spec.md` and `plan.md`
**Status**: Complete

## Task Rules

- Tasks trace to the accepted bugfix behaviors in `spec.md`.
- Tasks are ordered by dependency and verification speed.
- `[P]` marks work that can proceed independently after shared test setup.
- Exactly one task is `[IN_PROGRESS]` at a time.
- No task introduces a new API, data contract, or unrelated redesign.

## Phase 1: Setup

- [x] **T001** `[DONE]` Confirm the work-item folder contains `spec.md`,
      `plan.md`, and this `tasks.md`; no clarification artifact is needed because
      the user-approved audit scope is unambiguous.
- [x] **T002** `[DONE]` Confirm Node 24, frontend scripts, existing tests,
      E2E config, and the current clean frontend worktree.

## Phase 2: Tests First

- [x] **T003** `[DONE]` Add focused regression tests for BF-002 through
      BF-006: navigation state/anchor, auth validation, keyboard writing input,
      mutation error feedback, and semantic selected/pressed state. Focused and
      feature suites pass; the final coverage run passes 50 tests.
- [x] **T004** `[DONE]` Add Playwright regression coverage for BF-001 through
      BF-003: mobile desktop-nav visibility, the 768px hero layout, drawer
      Escape/focus/scroll lifecycle, home anchor routing, skip-link focus, and
      hierarchy back paths. The final E2E suite passes all 5 scenarios.
- [x] **T005** `[DONE]` Run the focused tests and record failing-before evidence
      for each practical regression; retain manual evidence for visual-only checks.
      The baseline exposed 10 practical regression failures and the browser test
      TypeScript project lacked the DOM library.

## Phase 3: Implementation

- [x] **T006** `[DONE]` Fix AppLayout navigation, mobile drawer focus/scroll
      lifecycle, skip-link focus, and route-independent “Cách học” destination for
      BF-002 and BF-003.
- [x] **T007** `[DONE]` Fix the responsive CSS cascade, tablet hero breakpoint,
      HSK sheet composition, touch sizing, and pressed states for BF-001 and BF-007.
- [x] **T008** `[DONE]` Preserve Level → Unit → Lesson back context through
      router state with direct-link fallbacks for BF-002.
- [x] **T009** `[DONE]` Implement auth field validation, localized known API
      errors, focus management, autocomplete-compatible names, and password
      visibility for BF-004.
- [x] **T010** `[DONE]` Add non-persistent keyboard writing alternatives to the
      curriculum and HSK canvases and localize the HSK mode label for BF-005 and
      BF-007.
- [x] **T011** `[DONE]` Implement learner progress/review error feedback and semantic
      states for progress and admin resource controls for BF-006.
- [x] **T012** `[DONE]` Repair the browser test TypeScript DOM configuration and
      expand coverage fixtures/tests without weakening configured thresholds for
      BF-007.

## Phase 4: Verification

- [x] **T013** `[DONE]` Run focused Vitest and Playwright checks after each
      implementation slice; update task evidence with passing results.
- [x] **T014** `[DONE]` Run format, lint, typecheck, coverage, build, and the full
      E2E suite with the Node 24 runtime.
- [x] **T015** `[DONE]` Perform final live visual/accessibility checks at 375px,
      768px, 1024px, and 1440px for Sage, Indigo, and Terracotta, including reduced
      motion where available.

## Phase 5: Docs And Review

- [x] **T016** `[DONE]` Resolve Canonical Docs state as `Not impacted` with the
      exact checked paths and reason; no existing docs should become stale.
- [x] **T017** `[DONE]` Write `review.md` with the primary findings verdict,
      regression evidence, and any remaining test gaps.
- [x] **T018** `[DONE]` Run `DOCS_IMPACT_FINAL_CHECK` from the actual diff and
      write `converge.md`.

## Persona Lens Notes

- **Planner**: Keep shared navigation and regression fixtures ahead of feature
  fixes; run the final E2E pass only after typecheck/build are green.
- **Engineer**: Use existing native controls and status patterns, and keep
  keyboard writing input explicitly local and non-persistent.

## Traceability

| Task      | Requirement      | Verification                                                   |
| --------- | ---------------- | -------------------------------------------------------------- |
| T003–T005 | BF-002–BF-006    | Vitest focused failures and Playwright assertions              |
| T006–T008 | BF-001–BF-003    | AppLayout/curriculum tests and live keyboard/navigation checks |
| T009      | BF-004           | Auth flow tests and blank/invalid form check                   |
| T010      | BF-005, BF-007   | Writing component tests and keyboard live pass                 |
| T011–T012 | BF-006, BF-007   | Mutation failure tests, typecheck, coverage, build             |
| T013–T015 | All              | Focused, full, and live verification                           |
| T016–T018 | Docs/convergence | Docs review, `review.md`, and `converge.md`                    |
