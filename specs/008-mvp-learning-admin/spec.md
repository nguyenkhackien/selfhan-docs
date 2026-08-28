# Feature Specification: MVP Learning And Admin Completion

**Feature ID**: 008-mvp-learning-admin
**Work Type**: feature
**Created**: 2026-08-28
**Status**: Approved
**Input**: Complete the remaining Must-have and Should-have MVP features, but
do not perform release hardening.

## Clarification Gate

The user confirmed that both priority groups are in scope and that release
hardening is out of scope. See `clarifications.md`.

## User Stories And Testing

### User Story 1: Practice And Preserve Learning (Priority: P1)

As a learner, I can take lesson quizzes and have my study progress saved, so
that I can return on another device and see reliable progress.

**Independent Test**: Register a learner, complete a lesson quiz, mark required
sections seen, sign in again, and verify completion and dashboard counts.

**Acceptance Scenarios**:

1. Given a published lesson quiz, when a learner submits answers, then the
   server validates ownership and question/option membership, calculates the
   score, and records an immutable attempt.
2. Given all required lesson sections have been seen and the latest relevant
   quiz score is at least 70%, when progress is read, then the lesson is
   completed exactly once for the learner.
3. Given an activity on the current application day, when the dashboard loads,
   then completed lessons, learned words, streak, and continuation state are
   derived from the learner's persisted records.

### User Story 2: Maintain Learning Content (Priority: P1)

As an administrator, I can maintain curriculum and quiz content without direct
database access, so that learners see only valid published content.

**Independent Test**: An admin creates and publishes a lesson with vocabulary
and a quiz; a learner can read it; a learner cannot call an admin mutation.

**Acceptance Scenarios**:

1. Given an authenticated admin, when valid Level, Unit, Lesson, vocabulary,
   grammar, or quiz content is submitted, then it is persisted with validated
   order and publish/archive state.
2. Given an authenticated learner, when an admin route is requested, then the
   API returns forbidden and does not change content.

### User Story 3: Review At The Right Time (Priority: P2)

As a learner, I can review due cards, tag useful vocabulary, take a Unit quiz,
and inspect my results, so that I can retain knowledge after the first lesson.

**Independent Test**: Review a due card, reload the session, verify its next
review date and tag, complete a Unit quiz, and read updated statistics.

**Acceptance Scenarios**:

1. Given a due review card, when a learner rates it, then the server records
   the event and applies the documented deterministic schedule.
2. Given a learner tags a vocabulary item or completes a Unit quiz, when the
   relevant learner view reloads, then the learner sees only their own state
   and aggregated correctness statistics.

## Functional Requirements

- **FR-001**: Provide authenticated quiz-attempt and lesson-progress APIs that
  score server-owned answers and derive completion from required sections and a
  70% latest score.
- **FR-002**: Provide an authenticated dashboard API with completed lessons,
  learned words, streak, continuation, due-review count, and accuracy.
- **FR-003**: Provide an administrator-only CRUD boundary for Level, Unit,
  Lesson, Vocabulary, GrammarPoint, Quiz, QuizQuestion, and QuizOption,
  including ordering and publish/archive transitions.
- **FR-004**: Provide frontend learner routes for quiz, progress, dashboard,
  flashcard review, tags, Unit quiz, and statistics using feature-owned API
  clients and explicit loading/error states.
- **FR-005**: Provide frontend administrator routes and forms for the accepted
  core content operations; learner users must not see or access them.
- **FR-006**: Provide due review, deterministic SRS updates, per-user
  vocabulary tags, Unit quiz access, and correctness statistics.
- **FR-007**: Preserve existing public curriculum and HSK read contracts.

## Non-Functional Requirements

- **NFR-001**: All learner state and admin mutations enforce authentication,
  ownership, role checks, DTO validation, and the existing safe error envelope.
- **NFR-002**: New persisted state uses additive immutable migrations,
  constraints, indexes, and deterministic clock-controlled tests.
- **NFR-003**: New UI controls are keyboard-operable, labelled, focus-visible,
  responsive at 375px and 1024px, and have retryable loading/error states.
- **NFR-004**: No feature persists handwriting strokes, exposes quiz answer
  keys, or introduces external learning-data requests.

## Edge Cases

- Invalid, duplicated, unpublished, or cross-user quiz/review/tag input is
  rejected without mutating learner state.
- A repeated progress event or completed lesson does not double-count learned
  words or a daily streak.
- A learner with no due cards, attempts, or tags sees an explicit empty state.
- Admin archive hides content from learner reads while preserving attempt and
  progress history.

## Key Entities

- **QuizAttempt and QuizAnswer**: Immutable learner submission and scored
  answer snapshot.
- **LessonProgress and LearningActivity**: Per-learner durable progress and
  streak source.
- **VocabularyReview and UserVocabularyTag**: Per-learner SRS event/state and
  personal tag.
- **Curriculum content and quiz entities**: Admin-owned ordered, publishable
  learning content.

## Success Criteria

- **SC-001**: A newly registered learner can complete a lesson, return in a new
  session, and see the same dashboard/progress/review state.
- **SC-002**: An administrator can publish valid content entirely through the
  UI while learner access remains read-only and published-only.
- **SC-003**: Due review, tags, Unit quiz, and statistics are persisted,
  ownership-safe, and accessible on desktop and mobile.

## Assumptions

- Existing legal demo seed remains the test content source; no third-party audio
  or course text is imported.
- SRS uses the existing documented intervals of 1, 3, 7, 14, and 30 days; an
  `again` rating schedules the next day.

## Out Of Scope

- Release hardening, deployment, monitoring, CI/CD, production migration
  rehearsal, and final release approval.
- Handwriting recognition, stroke persistence, payments, social features,
  leaderboards, and external translation/dictionary calls.

## Affected Documentation Domains

| Domain                   | Canonical Paths                                                              | Expected Action |
| ------------------------ | ---------------------------------------------------------------------------- | --------------- |
| Requirements and scope   | `specs/001-chinese-learning-mvp/spec.md`, this work item                     | Update          |
| UX and accessibility     | `specs/001-chinese-learning-mvp/plan.md`, frontend README                    | Update          |
| Architecture, data, API  | MVP data model/API contract, this work item `data-model.md` and `contracts/` | Update          |
| State, cache, validation | MVP data model/API contract, backend README                                  | Update          |
| Security and privacy     | MVP API contract, backend README                                             | Update          |
| Testing and traceability | MVP tasks, this work item tasks/review/converge                              | Update          |

## Docs Impact

- **Precheck result**: Impacted.
- **Canonical docs checked**: MVP specification, plan, data model, API
  contract, task traceability, and backend/frontend READMEs.
- **Canonical docs to update**: The listed paths plus this work item's data
  model and runtime contract.
- **Reason**: The accepted scope introduces private state, role-protected
  mutation APIs, learner and admin flows, and schema migrations.
- **Required verifier/owner**: Agent and Reviewer/QA.

## Requirement Traceability

| Requirement                 | User Story | Acceptance Scenario | Verification                                                   |
| --------------------------- | ---------- | ------------------- | -------------------------------------------------------------- |
| FR-001 to FR-002            | Story 1    | 1 to 3              | Backend use-case/controller tests; frontend learner flow tests |
| FR-003 / FR-005             | Story 2    | 1 to 2              | Admin authorization/API tests; admin UI tests                  |
| FR-004 / FR-006             | Story 3    | 1 to 2              | SRS clock tests; learner component and responsive checks       |
| FR-007 / NFR-001 to NFR-004 | All        | All                 | Full repository checks, API regression, docs review            |
