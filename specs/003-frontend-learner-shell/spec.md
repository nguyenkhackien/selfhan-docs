# Feature Specification: Frontend Learner Shell

**Work Item ID**: `003-frontend-learner-shell`
**Work Type**: `feature`
**Status**: Complete
**Parent**: `001-chinese-learning-mvp`

## Outcome

Replace the existing unrelated browser-only ZenLingo experience with an API-backed
SelfHan learner shell. Visitors can register or sign in, browse published
curriculum from Level to Unit to Lesson, study vocabulary and grammar, play
available audio, and practice writing a lesson character in a browser-only
canvas. The experience works at mobile and desktop widths with accessible
loading, error, empty, and keyboard states.

## Contract Boundary

The implementation consumes only the reviewed foundation endpoints currently
available from the backend: authentication and public level, unit, and lesson
reads defined in `001-chinese-learning-mvp/contracts/api-v1.md`.

Quiz submission, persisted lesson progress, dashboard/streak values, SRS,
flashcard outcomes, and admin CRUD are excluded because their backend contracts
are not implemented or approved. The UI must not simulate or persist those
server-owned outcomes.

## Acceptance Criteria

- A visitor can register, login, refresh a session, logout, and see inline
  error feedback from the safe API envelope.
- Published Levels, Units, and Lessons render from the backend with deep links.
- A lesson renders vocabulary, Pinyin, Vietnamese meanings, examples, grammar,
  optional audio players, and a browser-only handwriting canvas.
- Content loading, empty, and failure states are readable and offer a retry.
- All primary controls are keyboard reachable, visibly focused, labelled, and
  usable at 375 px, 768 px, 1024 px, and 1440 px without horizontal scrolling.

## Non-goals

- LocalStorage learner progress, mock quiz scoring, mock dashboard metrics, or
  mock admin mutations.
- Handwriting recognition, grading, or stroke persistence.
- New backend endpoints or contract changes.

## Docs Impact

- **Precheck**: Impacted.
- **Canonical docs**: `001-chinese-learning-mvp/spec.md`,
  `001-chinese-learning-mvp/plan.md`,
  `001-chinese-learning-mvp/contracts/api-v1.md`, and `../frontend/README.md`.
- **Expected updates**: Document the API base URL and learner-shell boundary;
  update MVP task traceability after verification.
- **Final check**: Synced. `../frontend/README.md` documents the API base URL,
  delivered routes, and deferred contract-dependent features.
