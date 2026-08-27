# Implementation Plan: Chinese Learning MVP

**Work Item ID**: `001-chinese-learning-mvp`
**Date**: `2026-08-25`
**Spec**: `spec.md`

## Summary

Deliver the MVP through reviewable repository runs. The coordinator may dispatch
an agent or, when the user explicitly authorizes it, implement directly in the
target repository; source-trust remains the specification and review authority.
The backend owns identity, content, assessment, and learner state. The frontend
keeps its current React/Vite application and replaces local durable state with a
versioned API client only after each backend contract is reviewed and published.

## Technical Context

- **Frontend**: React 19, Vite 8, React Router 8, TypeScript, Vitest, Playwright,
  axe-core. Existing learning UI and browser storage are at `../frontend/src/`.
- **Backend**: NestJS 11, TypeScript strict, PostgreSQL, TypeORM 0.3, Jest.
  Current API prefix and safe error envelope are established in `../backend/src/`.
- **Storage**: PostgreSQL is the system of record. Browser storage may retain
  non-sensitive UI preferences only after backend integration.
- **Testing**: Frontend uses `npm run verify`; backend uses format check, lint,
  typecheck, unit tests, integration tests, and build.
- **Target platform**: Responsive web at 375, 768, 1024, and 1440 px.
- **Deployment constraint**: API CORS and cookie domain settings must be supplied
  by the deployment environment; no source file contains production secrets.

## Constitution Check

- [x] Specification first: `spec.md` records accepted behaviour.
- [x] Clarification gate completed: remaining content-source uncertainty is a
  non-blocking dependency, not an implementation guess.
- [x] Architect and Planner decisions are recorded below.
- [x] Requirements are traceable in `spec.md` and `tasks.md`.
- [x] Simplicity preserved: separate repositories communicate only through
  reviewed HTTP contracts.
- [x] Testing strategy is defined per worker packet and per release run.
- [x] Existing repository patterns remain authoritative for every worker.
- [x] Review and convergence occur before any dependency is unlocked.

## Persona Lens Notes

- **Architect**: Choose staged backend-contract publication over parallel API and
  UI implementation because the existing frontend has no backend client and the
  backend has only foundation modules. The risk is slower initial throughput;
  the mitigation is independent frontend shell work only where it does not
  consume an unfinished contract.
- **Planner**: Use one OpenCode packet per repository per run. This matches the
  isolated-branch rule and prevents two workers from writing the same checkout
  or claiming the same contract.

## Architecture And Data Flow

```text
Learner browser
  -> React routes, accessible UI, browser-only writing canvas
  -> typed API client with access token in memory and credentialed refresh call
  -> NestJS controllers, guards, DTO validation, application use cases
  -> TypeORM repositories and PostgreSQL migrations

Administrator browser
  -> protected React admin routes
  -> admin-only API mutations
  -> published curriculum queried by learner endpoints
```

The backend is authoritative for roles, published visibility, quiz answers,
score, lesson completion, learned-word totals, streak, SRS, and statistics. The
frontend is authoritative only for ephemeral view state such as current card
face, quiz selection before submit, and canvas strokes.

## Run Sequence

| Run | Worker | Repository | Depends on | Delivers |
| --- | --- | --- | --- | --- |
| `001-auth-content-foundation` | `backend-foundation` | backend | None | Auth, role guard, content schema, migrations, published curriculum reads, small legal demo seed, reviewed `auth-content-v1` contract. |
| `002-learner-shell` | `frontend-shell` | frontend | `auth-content-v1` approved | Auth screens, API client, protected route shell, Level/Unit/Lesson pages, accessible writing canvas, responsive design. |
| `003-quiz-progress-api` | `backend-learning` | backend | `auth-content-v1` approved | Quiz attempts, lesson progress, dashboard, flashcard review API, reviewed `learning-state-v1` contract. |
| `004-learner-practice` | `frontend-practice` | frontend | `learning-state-v1` approved | Quiz UI, flashcard UI, dashboard, API-backed progress and E2E flows. |
| `005-admin-content-api` | `backend-admin` | backend | `auth-content-v1` approved | Admin content CRUD, publish/archive/reorder, reviewed `admin-content-v1` contract. |
| `006-admin-content-ui` | `frontend-admin` | frontend | `admin-content-v1` approved | Protected admin content forms, tables, validation, and responsive navigation. |
| `007-srs-and-analytics-api` | `backend-srs` | backend | `learning-state-v1` approved | Due queue, SRS scheduler, tags, Unit quiz, correctness statistics, reviewed `srs-v1` contract. |
| `008-srs-and-analytics-ui` | `frontend-srs` | frontend | `srs-v1` approved | Due review, difficult/favorite actions, Unit quiz, and statistics UI. |
| `009-release-hardening` | `frontend-qa` then `backend-qa` | frontend then backend | Core contracts approved | Cross-repo E2E, responsive/accessibility pass, ops docs, seed verification, release review. |

Runs 001 to 006 are the Must-have delivery path. Runs 007 and 008 are the
Should-have path. Daily goal, badges, search, and dark mode remain unplanned
unless the user promotes them after MVP acceptance.

## File-Level Impact

| Repository | Existing location | Planned responsibility |
| --- | --- | --- |
| backend | `src/modules/` | Add domain modules following the repository's presentation/application/domain/infrastructure structure. |
| backend | `src/infrastructure/database/migrations/` | Add immutable schema migrations only. |
| backend | `README.md`, `.env.example` | Document newly required development settings after review. |
| frontend | `src/app/`, `src/features/`, `src/data/` | Replace demo-only paths incrementally with API-backed features. |
| frontend | `src/styles.css`, accessibility and E2E tests | Apply responsive tokens and verify user journeys. |
| source-trust | `runs/*/workers/*` | Own OpenCode prompts, status, result, review, and approved contracts. |
| source-trust | `specs/001-chinese-learning-mvp/` | Own product decisions, canonical proposed contracts, task traceability, and release convergence. |

## Testing Approach

- Backend foundation: unit tests for credentials/role/published-content rules;
  integration tests for migrations, constraints, and API error envelope.
- Frontend shell: component tests for loading/error states; Playwright plus axe
  on auth, catalogue, lesson, and writing routes.
- Quiz/progress and SRS: deterministic clock tests for score, completion, streak,
  deduplication, schedule, and ownership.
- Admin: integration tests for validation and role denial; E2E creates content
  and confirms learner published visibility.
- Release hardening: clean database seed, new browser registration, complete
  lesson, sign in again, and complete a due review.

## Cross-Cutting Concerns

| Concern | Decision | Verification |
| --- | --- | --- |
| State, cache, and invalidation | Backend state is authoritative; client invalidates dashboard, lesson, and due-review queries after a mutation. | Refresh and new-session E2E preserves server state. |
| Validation, errors, retry, and offline | DTO validation rejects unknown input; frontend maps existing safe error codes to inline, focusable form feedback. Offline has an explicit retry state and never invents progress. | API negative tests; offline/error component tests. |
| Security, privacy, auth, and permissions | Argon2id, rotating hashed refresh token, HTTP-only cookie, access-token guard, server role/ownership checks, no writing stroke persistence. | Auth/role integration tests; response inspection. |
| Accessibility | Semantic controls, labels, visible focus, focusable error summaries, contrast, keyboard alternatives, 44 px targets, reduced motion. | Playwright axe and keyboard scenarios at required viewports. |
| Performance and reliability | Ordered content queries avoid N+1; dashboard/due endpoints have proper indexes and bounded pagination. | Query/integration review; browser performance smoke test. |
| Compatibility, migration, and release | Contract only changes after review; migrations are additive and immutable; seed fixtures use legal content. | Migration run/revert in test DB; contract approval record. |

## Docs Impact Plan

- **Precheck result**: Impacted
- **Canonical docs checked**: `AGENTS.md`, `opencode-repos.tsv`,
  `../frontend/README.md`, `../backend/README.md`.
- **Canonical docs to sync**: This work item, reviewed runtime contracts under
  `runs/*/contracts/`, and each affected repository README after implementation.
- **Project-wide traceability**: No existing feature traceability document was
  found; `spec.md` requirement traceability and `tasks.md` are the canonical
  mapping for this feature.
- **Sync task**: `T010`.
- **Required verifier/owner**: Orchestrator plus Reviewer/QA for worker changes;
  product owner provides licensed HSK/audio source before content import.

## Risks And Mitigations

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Frontend starts against an unstable API | Rework and broken learner flows | Do not dispatch contract-dependent UI until backend review and contract approval. |
| Unlicensed HSK text or audio | Legal and release risk | Use only team-authored demo seed until a licensed source is supplied. |
| Auth implementation leaks tokens | Account compromise | Use backend security guide, HttpOnly cookies, response DTOs, and auth tests. |
| Data migration breaks learner history | Data loss or outages | Add immutable migrations, constraints, test DB run/revert, and archive content instead of delete. |
| SRS rule ambiguity | Incorrect scheduling | Start with fixed documented intervals and deterministic clock tests. |
| Mobile lesson UI becomes inaccessible | Core user path unusable | Treat responsive and keyboard/axe E2E as release gates rather than polish. |

## Supporting Docs

- `data-model.md`: Required because the MVP introduces persistent cross-repo
  learning state.
- `contracts/api-v1.md`: Required because frontend and backend change together.
- `runs/*/contracts/`: Required for approved implementation contracts.
- `research.md`: Not required; no external product or technology research is
  needed to create the first foundation worker.

## Complexity Exceptions

None.
