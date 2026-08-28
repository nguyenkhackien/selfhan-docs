# Tasks: MVP Learning And Admin Completion

**Work Item ID**: 008-mvp-learning-admin
**Input**: `spec.md`, `plan.md`, `data-model.md`, and
`contracts/learning-admin-v1.md`
**Status**: Complete (release hardening excluded)

## Implementation Tasks

- [x] **T001** `[DONE]` Add learning-state migration, entities, and
      test fixtures for attempts, progress, activities, reviews, and tags.
      Completion: additive schema has ownership, ordering, and due-query indexes.
- [x] **T002** `[DONE]` Add authenticated quiz scoring, progress, dashboard,
      review, tag, Unit quiz, and statistics use cases/controllers/DTOs. Completion:
      all learner-state contract routes are validated and ownership-safe.
- [x] **T003** `[DONE]` Add admin curriculum authoring use cases/controllers/
      DTOs for accepted content types. Completion: admin CRUD, publish/archive, and
      ordering are role-protected and learner reads remain published-only.
- [x] **T004** `[DONE]` Add backend deterministic tests and focused API contract
      tests. Completion: score/completion/streak/SRS/ownership/role denial are
      covered before frontend integration.
- [x] **T005** `[DONE]` Add learner quiz, progress, dashboard, review/tag, Unit
      quiz, and statistics frontend features/routes. Completion: all learner flows
      use typed API clients with loading/error/empty states.
- [x] **T006** `[DONE]` Add administrator frontend feature/routes/forms.
      Completion: admins can author accepted content, learners cannot reach or use
      protected UI/API actions.
- [x] **T007** `[DONE]` Add frontend focused tests plus responsive/accessibility
      checks. Completion: learner/admin flows are keyboard-operable at 375px and
      1024px with no horizontal overflow.
- [x] **T008** `[DONE]` Run backend/frontend quality gates and migration/API
      smoke checks. Completion: format, lint, typecheck, tests, and builds pass.
- [x] **T009** `[DONE]` Synchronize canonical data/API/README/traceability docs.
      Completion: all paths in the docs-impact plan are updated and checked.
- [x] **T010** `[DONE]` Review final diff and run the final docs-impact check.
      Completion: no unresolved Critical, High, or Medium finding.
- [x] **T011** `[DONE]` Write convergence evidence without release hardening.
      Completion: accepted feature scope is converged and release remains explicitly
      pending.

## Persona Lens Notes

- **Planner**: T001 to T004 establish the shared server contract before T005;
  T003 may use the stabilized curriculum entity rules but must complete before
  T006. T009 to T011 are blocked until all behavior and verification evidence
  exists.

## Traceability

| Tasks        | Requirements                                              | Verification                                                |
| ------------ | --------------------------------------------------------- | ----------------------------------------------------------- |
| T001 to T004 | FR-001, FR-002, FR-003, FR-006, NFR-001, NFR-002, NFR-004 | Backend deterministic/controller tests and migration smoke. |
| T005 to T007 | FR-004, FR-005, FR-006, NFR-003                           | Frontend component/flow and responsive checks.              |
| T008 to T011 | FR-007, NFR-001 to NFR-004                                | Full quality gates, docs review, and convergence.           |
