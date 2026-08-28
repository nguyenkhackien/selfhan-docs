# API Contract: Learning And Admin V1

## Common Rules

- Base path: `/api/v1`.
- All learner state routes require a Bearer access token and derive `userId`
  from it; request bodies never select a user or role.
- All `/admin/*` routes require the existing administrator role.
- Existing safe error envelope, validation, and public curriculum/HSK contracts
  remain unchanged.

## Learner State

| Method and path                              | Request                                                 | Success                                    | Rules                                                      |
| -------------------------------------------- | ------------------------------------------------------- | ------------------------------------------ | ---------------------------------------------------------- |
| `GET /lessons/:lessonId/quizzes`             | None                                                    | Published lesson quiz metadata             | Never includes the answer key.                             |
| `GET /units/:unitId/quizzes`                 | None                                                    | Published Unit quiz metadata               | Never includes the answer key.                             |
| `GET /quizzes/:quizId`                       | None                                                    | Prompt, options, and passing score         | Never returns `isCorrect`.                                 |
| `POST /quizzes/:quizId/attempts`             | `{ "answers": [{ "questionId", "selectedOptionId" }] }` | Attempt score and per-question correctness | Server validates published ownership and scores.           |
| `GET /lessons/:lessonId/progress`            | None                                                    | Progress or not-started state              | Authenticated learner only.                                |
| `POST /lessons/:lessonId/progress`           | `{ "sectionsSeen": [string] }`                          | Lesson progress                            | Server derives completion; repeated events are idempotent. |
| `GET /dashboard`                             | None                                                    | `DashboardSummary`                         | Authenticated learner only.                                |
| `GET /reviews/due`                           | Bounded `limit` and optional opaque cursor              | `{ "items", "nextCursor" }`                | Returns only due cards for the learner.                    |
| `POST /reviews`                              | `{ "vocabularyId", "rating" }`                          | Updated review state                       | Rating is `again`, `hard`, `good`, or `easy`.              |
| `PUT /vocabulary/:vocabularyId/tags/:tag`    | None                                                    | `204`                                      | Tag is `favorite` or `difficult`.                          |
| `DELETE /vocabulary/:vocabularyId/tags/:tag` | None                                                    | `204`                                      | Idempotent.                                                |
| `GET /statistics`                            | Optional bounded filters                                | `StatisticsSummary`                        | Authenticated learner only.                                |

## Administrator Content

`/admin/levels`, `/admin/units`, `/admin/lessons`, `/admin/vocabulary`,
`/admin/grammar-points`, `/admin/quizzes`, `/admin/quiz-questions`, and
`/admin/quiz-options` use declared `GET`, `POST`, `PATCH`, and archive
transitions. DTOs whitelist mutable fields, validate required values and
ordering, and reject an incomplete published quiz. Quizzes are created as
drafts; after their questions/options exist, `PATCH` can publish them. Resources
with content status have `POST /admin/:resource/:id/archive`.

The frontend must treat response fields as display data and must never receive
the `correctAnswer` or any server-only answer key before a quiz submission is
scored.
