# Tasks: HSK Learner UI And Writing Practice

**Work Item ID**: 007-hsk-learner-writing
**Status**: Complete

## Implementation Tasks

- [x] **T001** [DONE] Add HSK API contract tests and application read-model
      ports/use cases. Evidence: HSK bands/list/detail use-case tests define
      bounded public behavior.
- [x] **T002** [DONE] Add HSK TypeORM query repository, public controller, DTO
      validation, and module registration. Evidence: controller/DTO tests plus the
      local API smoke test return cursor pages and ordered detail senses.
- [x] **T003** [DONE] Pin, extract, and license local HanziWriter character
      files for imported HSK characters. Evidence: `public/hsk-strokes/` contains
      selected data, source lock, notice, and ARPHIC license; component test checks
      application-origin loading and an unavailable non-blocking state.
- [x] **T004** [DONE] Add the feature-owned HSK frontend API, types, browse
      routes, query state, and vocabulary detail. Evidence: HSK component tests
      cover cursor navigation/reset; manual deep-link smoke test uses imported data.
- [x] **T005** [DONE] Implement three-mode word-order writing practice with
      HanziWriter and the existing pointer-canvas approach. Evidence: focused
      component tests and desktop/mobile manual checks cover guidance, background,
      white-paper, and character-order controls.
- [x] **T006** [DONE] Add focused frontend/backend tests, mobile and
      accessibility checks. Evidence: 27 frontend and 58 backend unit tests pass;
      375px browser check confirms no horizontal overflow and 44px mode controls.
- [x] **T007** [DONE] Synchronize API, data, operation, attribution, and
      traceability docs. Evidence: API contract, data model, task traceability, and
      both application READMEs are updated.
- [x] **T008** [DONE] Review, final docs-impact check, and converge. Evidence:
      `review.md` and `converge.md` record an approving verdict with no unresolved
      Critical, High, or Medium finding.

## Traceability

| Tasks        | Requirements                     | Evidence                                                   |
| ------------ | -------------------------------- | ---------------------------------------------------------- |
| T001 to T002 | FR-001, FR-002, NFR-002, NFR-004 | API tests, validation tests, existing contract regression. |
| T003         | FR-004, FR-006, NFR-001          | Asset lock, notice, and offline loader test.               |
| T004 to T005 | FR-003, FR-005, NFR-003          | Frontend tests and manual responsive check.                |
| T006 to T008 | All requirements                 | Quality gates, docs review, and convergence record.        |
