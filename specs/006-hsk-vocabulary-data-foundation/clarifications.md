# Clarifications: HSK Vocabulary Data Foundation

**Work Item ID**: `006-hsk-vocabulary-data-foundation`
**Date**: `2026-08-27`

## Accepted Decisions

- Use HSK 3.0 vocabulary from `drkameleon/complete-hsk-vocabulary`: levels 1–6 are separate and the source's `new-7` becomes the learner-facing band `HSK 7–9`.
- Use CVDICT as the Vietnamese-definition source under CC BY-SA 4.0, with attribution, source metadata, and a ShareAlike data license document.
- Use curated Hán–Việt character data where available and Unicode Unihan `kVietnamese` as a per-character fallback. Do not invent a reading when neither source has one.
- Make data ingestion reproducible and offline at application runtime. A maintainer runs the importer explicitly; the learner application never calls the source repositories or a translation API.
- Deliver data foundation before HSK browsing and three-mode writing UI. Add only the requested footer attribution in this work item.

## Scope Boundaries

- Imported definitions are source-imported, not product-owner-reviewed translations. Unmatched vocabulary and missing Hán–Việt readings remain visible to the future admin review workflow as `needs_review`.
- The currently accepted feature does not add quiz/progress, admin CRUD UI, HSK learner routes, or writing-mode UI.
