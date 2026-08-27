# Change Request Specification: Frontend Feature Architecture Refactor

**Work Item ID**: `005-frontend-feature-architecture-refactor`
**Work Type**: `change-request`
**Created**: `2026-08-27`
**Status**: Complete
**Input**: Refactor the current React frontend into a production-oriented, feature-based structure without changing user-facing behavior, API contracts, or dependencies.

## Canonical Baseline

- **Current accepted behavior**: The React/Vite learner application supports the documented public catalogue, lesson study, browser-only writing canvas, and authentication routes.
- **Canonical sources**: `specs/003-frontend-learner-shell/spec.md`, `specs/001-chinese-learning-mvp/spec.md`, `specs/001-chinese-learning-mvp/contracts/api-v1.md`, and `../frontend/README.md`.
- **Current consumers/dependencies**: Browser users of `/`, `/levels`, `/levels/:slug`, `/units/:slug`, `/lessons/:slug`, `/login`, and `/register`; the NestJS API contract; existing unit and Playwright tests.

## Requested Delta

| Area | Before | After | Reason |
| --- | --- | --- | --- |
| Application composition | `src/app/SelfHanApp.tsx` owns routing, layout, shared states, all curriculum UI, writing interaction, and auth form composition. | App router and layout compose focused feature pages and shared primitives. | Restore single-responsibility boundaries and make routes easier to extend and test. |
| API ownership | `src/api/client.ts` combines HTTP transport, auth, curriculum endpoints, and all domain types. | Shared HTTP transport is separate from feature-local endpoint adapters and types. | Prevent domain coupling and keep the API layer independent of UI. |
| Feature boundaries | Auth is partially isolated; curriculum UI is application-internal. | Auth and curriculum expose only deliberate public APIs through feature entry points. | Permit feature-local change without deep imports or circular imports. |
| Import ergonomics | Relative imports span app, API, and feature boundaries. | `@/` aliases describe app, feature, and shared ownership. | Improve traceability and reduce brittle path traversal. |

## Acceptance Changes

1. Given an existing supported route and API response, when the refactored app renders it, then it has the same user-visible content, loading, empty, error, auth, and writing-canvas behavior as before.
2. Given a feature needs shared HTTP, UI state, or utility behavior, when it imports that dependency, then it uses a shared public module; no shared module imports a feature implementation.
3. Given a new curriculum or auth capability is added, when application composition needs it, then it imports the feature public API rather than an internal feature path.
4. Given the supported quality scripts, when they run after the refactor, then formatting, linting, strict type checking, unit coverage, production build, and E2E/accessibility checks pass.

## Impact Analysis

- **Public API/types/events**: None. Browser routes and backend request/response shapes remain unchanged.
- **Database/data semantics**: None.
- **Business rules/security**: None. Access token remains memory-only, refresh remains cookie-based, and backend authorization remains authoritative.
- **UI/UX/user flow**: None intended. The existing visual styles and accessible controls remain unchanged.
- **Cross-repository consumers**: Backend contract consumer paths remain unchanged.
- **Operations/release**: Vite alias configuration changes internally; build and deployment behavior remain unchanged.

## Compatibility And Rollout

- **Backward compatibility**: Compatible. Existing browser URLs, API requests, response mappings, and environment variable name remain stable.
- **Migration**: None for users or backend services.
- **Rollout/rollback**: Ship as a normal frontend build; revert the refactor commit if a regression is found.
- **Deprecation**: Remove superseded internal modules only after their imports are migrated in the same change.

## Verification

- **Changed acceptance behavior**: Existing route, auth, error, empty-state, accessibility, and canvas tests continue to pass after imports and component ownership change.
- **Compatibility behavior**: `npm run verify` in `../frontend`, plus review of route and API-path constants.
- **Migration/rollout checks**: N/A; no persisted-data or service migration.

## Docs Impact

- **Precheck result**: Not impacted.
- **Canonical docs checked**: `specs/003-frontend-learner-shell/spec.md`, `specs/001-chinese-learning-mvp/spec.md`, `specs/001-chinese-learning-mvp/contracts/api-v1.md`, and `../frontend/README.md`.
- **Canonical docs to update**: None. These documents define routes, API behavior, security posture, and user outcomes, all of which remain stable.
- **Required verifier/owner**: Reviewer-QA.

## Out Of Scope

- New routes, user flows, backend endpoints, API response changes, state-management libraries, query libraries, form libraries, or styling frameworks.
- Changes to visual design, copy, accessibility semantics, token storage, authorization, caching policy, or business rules.
- Rewriting global CSS into a new styling system.

## Traceability

| Delta | Implementation Area | Verification | Canonical Doc |
| --- | --- | --- | --- |
| CR-001 | `../frontend/src/app`, `../frontend/src/features`, and `../frontend/src/shared` | Unit, accessibility, E2E, lint, typecheck, and build | `specs/003-frontend-learner-shell/spec.md` |
| CR-002 | `../frontend/vite.config.ts` and `../frontend/tsconfig.app.json` | Typecheck and build | `../frontend/README.md` |
