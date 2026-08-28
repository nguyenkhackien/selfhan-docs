# Data Model: MVP Learning And Admin Completion

## Learning State

| Entity                 | Key fields                                                                                  | Invariants                                                                                              |
| ---------------------- | ------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `quiz_attempts`        | `id`, `user_id`, `quiz_id`, `score`, `question_count`, `submitted_at`                       | Immutable; one submission validates all question IDs and options against the published quiz.            |
| `quiz_answers`         | `id`, `attempt_id`, `question_id`, `selected_option_id`, `is_correct`, `answered_at`        | One answer per question/attempt; answer keys never leave the backend before scoring.                    |
| `lesson_progress`      | `user_id`, `lesson_id`, `sections_seen`, `status`, `first_completed_at`, `last_activity_at` | Unique user/lesson; completion is idempotent and requires study sections plus a 70% latest lesson quiz. |
| `learning_activities`  | `id`, `user_id`, `activity_date`, `type`, `source_id`, `created_at`                         | Unique per learner/day/type/source; supports an Asia/Ho_Chi_Minh streak.                                |
| `vocabulary_reviews`   | `id`, `user_id`, `vocabulary_id`, `rating`, `reviewed_at`, `next_review_at`, `srs_stage`    | One mutable current state per learner/vocabulary with append-safe review semantics.                     |
| `user_vocabulary_tags` | `user_id`, `vocabulary_id`, `tag`                                                           | Unique user/vocabulary/tag; tag is `favorite` or `difficult`.                                           |

## Admin Content Boundary

Admin changes use the existing curriculum entities: Levels, Units, Lessons,
Vocabulary, GrammarPoints, Quizzes, QuizQuestions, and QuizOptions. Mutations
allow only declared authoring fields, retain ordering uniqueness per parent,
and use publish/archive state instead of hard deletes for learner-visible
content.

## Read Models

| Model               | Fields                                                                                  | Invariants                                                     |
| ------------------- | --------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| `DashboardSummary`  | completedLessons, learnedWords, currentStreak, continueLesson, dueReviewCount, accuracy | Derived only from the authenticated learner's persisted state. |
| `ReviewCard`        | vocabulary projection, due state, allowed ratings                                       | Never exposes another learner's review data.                   |
| `StatisticsSummary` | attempt counts, correct/total answers, accuracy by applicable period or target          | Aggregates only authenticated learner events.                  |

## Lifecycle Rules

- Quiz attempts and answers are append-only after submission.
- Review rating updates the learner/vocabulary schedule using fixed intervals:
  `again` = next day; successful stages = 1, 3, 7, 14, and 30 days.
- Publishing makes complete valid content learner-readable; archiving removes it
  from new learner reads without deleting historical learner records.
