# Implementation Plan: MVP Learning And Admin Completion

**Work Item ID**: 008-mvp-learning-admin
**Date**: 2026-08-28
**Spec**: `spec.md`

## Summary

Add the remaining learning-state and administrator boundaries to the existing
NestJS/TypeORM backend, then consume the published contract in feature-owned
React UI. Must-have and Should-have share the same authenticated learner state
but are implemented in dependency order; release hardening is excluded.

## Technical Context

- **Backend**: NestJS 11, TypeORM 0.3, PostgreSQL, Jest.
- **Frontend**: React 19, React Router 8, TypeScript, Vitest, Playwright.
- **Storage**: PostgreSQL with immutable additive migrations.
- **Testing**: Backend unit/controller tests and frontend component/flow tests;
  manual responsive checks at 375px and 1024px.
- **Constraints**: Existing public curriculum and HSK contracts remain stable;
  no external learning data or handwriting persistence.

## Constitution Check

- [x] Specification first: scope and acceptance are recorded in `spec.md`.
- [x] Clarification gate completed: both priority groups are accepted; release
      hardening is excluded.
- [x] Material Architect and Planner decisions are recorded below.
- [x] Requirements are traceable in the specification and tasks.
- [x] Simplicity preserved: existing module/feature boundaries are extended.
- [x] Testing strategy is defined before implementation.
- [x] Existing patterns remain authoritative.
- [x] Review and convergence are planned after implementation, not release.

## Persona Lens Notes

- **Architect**: Use a learner-state module for quiz/progress/review/dashboard
  data and an admin-curriculum module for privileged authoring, rather than
  expanding public curriculum controllers. Risk is duplicated projections;
  mitigation is shared TypeORM entities with separate read/write use cases.
- **Planner**: Complete migration and learning-state contract first, then its
  learner UI; admin backend/UI can proceed only after shared curriculum entity
  rules are stable. SRS follows quiz/progress because it reuses learner
  ownership and dashboard invalidation.

## Architecture And Data Flow

```text
Learner UI -> typed feature API -> authenticated learner-state controllers
  -> use cases -> learner state / curriculum entities -> PostgreSQL

Admin UI -> typed admin API -> role-protected admin controllers
  -> authoring use cases -> curriculum / quiz entities -> PostgreSQL
```

The server owns scoring, completion, SRS scheduling, ownership, and role
checks. The browser owns only in-flight selections, navigation, and visual
state. All writes trigger feature-local reloads of the changed learner/admin
read model.

## File-Level Impact

| Area                                        | Action | Responsibility                                                     |
| ------------------------------------------- | ------ | ------------------------------------------------------------------ |
| Backend learning-state module and migration | Create | Quiz, progress, dashboard, reviews, tags, statistics, constraints. |
| Backend admin curriculum module             | Create | Role-protected content authoring and publish/archive operations.   |
| Backend curriculum entities/reads           | Modify | Expose required relations without changing public projections.     |
| Frontend learner features/routes            | Create | Quiz, dashboard, progress, reviews, tags, and statistics UI.       |
| Frontend admin feature/routes               | Create | Protected authoring lists/forms and mutation states.               |
| Existing layout/router/styles/tests         | Modify | Navigation, deep links, responsive/accessibility verification.     |
| Canonical docs and READMEs                  | Modify | Runtime contracts, state model, traceability, operations.          |

## Testing Approach

- Add deterministic backend use-case tests for scoring, completion, streak,
  schedule, ownership, validation, and role denial.
- Add frontend component/flow tests for quiz selection/submission, dashboard,
  review/tag controls, admin error states, and protected navigation.
- Run backend/frontend format, lint, typecheck, tests, and builds after each
  coherent slice; run manual desktop and 375px checks for new interactive UI.

## Cross-Cutting Concerns

| Concern                                  | Decision                                                                                                  | Verification                                    |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------------- | ----------------------------------------------- |
| State, cache, and invalidation           | Reload only affected dashboard, progress, due-review, statistics, and admin list resources after a write. | Feature flow tests.                             |
| Validation, errors, retry, and offline   | DTO whitelist/bounds and existing retryable error components; no optimistic durable state.                | Negative API and component tests.               |
| Security, privacy, auth, and permissions | Existing access guard plus learner ownership and admin role guard.                                        | Auth/role and cross-user tests.                 |
| Accessibility                            | Native labelled controls, visible focus, status feedback, keyboard navigation, responsive controls.       | Component semantics and manual viewport checks. |
| Performance and reliability              | Bounded pages, indexed due-state queries, transaction for attempt/progress updates.                       | Repository/use-case tests and review.           |
| Compatibility, migration, and release    | Additive migration; no release hardening in scope.                                                        | Migration and regression test checks.           |

## Docs Impact Plan

- **Precheck result**: Impacted.
- **Canonical docs checked**: MVP specification, plan, data model, API contract,
  task traceability, and both application READMEs.
- **Canonical docs to sync**: The checked paths plus this work item's data model
  and `contracts/learning-admin-v1.md`.
- **Project-wide traceability**: Update
  `specs/001-chinese-learning-mvp/tasks.md` as each accepted delivery group
  completes.
- **Sync task**: T010.
- **Required verifier/owner**: Agent and Reviewer/QA.

## Risks And Mitigations

| Risk                                     | Impact                       | Mitigation                                                             |
| ---------------------------------------- | ---------------------------- | ---------------------------------------------------------------------- |
| Scoring or SRS is inconsistent           | Incorrect learner data       | Server-only rules and deterministic clock tests.                       |
| Admin mutation leaks unpublished content | Learner sees invalid content | Role guard, publish validation, archive reads, and denial tests.       |
| Cross-user reads or tags leak state      | Privacy violation            | Derive user from guard and test ownership in every write/read.         |
| Broad UI scope regresses shell routes    | Broken learning navigation   | Feature-owned routes/tests and full existing suite before convergence. |

## Supporting Docs

- `data-model.md`: Required for new durable learner state.
- `contracts/learning-admin-v1.md`: Required for cross-repository API changes.
- `research.md`: Not required; existing product decisions and framework patterns
  are sufficient.

## Complexity Exceptions

None.
