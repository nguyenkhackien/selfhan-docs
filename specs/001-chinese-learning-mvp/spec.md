# Feature Specification: Chinese Learning MVP

**Feature ID**: `001-chinese-learning-mvp`
**Work Type**: `feature`
**Created**: `2026-08-25`
**Status**: Active
**Input**: User-defined MVP scope for SelfHan, with MoSCoW priorities.

## Product Outcome

SelfHan becomes a Vietnamese-first Chinese learning web application. A learner
can create an account, follow a Level to Unit to Lesson path, learn vocabulary
and basic grammar, practice handwriting, take quizzes, review flashcards, and
see durable learning progress. An administrator can maintain the learning
content without database access.

The first shippable release is the **Must have** scope. The **Should have**
scope is a planned extension that uses the same account, content, and progress
models. **Could have** work does not delay the core release.

## User Stories And Testing

### User Story 1: Start A Personal Learning Journey (Priority: P1)

As a learner, I can register and sign in so that my study history follows me
between sessions and devices.

**Why this priority**: Accounts are required before progress, streaks, SRS, and
personal vocabulary tags have a reliable owner.

**Independent Test**: Register an email/password account, sign in, sign out,
then sign in again and retrieve the same profile.

**Acceptance Scenarios**:

1. Given an unused valid email and a valid password, when a visitor registers,
   then the system creates a learner account and opens an authenticated session.
2. Given a learner with valid credentials, when they sign in on a new browser,
   then the system authenticates them without exposing their password or refresh
   token to application JavaScript.
3. Given an unauthenticated request to a private learner or admin resource,
   when the request is sent, then the API returns a standard unauthenticated
   error and no private data.

### User Story 2: Learn A Structured Lesson (Priority: P1)

As a learner, I can navigate from a level to a unit to a lesson and study the
vocabulary and grammar placed in that lesson.

**Why this priority**: This is the main teaching path and defines the content
model used by all later learning features.

**Independent Test**: With seeded content, open a Level, choose a Unit, open a
Lesson, and see ordered vocabulary, Pinyin, Vietnamese meaning, optional audio,
examples, and grammar.

**Acceptance Scenarios**:

1. Given published content, when a learner opens a lesson, then content is
   rendered in its admin-defined order and unpublished content is not exposed.
2. Given a vocabulary item with an audio URL, when the learner activates audio,
   then the native player can play that URL and offers an accessible label.
3. Given a character presented for writing practice, when a learner draws or
   clears strokes, then the browser provides a usable canvas without claiming to
   recognise or score the character.

### User Story 3: Practice And Retain Vocabulary (Priority: P1)

As a learner, I can take quizzes and flashcard reviews so that I know what I
remember and the app preserves my results.

**Why this priority**: Active recall is the MVP learning loop; it supplies
progress and later SRS signals.

**Independent Test**: Complete one question of each quiz type, complete a
flashcard review, reload/sign in again, and observe the saved outcomes.

**Acceptance Scenarios**:

1. Given a lesson quiz, when a learner answers Hán tự to meaning, meaning to
   Hán tự, Pinyin, or listening questions, then the server evaluates the answer
   and records the attempt.
2. Given an unfinished lesson, when its required study sections are viewed and
   the learner achieves at least 70% on a submitted lesson quiz, then the lesson
   is marked complete.
3. Given a learner's first completed lesson, when the dashboard loads, then the
   learned-word count, completed-lesson count, and daily streak reflect that
   activity.

### User Story 4: Maintain Learning Content (Priority: P1)

As an administrator, I can create, update, reorder, publish, and archive
Levels, Units, Lessons, Vocabulary, Grammar, and Quizzes.

**Why this priority**: The MVP cannot scale beyond demo content if content
changes require direct database edits.

**Independent Test**: An admin creates and publishes a complete lesson; a
learner can study it; a learner cannot access the admin route or mutate content.

**Acceptance Scenarios**:

1. Given an authenticated admin, when they submit valid content changes, then
   the system persists them and validates all required fields and order values.
2. Given an authenticated learner, when they attempt an admin mutation, then
   the API returns a standard forbidden error and leaves content unchanged.

### User Story 5: Study At The Right Time (Priority: P2)

As a learner, I can review due words, mark vocabulary favorite or difficult,
see correctness statistics, and complete a Unit quiz.

**Why this priority**: These improve retention after the core loop is proven.

**Independent Test**: Submit correct and incorrect reviews, advance time in a
test clock, and verify that due items, tags, and statistics are correct.

### User Story 6: Personalise Motivation (Priority: P3)

As a learner, I can set a daily goal, search vocabulary, use dark mode, and see
badges.

**Why this priority**: Useful polish, but not needed to prove the learning loop.

## Functional Requirements

- **FR-001**: Support email/password registration, login, refresh, logout, and
  authenticated profile retrieval.
- **FR-002**: Assign every account one server-owned role: `learner` or `admin`.
- **FR-003**: Model published learning content as Level, Unit, and Lesson with
  stable order within each parent.
- **FR-004**: Model vocabulary with Hán tự, Pinyin, Vietnamese meaning, optional
  audio URL, ordered examples, and many-to-many lesson placement.
- **FR-005**: Model ordered grammar points attached to a lesson.
- **FR-006**: Render the complete published lesson on mobile and desktop.
- **FR-007**: Provide browser-only handwriting canvas controls: draw, clear, and
  reset template; do not classify handwriting or persist strokes by default.
- **FR-008**: Support four quiz question modes: `hanzi_to_meaning`,
  `meaning_to_hanzi`, `pinyin`, and `listening`.
- **FR-009**: Evaluate quiz attempts server-side and record answers, score,
  question type, and completion time.
- **FR-010**: Provide flashcard review using lesson vocabulary and record review
  outcomes.
- **FR-011**: Persist learner lesson progress, last activity, first completion,
  and completion state.
- **FR-012**: Calculate dashboard totals for learned words, completed lessons,
  current streak, and the lesson to continue.
- **FR-013**: Permit admins to create, update, reorder, publish, and archive
  every content entity in FR-003 through FR-008.
- **FR-014**: Seed a legal, small demo curriculum for development and E2E tests.
- **FR-015**: Implement four SRS states and the intervals 1, 3, 7, 14, and 30
  days for review outcomes.
- **FR-016**: Support favorite and difficult tags per learner and vocabulary.
- **FR-017**: Offer Unit quizzes and learner correctness statistics by question
  type and vocabulary.
- **FR-018**: Make daily goal, badges, search, and dark mode separately gated
  features that do not alter core learning data semantics.

## Non-Functional Requirements

- **NFR-001**: Use the existing React 19/Vite frontend and NestJS 11/PostgreSQL/
  TypeORM backend; do not replace either stack.
- **NFR-002**: API routes stay under `/api/v1`, validate external input, and use
  the existing safe error envelope and request ID header.
- **NFR-003**: Passwords use Argon2id; refresh tokens are hashed at rest and
  delivered in an HttpOnly secure cookie; authorization is enforced server-side.
- **NFR-004**: Use schema migrations only; TypeORM `synchronize` remains off.
- **NFR-005**: All primary flows work at 375 px, 768 px, 1024 px, and 1440 px
  with no horizontal scroll; controls have keyboard and accessible names.
- **NFR-006**: Body text and meaningful controls meet WCAG AA contrast; visible
  focus and reduced-motion preferences are respected.
- **NFR-007**: Each backend worker runs format, lint, typecheck, unit tests, and
  applicable integration tests. Each frontend worker runs its existing `verify`
  script and focused accessibility/E2E coverage for changed flows.
- **NFR-008**: Only licensed or team-authored HSK text, examples, and audio may
  be imported; the system must not scrape or reproduce protected course media.

## Edge Cases

- Duplicate normalized email returns a conflict without leaking account details.
- Expired, malformed, reused, or revoked refresh tokens fail safely and cannot
  create a new session.
- Empty levels, units, and lessons show a useful empty state; drafts never
  appear in the learner catalogue.
- A quiz without enough valid distractors remains unpublished; it cannot present
  duplicate answers or fewer than four options for a multiple-choice question.
- Audio unavailable at playback time is reported locally without losing lesson
  progress or quiz state.
- Repeating a lesson does not double-count learned vocabulary or completion.
- The streak boundary uses the learner profile timezone once introduced; until
  then it uses the application-configured `Asia/Ho_Chi_Minh` timezone.

## Key Entities

- **User**: Credential owner and role-bearing learner or administrator.
- **RefreshSession**: Rotating, revocable server session with a hashed token.
- **Level, Unit, Lesson**: Ordered, publishable curriculum hierarchy.
- **Vocabulary, ExampleSentence, GrammarPoint**: Learning material associated
  with a lesson in defined display order.
- **Quiz, QuizQuestion, QuizOption**: Authorable practice content with one
  answer-key model per question type.
- **LessonProgress, QuizAttempt, QuizAnswer, VocabularyReview**: Immutable or
  append-only learner events and derived progress state.
- **UserVocabularyTag**: Per-user favorite/difficult classification.

## Success Criteria

- **SC-001**: A new learner can register, complete a seeded lesson, log out, log
  in on a clean browser session, and see the same lesson completion and dashboard
  values.
- **SC-002**: An admin can create and publish a lesson containing vocabulary,
  grammar, audio URL, examples, and a valid quiz without direct SQL.
- **SC-003**: All four quiz modes and flashcard review save results that survive
  refresh and obey ownership restrictions.
- **SC-004**: The core learner and admin flows pass keyboard, screen-reader, and
  responsive E2E checks at the target viewport sizes.

## Assumptions

- The first login method is email/password. OAuth, email verification, password
  reset, and passkeys are follow-up security features, not MVP blockers.
- Audio is initially an admin-provided HTTPS URL. Upload, transcoding, and a
  media CDN are deferred until a storage provider is selected.
- The existing frontend's browser-only data remains a temporary demo adapter
  until the approved backend contracts are integrated.
- HSK 1-3 content import begins only after the product owner supplies licensed
  source material or confirms team-authored seed data.

## Implementation Readiness

The specification is implementation-ready for the backend foundation and the
frontend learner shell. The following decisions are deliberately required before
their later implementation phases; a worker must not invent them:

| Phase | Decision required | Owner |
| --- | --- | --- |
| Quiz and progress | Required lesson sections, exact quiz prompt/option schema for each mode, passing-score exceptions, and retry policy. | Product owner and backend contract reviewer |
| Dashboard and streak | Learner timezone selection and changes, qualifying activity types, and the empty-state definition for `continueLesson`. | Product owner |
| Admin content | Create/update DTO fields, bulk-reorder conflict policy, publish validation, and archive effects on learner history. | Product owner and backend contract reviewer |
| SRS and analytics | Exact four state names, rating-to-interval mapping, lapse behavior, and statistics denominators. | Product owner and backend contract reviewer |
| HSK 1-3 import | Licensed source, attribution requirements, and approved audio source. | Product owner/legal owner |

These decisions do not block the current foundation because it returns only
published content and does not create quiz, learner-state, or admin records.

## Out Of Scope

- AI chatbot, pronunciation scoring, handwriting recognition, community,
  leaderboard, and HSK 4-6.
- Claiming that canvas handwriting evaluates, recognises, or grades a character.
- Copying third-party course text or audio without a clear licence.

## Affected Documentation Domains

| Domain | Canonical Paths | Expected Action |
| --- | --- | --- |
| Requirements and scope | `specs/001-chinese-learning-mvp/spec.md` | Update |
| UX and accessibility | `specs/001-chinese-learning-mvp/plan.md` | Update |
| Architecture, data, API | `specs/001-chinese-learning-mvp/data-model.md`, `specs/001-chinese-learning-mvp/contracts/api-v1.md` | Update |
| State, cache, validation | `specs/001-chinese-learning-mvp/data-model.md`, `specs/001-chinese-learning-mvp/contracts/api-v1.md` | Update |
| Security and privacy | `specs/001-chinese-learning-mvp/contracts/api-v1.md`, `../backend/README.md` | Update |
| Testing and traceability | `specs/001-chinese-learning-mvp/tasks.md`, `../frontend/README.md`, `../backend/README.md` | Update |

## Docs Impact

- **Precheck result**: Impacted
- **Canonical docs checked**: `AGENTS.md`, `opencode-repos.tsv`,
  `../frontend/README.md`, `../backend/README.md`.
- **Canonical docs to update**: This work-item folder; `../backend/README.md`
  for auth/content API and setup; `../frontend/README.md` for account-backed
  progress and updated routes once each implementation worker is reviewed.
- **Reason**: The current frontend README explicitly says there is no backend or
  account synchronization, while this feature introduces both.
- **Required verifier/owner**: Reviewer/QA validates per-worker docs; the
  orchestrator validates cross-repo documentation before release convergence.

## Requirement Traceability

| Requirement | User Story | Acceptance Scenario | Verification |
| --- | --- | --- | --- |
| FR-001 to FR-002 | Story 1 | 1 to 3 | Backend auth unit/integration tests; frontend auth E2E |
| FR-003 to FR-007 | Story 2 | 1 to 3 | Content API tests; lesson and writing E2E |
| FR-008 to FR-012 | Story 3 | 1 to 3 | Quiz/progress tests; dashboard E2E |
| FR-013 to FR-014 | Story 4 | 1 to 2 | Admin authorization/integration tests; seeded E2E |
| FR-015 to FR-017 | Story 5 | Due-review scenario | SRS clock tests; stats E2E |
| FR-018 | Story 6 | Feature-gate scenario | Focused component/E2E tests |
| NFR-001 to NFR-008 | Stories 1 to 5 | All | Repository verification and QA checklist |
