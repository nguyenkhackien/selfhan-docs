# API Contract: Chinese Learning MVP V1

**Work Item ID**: `001-chinese-learning-mvp`
**Status**: Proposed. A worker implementation may not broaden this contract;
the orchestrator publishes the reviewed runtime contract before frontend
integration.

## Common Rules

- Base path: `/api/v1`.
- Content type: `application/json` unless an endpoint only returns `204`.
- All errors use the existing envelope:

```json
{
  "error": {
    "code": "MACHINE_CODE",
    "message": "Safe message",
    "details": null,
    "requestId": "req_123"
  }
}
```

- Bearer access tokens authenticate private routes. Refresh tokens are only
  carried in the `selfhan_refresh` HttpOnly cookie scoped to `/api/v1/auth`.
- All learner state routes are scoped to the authenticated principal; request
  bodies never select `userId` or role.

## Foundation Contract Boundary

The first backend foundation delivers only Authentication and the four public
catalogue reads in this document. Lesson progress, quiz metadata, review cards,
dashboard values, and admin mutations remain proposed endpoints for their
later named contracts. A client must not expect those fields from the
foundation responses.

## Authentication

| Method and path       | Request                   | Success                                                       | Errors                                         |
| --------------------- | ------------------------- | ------------------------------------------------------------- | ---------------------------------------------- |
| `POST /auth/register` | `{ "email", "password" }` | `201` `{ "accessToken", "user" }` plus refresh cookie         | `VALIDATION_ERROR`, `EMAIL_ALREADY_REGISTERED` |
| `POST /auth/login`    | `{ "email", "password" }` | `200` `{ "accessToken", "user" }` plus rotated refresh cookie | `VALIDATION_ERROR`, `INVALID_CREDENTIALS`      |
| `POST /auth/refresh`  | No body; refresh cookie   | `200` `{ "accessToken", "user" }` plus rotated refresh cookie | `INVALID_REFRESH_TOKEN`                        |
| `POST /auth/logout`   | No body; cookie optional  | `204`, refresh cookie cleared                                 | Never reveals session existence                |
| `GET /auth/me`        | Access token              | `200` `{ "id", "email", "role", "createdAt" }`                | `INVALID_ACCESS_TOKEN`                         |

Password validation is 12 to 128 characters. Email is trimmed and lowercased
server-side before uniqueness checks. The response user object never contains a
password hash, refresh token, or internal session ID.

The refresh cookie is named `selfhan_refresh`, has `HttpOnly`, `SameSite=Strict`,
and path `/api/v1/auth`. It has the `Secure` attribute in production. Refresh
tokens are opaque random values; only their SHA-256 hashes are stored by the
backend.

## Public Learner Content

| Method and path                    | Success shape                       | Rules                                                              |
| ---------------------------------- | ----------------------------------- | ------------------------------------------------------------------ |
| `GET /levels`                      | `{ "items": [LevelSummary] }`       | Published levels only.                                             |
| `GET /levels/:levelSlug`           | `LevelDetail` with published Units  | Stable `sortOrder`.                                                |
| `GET /units/:unitSlug`             | `UnitDetail` with published Lessons | Foundation returns content only; progress is added later.          |
| `GET /lessons/:lessonSlug`         | `LessonDetail`                      | Vocabulary, examples, grammar, and writing character only.         |
| `POST /lessons/:lessonId/progress` | `LessonProgress`                    | Body only declares viewed section IDs; backend derives completion. |

`LessonDetail` contains a display-safe content projection. The later learning
contract may add quiz prompt and option data, but it never exposes
`correctAnswer` or server-only answer keys.

### Foundation Response Shapes

```json
{
  "items": [
    {
      "id": "uuid",
      "slug": "starter-chinese",
      "title": "Tiếng Trung nhập môn",
      "description": "...",
      "sortOrder": 1
    }
  ]
}
```

`GET /levels/:levelSlug` returns the same Level fields plus `units`; each Unit
has `id`, `slug`, `title`, `description`, and `sortOrder`. `GET /units/:unitSlug`
returns those Unit fields plus `lessons`; each Lesson has `id`, `slug`, `title`,
`summary`, `writingCharacter`, and `sortOrder`.

`GET /lessons/:lessonSlug` returns the Lesson fields plus ordered `vocabulary`
and `grammarPoints`. Vocabulary has `id`, `hanzi`, `pinyin`, `meaningVi`,
nullable `audioUrl`, and ordered `examples`. Each example has `hanzi`,
`pinyin`, `meaningVi`, nullable `audioUrl`, and `sortOrder`. A grammar point
has `id`, `title`, `explanationVi`, string-array `examples`, and `sortOrder`.
No response exposes a `status`, password hash, refresh-session hash, or quiz
answer data.

`levelSlug`, `unitSlug`, and `lessonSlug` are lowercase kebab-case identifiers.
Level, Unit, and Lesson slugs are globally unique among non-archived records,
which makes their direct public URLs unambiguous.

## Public HSK Vocabulary

The HSK source snapshot is separate from administrator-owned curriculum
content. Its three public read-only endpoints neither require a learner session
nor alter curriculum, progress, quiz, or review responses.

| Method and path           | Success shape                   | Rules                                                                                                                            |
| ------------------------- | ------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `GET /hsk/bands`          | `{ "items": [HskBandSummary] }` | Returns bands 1 through 7 in order; 7 displays as HSK 7–9.                                                                       |
| `GET /hsk/vocabulary`     | `HskVocabularyPage`             | Optional `band` is 1 through 7, `query` is at most 100 characters, `cursor` is opaque, and `limit` is 1 through 50 (default 24). |
| `GET /hsk/vocabulary/:id` | `HskVocabularyDetail`           | `id` is a UUID; senses retain the imported source order.                                                                         |

`HskBandSummary` contains `band`, `displayBand`, and `count`.
`HskVocabularyPage` contains `items` and nullable `nextCursor`. A summary item
contains `id`, `hskBand`, `sourceOrder`, `simplified`, `pinyin`, nullable
`sinoViet`, nullable `primaryMeaning`, and `importStatus` (`ready` or
`needs_review`). Detail adds nullable `traditional` and `frequency`, plus the
ordered string-array `senses`. Searches match imported local fields only; the
backend never requests a dictionary, translation service, or source repository
at runtime.

## Quiz, Review, And Dashboard

| Method and path                              | Request                                                 | Success                                                                                             | Rules                                                                        |
| -------------------------------------------- | ------------------------------------------------------- | --------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| `POST /quizzes/:quizId/attempts`             | `{ "answers": [{ "questionId", "selectedOptionId" }] }` | Score plus per-question correctness and next action                                                 | Server validates ownership, published status, option membership, and scores. |
| `GET /reviews/due`                           | Optional bounded cursor                                 | Due review cards                                                                                    | Never returns another learner's schedule.                                    |
| `POST /reviews`                              | `{ "vocabularyId", "rating" }`                          | Updated SRS state and next review time                                                              | Server computes schedule.                                                    |
| `PUT /vocabulary/:vocabularyId/tags/:tag`    | No body                                                 | `204`                                                                                               | Tag is `favorite` or `difficult`.                                            |
| `DELETE /vocabulary/:vocabularyId/tags/:tag` | No body                                                 | `204`                                                                                               | Idempotent.                                                                  |
| `GET /dashboard`                             | `DashboardSummary`                                      | `learnedWords`, `completedLessons`, `currentStreak`, `continueLesson`, `dueReviewCount`, `accuracy` | Learner only.                                                                |

## Admin Content Management

All `/admin/*` routes require the `admin` role. They use plural resource nouns
and standard `GET`, `POST`, `PATCH`, and archive transitions. Every create or
update explicitly accepts only mutable fields. Admin endpoints must support
Level, Unit, Lesson, Vocabulary, GrammarPoint, Quiz, QuizQuestion, and
QuizOption management, including content ordering and status transition.

## Contract Ownership And Versioning

- Backend owns API validation, authorization, persisted state, and error codes.
- Frontend owns rendering, client loading/error states, and browser canvas
  interaction. It must not recalculate completion, quiz score, streak, or SRS.
- Any contract change after frontend integration requires an updated reviewed
  contract and a new dispatch dependency; no worker changes an approved contract
  unilaterally.
