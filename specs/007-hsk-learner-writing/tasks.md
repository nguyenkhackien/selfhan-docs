# Tasks: HSK Learner UI And Writing Practice

**Work Item ID**: 007-hsk-learner-writing
**Status**: In Progress

## Implementation Tasks

- [ ] **T001** [IN_PROGRESS] Add HSK API contract tests and application
  read-model ports/use cases. Completion: tests define bounded public bands,
  list, and detail behavior before persistence wiring.
- [ ] **T002** [TODO] Add HSK TypeORM query repository, public controller, DTO
  validation, and module registration. Completion: routes return stable cursor
  pages and ordered detail senses.
- [ ] **T003** [TODO] Pin, extract, and license local HanziWriter character
  files for imported HSK characters. Completion: no browser request uses a CDN
  and unavailable data is detectable.
- [ ] **T004** [TODO] Add the feature-owned HSK frontend API, types, browse
  routes, query state, and vocabulary detail. Completion: learners can browse
  and search a band using deep links.
- [ ] **T005** [TODO] Implement three-mode word-order writing practice with
  HanziWriter and the existing pointer-canvas approach. Completion: guidance,
  background, and white-paper modes work with accessible controls.
- [ ] **T006** [TODO] Add focused frontend/backend tests, mobile and
  accessibility checks. Completion: user stories and offline behavior have
  fresh evidence.
- [ ] **T007** [TODO] Synchronize API, data, operation, attribution, and
  traceability docs. Completion: all paths in plan.md are updated or checked.
- [ ] **T008** [TODO] Review, final docs-impact check, and converge.
  Completion: no unresolved Critical, High, or Medium finding.

## Traceability

| Tasks | Requirements | Evidence |
| --- | --- | --- |
| T001 to T002 | FR-001, FR-002, NFR-002, NFR-004 | API tests, validation tests, existing contract regression. |
| T003 | FR-004, FR-006, NFR-001 | Asset lock, notice, and offline loader test. |
| T004 to T005 | FR-003, FR-005, NFR-003 | Frontend tests and manual responsive check. |
| T006 to T008 | All requirements | Quality gates, docs review, and convergence record. |
