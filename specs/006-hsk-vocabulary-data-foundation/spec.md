# Feature Specification: HSK Vocabulary Data Foundation

**Feature ID**: `006-hsk-vocabulary-data-foundation`
**Work Type**: `feature`
**Created**: `2026-08-27`
**Status**: Approved
**Input**: Build the data layer first for HSK 3.0 levels 1–6 and combined 7–9, using CVDICT Vietnamese definitions, Hán–Việt readings, and the accepted attribution requirements.

## Clarification Gate

Resolved decisions are recorded in `clarifications.md`.

## User Stories And Testing

### User Story 1: Trustworthy HSK Vocabulary Foundation (Priority: P1)

As a SelfHan maintainer, I can import the approved HSK vocabulary snapshot with Vietnamese meanings and Hán–Việt readings, so later learner features use a consistent local dataset without external runtime dependencies.

**Why this priority**: HSK browsing and writing practice require licensed, traceable vocabulary data before UI can be built safely.

**Independent Test**: Build the normalized dataset from locked source snapshots and verify counts, match status, senses, level bands, and source metadata.

**Acceptance Scenarios**:

1. Given the locked HSK and CVDICT revisions, when the data builder runs, then it creates 10,969 HSK entries across bands 1–6 and 7–9 with source-order preserved.
2. Given a source word has multiple CVDICT entries, when it is imported, then every Vietnamese sense remains ordered and available for review.
3. Given a source word lacks a CVDICT match or a complete Hán–Việt reading, when it is imported, then it is marked `needs_review` and receives no synthetic meaning or reading.

### User Story 2: License-Aware Dataset Distribution (Priority: P1)

As a learner or data user, I can see the provenance and license of imported vocabulary, so SelfHan uses community data legally and transparently.

**Independent Test**: Check the data license document, source lock metadata, generated attribution, and visible footer links.

**Acceptance Scenarios**:

1. Given a visitor opens the app, when the footer is rendered, then it names CVDICT and CC-CEDICT with an accessible link to the CC BY-SA 4.0 license.
2. Given a maintainer inspects the generated data directory, when they read `DATA_LICENSE.md`, then they can identify the source revisions, licenses, modifications, and ShareAlike duties.

## Functional Requirements

- **FR-001**: Produce a normalized, version-locked local HSK dataset from HSK, CVDICT, curated Hán–Việt, and Unihan inputs.
- **FR-002**: Preserve the source's exclusive HSK order and represent `new-7` as `HSK 7–9`.
- **FR-003**: Preserve all CVDICT Vietnamese senses and make unmatched or incomplete records explicit review candidates.
- **FR-004**: Store imported HSK words, senses, and character readings in dedicated database tables through an idempotent seed/import command.
- **FR-005**: Include source lock, attribution, license, and modification records with the distributed dataset.
- **FR-006**: Render the accepted vocabulary attribution in the frontend footer.

## Non-Functional Requirements

- **NFR-001**: No learner-facing runtime request may contact a source repository, CDN, or translation service.
- **NFR-002**: The build/import process must reject malformed rows and report unmatched vocabulary without losing valid rows.
- **NFR-003**: The importer must preserve database integrity with constraints and a transaction.
- **NFR-004**: Generated data must be deterministic from source locks and verified checksums.

## Edge Cases

- One simplified word maps to multiple CVDICT rows; retain all ordered senses.
- A HSK word has no CVDICT match; import the word with `needs_review` and no invented definition.
- A word includes a character without a Hán–Việt reading in either source; set `sinoViet` to null and mark `needs_review`.
- Source file version, hash, or parser format does not match the lock; stop the command before a database write.

## Key Entities

- **HskVocabulary**: Source-ordered HSK word and provenance record.
- **HskVocabularySense**: One source-imported Vietnamese sense for a vocabulary word.
- **HskCharacterReading**: Source-attributed Hán–Việt reading for one character.
- **Data source lock**: Immutable revisions, URLs, checksums, and licenses used to create the dataset.

## Success Criteria

- **SC-001**: The seeded database has the locked number of HSK records and a deterministic review report.
- **SC-002**: Dataset attribution is complete in the repository and visible in the app footer.
- **SC-003**: Backend build, lint, typecheck, unit tests, migration checks, and frontend verification pass.

## Assumptions

- CVDICT source-imported definitions may remain imperfect and will be reviewed by later admin tooling.
- The repository can store the normalized dataset and source lock, not the full upstream repositories.
- HSK data is not yet exposed by a new learner API in this data-first work item.

## Out Of Scope

- HSK browsing/search routes, learner HSK pages, quiz/progress, admin CRUD UI, export feature, audio, and three-mode writing UI.
- Automatic translation, external translation APIs, and runtime source-data fetches.
- Fabricating a Vietnamese definition, Hán–Việt reading, example sentence, or stroke sequence.

## Affected Documentation Domains

| Domain                   | Canonical Paths                                                                                                              | Expected Action                                                                   |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| Requirements and scope   | `specs/001-chinese-learning-mvp/spec.md`, `specs/001-chinese-learning-mvp/tasks.md`                                          | Update.                                                                           |
| UX and accessibility     | `../frontend/README.md`                                                                                                      | Update footer attribution description.                                            |
| Architecture, data, API  | `specs/001-chinese-learning-mvp/data-model.md`, `specs/001-chinese-learning-mvp/contracts/api-v1.md`, `../backend/README.md` | Update data model and import operation; API contract remains no new public route. |
| State, cache, validation | `specs/006-hsk-vocabulary-data-foundation/data-model.md`                                                                     | Add.                                                                              |
| Security and privacy     | `../backend/README.md`                                                                                                       | Update offline-source and no-runtime-fetch posture.                               |
| Testing and traceability | `specs/001-chinese-learning-mvp/tasks.md`, `../backend/README.md`, `../frontend/README.md`                                   | Update.                                                                           |

## Docs Impact

- **Precheck result**: Impacted.
- **Canonical docs checked**: `specs/001-chinese-learning-mvp/spec.md`, `specs/001-chinese-learning-mvp/data-model.md`, `specs/001-chinese-learning-mvp/contracts/api-v1.md`, `specs/001-chinese-learning-mvp/tasks.md`, `../backend/README.md`, and `../frontend/README.md`.
- **Canonical docs to update**: All paths listed in Affected Documentation Domains, plus `../backend/data/hsk/DATA_LICENSE.md`.
- **Reason**: This adds licensed HSK source data, a new persistence model, import operation, and visible application attribution.
- **Required verifier/owner**: Reviewer-QA.

## Requirement Traceability

| Requirement        | User Story  | Acceptance Scenario | Verification                                                           |
| ------------------ | ----------- | ------------------- | ---------------------------------------------------------------------- |
| FR-001 to FR-004   | Story 1     | Scenarios 1–3       | Parser/import unit tests, migration test, seed report.                 |
| FR-005 to FR-006   | Story 2     | Scenarios 1–2       | License-document review, frontend component test, accessibility check. |
| NFR-001 to NFR-004 | Stories 1–2 | All scenarios       | Source scan, malformed fixture test, transaction test, checksum test.  |
