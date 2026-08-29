# Implementation Plan: Frontend UI/UX Hardening

**Work Item ID**: `011-frontend-ui-ux-hardening`
**Date**: `2026-08-29`
**Spec**: `spec.md`

## Summary

Apply the audited frontend fixes at their confirmed sources while preserving
the existing React feature boundaries and semantic theme system. The work is
ordered from shared navigation/responsive behavior to forms, writing, learner
mutations, semantic state, and verification. No backend or persisted-data
change is required.

## Technical Context

- **Language/Version**: TypeScript, React 19, Node 24
- **Primary Dependencies**: React Router, Vitest, Testing Library, Playwright,
  axe-core, HanziWriter
- **Storage**: Browser theme preference only; no new storage
- **Testing**: Vitest + Testing Library, Playwright + axe-core
- **Target Platform**: Responsive web, phone/tablet/desktop widths
- **Project Type**: Vietnamese-first learner application with sidebar layout
- **Performance Goals**: Preserve current local stroke-data loading and avoid
  new runtime dependencies or synchronous layout work
- **Constraints**: Keep existing API contracts, theme tokens, deep links, and
  no persisted handwriting strokes
- **Scale/Scope**: Frontend cross-cutting bugfix across navigation, CSS,
  authentication, curriculum, HSK, learning, admin, tests, and TypeScript

## Constitution Check

- [x] Specification first
- [x] Clarification gate completed
- [x] Material persona lens decisions recorded where they affect the plan
- [x] Requirements are traceable
- [x] Simplicity preserved
- [x] Testing strategy defined
- [x] Existing patterns followed
- [x] Review and convergence planned

## Persona Lens Notes

- **Planner**: Tests and shared navigation behavior precede feature-specific
  fixes; final live verification runs only after all dependent source changes.
- **Engineer**: Reuse native inputs, router link state, `aria-pressed`, and
  existing status/error patterns. Keep the pointer canvas for pointer users and
  add a nearby text-entry alternative rather than inventing a drawing protocol.

## Architecture And Data Flow

AppLayout remains the owner of the persistent header, mobile drawer, skip link,
and footer. Router link state carries only a local `from` path for hierarchy
back-links; route URLs and API contracts remain unchanged. AuthPage validates
before invoking the existing auth hook and maps known API copy at the UI edge.
Writing components retain local canvas state and add non-persistent keyboard
input. Learning mutation components catch failures locally and expose status
without changing server behavior. CSS fixes are appended or narrowly scoped to
the existing semantic token layer so current theme choices continue to apply.

## Project Structure

### Documentation

```text
specs/011-frontend-ui-ux-hardening/
  spec.md
  plan.md
  tasks.md
  review.md
  converge.md
```

### Source Code

```text
../frontend/src/layouts/AppLayout.tsx
../frontend/src/styles/global.css
../frontend/src/features/auth/pages/AuthPage.tsx
../frontend/src/features/curriculum/pages/LevelPage.tsx
../frontend/src/features/curriculum/pages/UnitPage.tsx
../frontend/src/features/curriculum/pages/LessonPage.tsx
../frontend/src/features/curriculum/components/WritingCanvas.tsx
../frontend/src/features/hsk/components/WordWritingPractice.tsx
../frontend/src/features/learning/components/LessonProgressPanel.tsx
../frontend/src/features/learning/pages/ReviewPage.tsx
../frontend/src/features/admin/pages/AdminContentPage.tsx
../frontend/tsconfig.node.json
../frontend/tests/e2e/selfhan.spec.ts
```

## File-Level Impact

| File                                                                   | Action | Responsibility                                                                     |
| ---------------------------------------------------------------------- | ------ | ---------------------------------------------------------------------------------- |
| `../frontend/src/layouts/AppLayout.tsx`                                | Modify | Mobile drawer lifecycle, skip-link focus, anchor destination, menu state semantics |
| `../frontend/src/styles/global.css`                                    | Modify | Mobile cascade, tablet hero, sheet composition, pressed state, touch sizing        |
| `../frontend/src/features/auth/pages/AuthPage.tsx`                     | Modify | Inline validation, localized feedback, password visibility                         |
| `../frontend/src/features/curriculum/pages/LevelPage.tsx`              | Modify | Pass Level back context to Unit links                                              |
| `../frontend/src/features/curriculum/pages/UnitPage.tsx`               | Modify | Preserve Level context and pass Unit context to Lesson links                       |
| `../frontend/src/features/curriculum/pages/LessonPage.tsx`             | Modify | Use contextual Unit back path                                                      |
| `../frontend/src/features/curriculum/components/WritingCanvas.tsx`     | Modify | Accessible keyboard alternative and status                                         |
| `../frontend/src/features/hsk/components/WordWritingPractice.tsx`      | Modify | Accessible keyboard alternative and localized mode copy                            |
| `../frontend/src/features/learning/components/LessonProgressPanel.tsx` | Modify | Semantic toggle state and save error feedback                                      |
| `../frontend/src/features/learning/pages/ReviewPage.tsx`               | Modify | Review save error/status feedback                                                  |
| `../frontend/src/features/admin/pages/AdminContentPage.tsx`            | Modify | Resource group and selected state semantics                                        |
| `../frontend/tsconfig.node.json`                                       | Modify | DOM types for browser test code                                                    |
| `../frontend/tests/e2e/selfhan.spec.ts`                                | Modify | Regression coverage for responsive/accessibility findings                          |
| Existing `*.test.tsx` files                                            | Modify | Focused component and flow regression coverage                                     |

## Testing Approach

- Add failing-before unit checks for form errors, router state, mutation
  failures, writing alternatives, and selected/pressed semantics where
  Testability permits.
- Add Playwright assertions for desktop-nav visibility at 375px, hero layout
  at 768px, menu Escape/focus/scroll lifecycle, home anchor routing,
  skip-link focus, and hierarchy back links.
- Run `npm run format:check`, `npm run lint`, `npm run typecheck`,
  `npm run test:coverage`, `npm run build`, and `npm run test:e2e:run` with the
  Node 24 runtime.
- Manually inspect reduced motion, 375px, 768px, 1024px, 1440px, and each
  theme after automated checks.

## Cross-Cutting Concerns

| Concern                                  | Decision Or N/A                                                                                            | Verification                                                    |
| ---------------------------------------- | ---------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| State, cache, and invalidation           | Keep existing remote-resource keys; local route state is only a back-path hint.                            | Curriculum flow tests and browser navigation checks             |
| Validation, errors, retry, and offline   | Validate auth locally; catch learner mutation errors; keep existing retryable loading/error components.    | Vitest failure-path tests and live blank-form check             |
| Security, privacy, auth, and permissions | No API or permission changes; password input remains autocomplete-compatible and no secrets are logged.    | Auth tests, source review, and existing protected-route checks  |
| Accessibility                            | Add focus lifecycle, field associations, pressed states, keyboard writing alternative, and visible status. | axe, Testing Library roles/properties, Playwright keyboard pass |
| Performance and reliability              | No new dependency or persistent drawing state; avoid layout shift in pressed styles.                       | Build, live interaction pass, and source review                 |
| Compatibility, migration, and release    | No migration; fix Node test project DOM typing and make configured verification runnable.                  | Full `npm run verify`                                           |

## Docs Impact Plan

- **Precheck result**: `Not impacted`
- **Canonical docs checked**: `specs/003-frontend-learner-shell/spec.md`,
  `specs/007-hsk-learner-writing/spec.md`,
  `specs/008-mvp-learning-admin/spec.md`,
  `specs/010-theme-settings/work.md`, and `../frontend/README.md`
- **Canonical docs to sync**: None; fixes restore documented behavior and this
  work item records the new regression evidence.
- **Project-wide traceability**: Existing specs remain accurate; no requirement
  or API mapping changes.
- **Sync task**: T016
- **Required verifier/owner**: Agent and Reviewer-QA
- **Completion evidence**: `review.md` and `converge.md` record exact paths and
  the no-impact rationale.

## Risks And Mitigations

| Risk                                                              | Impact                                               | Mitigation                                                                                                                            |
| ----------------------------------------------------------------- | ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| Focus restoration conflicts with route changes from the drawer    | Keyboard focus may return to an unhelpful element.   | Capture the trigger before open, focus the first close control, and restore on unmount; verify both close paths and route navigation. |
| Router state is absent on a direct deep link                      | Back link cannot infer a parent.                     | Use a stable `/levels` fallback and test direct route loads.                                                                          |
| Adding a keyboard alternative could imply handwriting recognition | Learner expectations or data boundary could change.  | Use explicit “nhập chữ thay thế” copy and keep it local/non-scoring/non-persistent.                                                   |
| Coverage threshold exposes unrelated untested modules             | Verification may remain red after behavior is fixed. | Add focused tests for touched and currently uncovered feature paths; do not weaken thresholds.                                        |
| CSS fixes regress one of the three themes                         | Visual hierarchy may diverge by theme.               | Use existing semantic tokens and run theme/contrast checks after the layout changes.                                                  |

## Supporting Docs

- `research.md`: Not required; existing audit evidence and local UI/UX
  guidance are sufficient.
- `data-model.md`: Not required; no persisted data changes.
- `contracts/`: Not required; no API contract changes.

## Complexity Exceptions

None.
