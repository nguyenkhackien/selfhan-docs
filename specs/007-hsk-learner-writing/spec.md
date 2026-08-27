# Feature Specification: HSK Learner UI And Writing Practice

**Feature ID**: 007-hsk-learner-writing
**Work Type**: feature
**Created**: 2026-08-27
**Status**: Approved

## User Stories And Testing

### User Story 1: Browse HSK Vocabulary

As a learner, I can select an HSK band, search its vocabulary, and open a word
detail, so that I can study the locally imported HSK data without an external
dictionary request.

**Acceptance scenarios**:

1. A learner can open a level list for HSK 1–6 and the combined HSK 7–9 band.
2. A learner can search within a selected band and see paginated matching
   words with Pinyin, Vietnamese meanings, Hán–Việt reading when available,
   and an explicit review notice when data is incomplete.
3. A learner can open one word with all imported Vietnamese senses in source
   order.

### User Story 2: Write A Word In Character Order

As a learner, I can practice every Han character in a vocabulary word in its
display order, so that writing practice follows the word I am studying.

**Acceptance scenarios**:

1. The writing panel enumerates the Chinese characters of the selected word
   in order and provides previous and next controls.
2. Guidance mode renders local HanziWriter stroke animation with an accessible
   control to replay the animation.
3. Background-character mode shows the current character as a faint stroke
   guide under the learner drawing surface.
4. White-paper mode provides a blank four-quadrant practice sheet. It does not
   score, store, or claim recognition of handwriting.
5. Missing local stroke data is an explicit non-blocking state and never
   triggers a CDN request.

## Functional Requirements

- FR-001: Add public read-only HSK band, vocabulary-list, and vocabulary-detail
  API endpoints backed by the imported HSK tables.
- FR-002: Support a bounded band filter, text query, and stable cursor/limit
  pagination for HSK vocabulary.
- FR-003: Add frontend routes for HSK list and word detail using feature-owned
  API, hooks, types, pages, and components.
- FR-004: Bundle only the HanziWriter character files used by imported HSK
  vocabulary and load them from the deployed application origin.
- FR-005: Provide guidance, background-character, and white-paper writing
  modes, with character navigation in word order.
- FR-006: Preserve vocabulary and stroke-data attribution in the application
  and repository notices.

## Non-Functional Requirements

- NFR-001: No HSK learner request may contact a source repository, CDN, or
  translation service.
- NFR-002: HSK search and pagination inputs are strictly validated and bounded.
- NFR-003: The writing interaction is usable with keyboard controls and has
  semantic labels, status feedback, visible focus, and no color-only state.
- NFR-004: The app must retain its existing curriculum endpoints and learner
  routes without changing their response contracts.

## Out Of Scope

- Handwriting recognition, scoring, persistence, export, audio, SRS, quiz,
  dashboard, and admin management.
- Manual correction workflow for needs-review vocabulary.
- Any remote stroke-data fallback.

## Docs Impact Precheck

- **Result**: Impacted.
- **Canonical docs to update**: MVP API contract, data model, tasks, backend
  README, frontend README, and stroke-data license notices.
- **No migration expected**: HSK persistence already exists; this feature adds
  read models and local static assets only.
