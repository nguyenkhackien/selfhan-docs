# Tasks: Chinese Learning MVP

**Work Item ID**: `001-chinese-learning-mvp`
**Input**: `spec.md`, `plan.md`, `data-model.md`, and `contracts/api-v1.md`
**Status**: Active

## Task Rules

- Each dispatched worker receives one packet and one isolated repository branch.
  When the user explicitly authorizes direct implementation, the coordinator
  records the same verification and review evidence against the target branch.
- A dependent worker is not dispatched until the dependency is completed,
  independently reviewed, and any declared contract is approved.
- Only the currently dispatched run may be `[IN_PROGRESS]`.
- The coordinator updates this task file; workers update only their packet task
  status through harness callbacks.

## Phase 1: Foundation Contract

- [x] **T001** `[DONE]` Record the approved MVP scope, data model, proposed API,
      docs-impact precheck, and staged cross-repository plan in this work-item folder.
      Evidence: `spec.md`, `data-model.md`, `contracts/api-v1.md`, and `plan.md`.
- [x] **T002** `[DONE]` Implement run `001-auth-content-foundation` directly in
      the `backend` source checkout after the worker runtime proved unreliable.
      Deliver auth, role enforcement, initial content migration/read endpoints,
      legal demo seed, and unit coverage. Verification: backend format, lint,
      typecheck, unit tests, build, and integration-test runner. PostgreSQL
      migration/seed smoke testing remains part of review because no disposable
      database is configured. Trace: FR-001 to FR-005, FR-014, NFR-001 to NFR-004.
- [x] **T003** `[DONE]` Review the completed backend foundation diff and use the
      implemented auth/content foundation contract for direct frontend integration.
      Trace: FR-001 to FR-005, NFR-002 to NFR-004.

## Phase 2: Core Learner Path

- [x] **T004** `[DONE]` Implement run `002-learner-shell` directly in
      `frontend`. Deliver auth routes, typed API adapter, catalogue, lesson UI,
      responsive green sidebar navigation, and handwriting canvas. Trace: FR-001,
      FR-003 to FR-007, NFR-005 to NFR-007.
- [x] **T005** `[DONE]` Review the learner shell for contract adherence,
      responsive behavior, and accessibility. Evidence: frontend `npm run verify`
      passed with coverage and Playwright/axe checks. Trace: FR-001, FR-003 to
      FR-007, NFR-005 to NFR-007.
- [x] **T006** `[DONE]` Implement quiz-progress API directly in `backend`.
      `auth-content-v1` approval. Deliver quiz scoring, lesson progress, dashboard,
      flashcard review API, and `learning-state-v1`. Trace: FR-008 to FR-012.
- [x] **T007** `[DONE]` Review learner-state contract and implementation. Trace:
      FR-008 to FR-012, NFR-002 to NFR-004.
- [x] **T008** `[DONE]` Implement learner practice UI directly in `frontend`.
      `learning-state-v1` approval. Deliver quiz, flashcard, dashboard, and
      cross-session learner E2E. Trace: FR-008 to FR-012, NFR-005 to NFR-007.

## Phase 3: Core Content Operations

- [x] **T009** `[DONE]` Implement and review backend admin CRUD plus frontend UI.
      backend admin CRUD then frontend admin UI. Trace: FR-013, NFR-002 to NFR-007.

## Phase 4: Should-Have Learning Features

- [x] **T010** `[DONE]` Implement and review SRS, tags, Unit quiz, statistics,
      vocabulary tags, Unit quiz, statistics, and their learner UI. Trace: FR-015
      to FR-017.

## Phase 5: Release Review And Documentation

- [x] **T011** `[DONE]` Complete run
      `006-hsk-vocabulary-data-foundation`: record pinned licensed sources for the
      HSK 3.0 vocabulary snapshot, import bands 1–6 and combined 7–9 into
      dedicated tables, and preserve CVDICT/CC-CEDICT attribution. No source audio
      is imported. Evidence: `specs/006-hsk-vocabulary-data-foundation/`,
      backend data license, source lock, migration, and import verification.
      Trace: FR-014 and NFR-008.
- [ ] **T012** `[TODO]` Dispatch and review run 009 release hardening after all
      Must-have workers are approved. Trace: SC-001 to SC-004 and NFR-001 to NFR-008.
- [x] **T013** `[DONE]` Run `DOCS_IMPACT_FINAL_CHECK`, write `review.md` and
      `converge.md`, and reconcile worker commits before declaring the MVP complete.
- [x] **T014** `[DONE]` Complete run `007-hsk-learner-writing`: expose the
      imported HSK tables through public read-only query endpoints, add HSK
      browse/detail learner routes, and provide offline three-mode writing with
      local stroke assets. Trace: HSK learner specification FR-001 to FR-006 and
      NFR-001 to NFR-004.

## Persona Lens Notes

- **Planner**: T002 is alone in progress because `auth-content-v1` is the first
  shared contract. T004 and T006 are safe to prepare only after that contract is
  approved; T008 is strictly blocked by `learning-state-v1`.

## Traceability

| Task         | Requirement                                  | Verification                                                                 |
| ------------ | -------------------------------------------- | ---------------------------------------------------------------------------- |
| T002 to T003 | FR-001 to FR-005, FR-014, NFR-001 to NFR-004 | Backend worker tests, migration evidence, reviewer contract approval         |
| T004 to T005 | FR-001, FR-003 to FR-007, NFR-005 to NFR-007 | Frontend verify, axe, responsive E2E, reviewer verdict                       |
| T006 to T008 | FR-008 to FR-012                             | API tests, learner-practice E2E, approved contract                           |
| T009         | FR-013                                       | Admin role/API/integration/E2E verification                                  |
| T010         | FR-015 to FR-017                             | Deterministic SRS tests and UI E2E                                           |
| T011         | FR-014, NFR-008                              | Pinned source lock, attribution, database import, and no source-audio import |
| T012 to T013 | SC-001 to SC-004, NFR-001 to NFR-008         | Release test evidence, docs/review/convergence artifacts                     |
