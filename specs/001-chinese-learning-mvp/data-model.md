# Data Model: Chinese Learning MVP

**Work Item ID**: `001-chinese-learning-mvp`
**Status**: Proposed contract for backend workers; validate against the first
reviewed migration before publishing a runtime contract.

## Modelling Rules

- Use UUID primary keys, UTC `timestamptz` timestamps, foreign keys, database
  constraints, and TypeORM migrations.
- Keep published content immutable to learners; archive rather than hard-delete
  content that has learner history.
- Keep answer and review events append-only. Derived progress may be updated
  transactionally from those events.
- Store Vietnamese text as Unicode, Pinyin with tone marks or tone numbers as
  authored, and canonical Hán tự without visual normalisation by the client.

## Identity And Authorization

| Entity | Key fields | Invariants |
| --- | --- | --- |
| `users` | `id`, normalized `email`, `password_hash`, `role`, `created_at` | Email unique case-insensitively; role is `learner` or `admin`; never return password hash. |
| `refresh_sessions` | `id`, `user_id`, `token_hash`, `expires_at`, `revoked_at`, `created_at` | Token hash only; one row represents one issued refresh token; rotation revokes the previous row. |

Refresh tokens are opaque, high-entropy random values rather than JWTs. The
access token contains only the subject ID, role, expiry, and an `access` token
type marker. Refresh rotation atomically revokes the current row and inserts
the replacement row; a reuse attempt revokes all active sessions for that user.

## Curriculum Content

| Entity | Key fields | Relations and invariants |
| --- | --- | --- |
| `levels` | `id`, `slug`, `title`, `description`, `sort_order`, `status` | Slug unique; status is draft, published, or archived. |
| `units` | `id`, `level_id`, `slug`, `title`, `description`, `sort_order`, `status` | `slug` is unique among non-archived records because the public route addresses a Unit directly; `(level_id, sort_order)` is unique among non-archived records. |
| `lessons` | `id`, `unit_id`, `slug`, `title`, `summary`, `sort_order`, `status`, `writing_character` | `slug` is unique among non-archived records because the public route addresses a Lesson directly; `(unit_id, sort_order)` is unique among non-archived records. |
| `vocabulary` | `id`, `hanzi`, `pinyin`, `meaning_vi`, `audio_url`, `status` | `audio_url` nullable HTTPS URL; content changes preserve historical attempt text through snapshots. |
| `lesson_vocabulary` | `lesson_id`, `vocabulary_id`, `sort_order` | Unique `(lesson_id, vocabulary_id)` and `(lesson_id, sort_order)`. |
| `example_sentences` | `id`, `vocabulary_id`, `hanzi`, `pinyin`, `meaning_vi`, `audio_url`, `sort_order` | Unique `(vocabulary_id, sort_order)`; audio optional. |
| `grammar_points` | `id`, `lesson_id`, `title`, `explanation_vi`, `examples_json`, `sort_order` | Unique `(lesson_id, sort_order)`; examples are structured display data, not executable HTML. |

## Assessment Content

| Entity | Key fields | Relations and invariants |
| --- | --- | --- |
| `quizzes` | `id`, `lesson_id` or `unit_id`, `title`, `kind`, `passing_score`, `status` | Exactly one target: lesson or unit; kind is lesson or unit; 0 to 100 score threshold. |
| `quiz_questions` | `id`, `quiz_id`, `type`, `prompt`, `audio_url`, `correct_answer`, `sort_order` | Type is one of the four MVP modes; unique order per quiz; required media for listening type. |
| `quiz_options` | `id`, `question_id`, `label`, `sort_order` | Multiple choice needs exactly four distinct options and one server-held correct answer. |

## Learner State And Events

| Entity | Key fields | Relations and invariants |
| --- | --- | --- |
| `lesson_progress` | `user_id`, `lesson_id`, `sections_seen`, `status`, `first_completed_at`, `last_activity_at` | Unique user/lesson; completion is idempotent; state belongs only to its user. |
| `quiz_attempts` | `id`, `user_id`, `quiz_id`, `score`, `question_count`, `submitted_at` | Immutable after submit; server calculates score. |
| `quiz_answers` | `id`, `attempt_id`, `question_id`, `selected_option_id`, `is_correct`, `answered_at` | One answer per question/attempt; answer keys never come from client. |
| `vocabulary_reviews` | `id`, `user_id`, `vocabulary_id`, `rating`, `reviewed_at`, `next_review_at`, `srs_stage` | Rating is again, hard, good, or easy; scheduling computed server-side. |
| `user_vocabulary_tags` | `user_id`, `vocabulary_id`, `tag` | Tag is favorite or difficult; unique user/vocabulary/tag. |
| `learning_activities` | `id`, `user_id`, `activity_date`, `type`, `source_id`, `created_at` | At most one daily streak-qualifying row per user/day/type/source as defined by backend. |

## Derived Business Rules

1. A lesson becomes complete after required sections are seen and the latest
   submitted lesson quiz meets or exceeds 70%.
2. A vocabulary item is counted as learned once per user when a containing
   lesson is first completed. Repeating a lesson cannot increment the total.
3. A streak counts at least one qualifying activity per application day in
   `Asia/Ho_Chi_Minh` until user timezone is deliberately introduced.
4. SRS starts at `new`, then schedules 1, 3, 7, 14, and 30 days for successful
   reviews. An `again` result resets to `learning` and schedules the next day.
5. Quiz answers are evaluated against server-owned answer data; UI receives
   only the information required to render the current question.

## Migration And Retention Notes

- The foundation migration creates identity and ordered curriculum foreign keys
  and indexes. Later learning-state migrations add the dashboard and due-review
  tables and their indexes.
- Do not create a hard-delete endpoint for published curriculum in the first
  release. Archive status avoids orphaning historical attempts.
- Do not persist writing-canvas strokes in the MVP. If this changes later, it
  requires a privacy review, explicit retention rule, and new migration.
