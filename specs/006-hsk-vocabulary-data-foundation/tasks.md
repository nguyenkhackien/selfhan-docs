# Tasks: HSK Vocabulary Data Foundation

**Work Item ID**: `006-hsk-vocabulary-data-foundation`
**Input**: `spec.md`, `clarifications.md`, `research.md`, `data-model.md`, and `plan.md`
**Status**: Complete

## Phase 1: Setup

- [x] **T001** `[DONE]` Create approved feature, research, data-model, and plan artifacts. Evidence: the work-item folder records source revisions, licenses, data counts, and accepted scope.
- [x] **T002** `[DONE]` Confirm source schema and import feasibility. Evidence: pinned source audit finds 10,969 HSK entries, 10,941 CVDICT matches, and 859 multi-row sense matches.

## Phase 2: Tests First

- [x] **T003** `[DONE]` Add parser and normalizer tests for source lock, HSK
      order, CVDICT senses, unmatched records, and Hán–Việt fallback. Evidence:
      focused Jest tests failed before the builder existed, then passed with six
      assertions against fixtures and the generated snapshot.

## Phase 3: Implementation

- [x] **T004** `[DONE]` Create source lock, data license, deterministic
      builder, generated HSK snapshot, and review report. Evidence: locked build
      produced 10,969 entries and rejected unchecked source inputs before output.
- [x] **T005** `[DONE]` Add HSK source schema migration, TypeORM ownership,
      and idempotent transactional import command. Evidence: migration applied to
      local PostgreSQL; two imports resulted in 10,969 vocabulary rows, 32,329
      senses, 2,191 character readings, and zero duplicate band/word keys.
- [x] **T006** `[DONE]` Add footer attribution and focused accessibility
      coverage. Evidence: React Testing Library verifies CVDICT and CC BY-SA links;
      the existing axe suite passes with the semantic footer rendered.

## Phase 4: Verification

- [x] **T007** `[DONE]` Run focused parser/import tests and backend/frontend
      quality checks. Evidence: backend format, lint, typecheck, unit tests, and
      build pass; frontend format, lint, typecheck, unit tests, and build pass.
- [x] **T008** `[DONE]` Run migration status or disposable-database smoke test
      and diff/attribution checks. Evidence: TypeORM discovered and applied the
      add-only migration; live count query confirms the idempotent import.

## Phase 5: Docs And Review

- [x] **T009** `[DONE]` Synchronize canonical data, operations, attribution,
      and traceability docs. Evidence: backend and frontend READMEs, source MVP
      specification/data model/tasks, data license, and this work item are updated;
      the API contract was checked and needs no change because no route was added.
- [x] **T010** `[DONE]` Write `review.md`. Evidence: primary review fixed
      Unihan provenance and records no unresolved Critical, High, or Medium
      finding.
- [x] **T011** `[DONE]` Run final docs-impact check and write
      `converge.md`. Evidence: requirements, verification, review, traceability,
      and canonical docs converge with no open blocker.

## Persona Lens Notes

- **Planner**: T003 and T004 precede schema work because database imports must consume a tested normalized snapshot, not raw third-party files. T006 is independent after source attribution is finalized but remains sequenced to reduce simultaneous frontend/backend changes.

## Traceability

| Task         | Requirement                                 | Verification                                      |
| ------------ | ------------------------------------------- | ------------------------------------------------- |
| T003 to T004 | FR-001 to FR-003, NFR-001, NFR-002, NFR-004 | Parser/normalizer tests and locked-count report.  |
| T005         | FR-004, NFR-003                             | Migration and idempotent import tests.            |
| T006         | FR-005, FR-006                              | Footer test and data-license review.              |
| T007 to T008 | SC-001, SC-003                              | Backend/frontend checks and migration smoke test. |
| T009 to T011 | SC-002, NFR-001 to NFR-004                  | Docs review, QA review, and convergence.          |
