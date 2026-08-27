# Implementation Plan: Frontend Feature Architecture Refactor

**Work Item ID**: `005-frontend-feature-architecture-refactor`
**Date**: `2026-08-27`
**Spec**: `spec.md`

## Summary

Split the 853-line application module into app composition, feature-owned route pages and components, and genuinely shared infrastructure. Preserve the current global CSS and all externally observable behavior. Add the existing Vite/TypeScript `@/` alias rather than installing a dependency.

## Technical Context

- **Language/Version**: TypeScript 5.9 with React 19 and Vite 8.
- **Primary Dependencies**: `react-router`, `lucide-react`.
- **Storage**: Access token in module memory; refresh token in an HTTP-only backend cookie.
- **Testing**: Vitest, React Testing Library, axe-core, and Playwright.
- **Target Platform**: Modern browser SPA at desktop and mobile widths.
- **Project Type**: React frontend.
- **Performance Goals**: No additional requests, rerender loops, or bundle dependencies.
- **Constraints**: Preserve routes, API paths, user-visible behavior, current CSS, and the existing dependency set.
- **Scale/Scope**: Two current business capabilities: auth and curriculum.

## Constitution Check

- [x] Specification first
- [x] Clarification gate completed; the request explicitly excludes behavior changes and repository evidence identifies the existing contracts.
- [x] Material persona lens decisions recorded where they affect the plan
- [x] Requirements are traceable
- [x] Simplicity preserved
- [x] Testing strategy defined
- [x] Existing patterns followed
- [x] Review and convergence planned

## Persona Lens Notes

- **Architect**: Choose two feature roots, `auth` and `curriculum`, plus a small `shared` layer over a generic cross-feature domain because only those two domain boundaries are present. The risk is unnecessary folder proliferation; mitigation is to create directories only for concrete current responsibilities and leave styling global.
- **Architect**: Keep React state and the existing `useRemote` behavior rather than introduce TanStack Query or a form/state library. The server-state surface is small and no cache semantics are required by the accepted behavior.
- **Planner**: Establish shared HTTP/types and app composition before moving route pages, then run the complete existing suite after each cohesive migration. This prevents partial imports from hiding a behavioral regression.

## Architecture And Data Flow

`main.tsx` loads global styles and the app entry. `app/App.tsx` owns providers and the error boundary; `app/router` owns route composition; `layouts` owns the shell and navigation. The router imports each feature only from its `index.ts` public boundary.

`shared/api/httpClient.ts` owns base URL configuration, credentials, bearer header attachment, response parsing, and error normalization. `features/auth/api` and `features/curriculum/api` own their endpoint paths, request payloads, response types, and mappings. Feature hooks own session restoration and individual remote-resource lifecycle. Presentational states and small navigation primitives sit in `shared/components` only because multiple features consume them.

Dependency direction is `app and layouts` → `features` → `shared`. Features must not import one another's internals, and shared code must not import features.

## Project Structure

### Source Code

```text
../frontend/src/
  app/
    App.tsx
    providers/AppProviders.tsx
    router/AppRouter.tsx
  features/
    auth/
      api/
      hooks/
      pages/
      types/
      index.ts
    curriculum/
      api/
      components/
      hooks/
      pages/
      types/
      index.ts
  layouts/
    AppLayout.tsx
  shared/
    api/httpClient.ts
    components/
    hooks/
  styles/global.css
```

## File-Level Impact

| File | Action | Responsibility |
| --- | --- | --- |
| `../frontend/src/app/SelfHanApp.tsx` | Delete after migration | Superseded monolithic composition. |
| `../frontend/src/api/client.ts` | Delete after migration | Superseded combined transport/domain client. |
| `../frontend/src/app/App.tsx` | Create | Root app composition and error boundary. |
| `../frontend/src/app/providers/AppProviders.tsx` | Create | Browser router provider. |
| `../frontend/src/app/router/AppRouter.tsx` | Create | Central route declarations. |
| `../frontend/src/layouts/AppLayout.tsx` | Create | Header, mobile navigation, main landmark, and footer. |
| `../frontend/src/shared/api/httpClient.ts` | Create | Environment-derived HTTP transport and normalized errors. |
| `../frontend/src/shared/components/*` | Create | Reused loading, error, empty, page frame, and back-link UI. |
| `../frontend/src/shared/hooks/useRemoteResource.ts` | Create | Reused request lifecycle and retry state. |
| `../frontend/src/features/auth/*` | Move and split | Auth endpoint adapter, types, hook, page, and public API. |
| `../frontend/src/features/curriculum/*` | Create | Curriculum endpoint adapter, types, route pages, learning components, and public API. |
| `../frontend/src/main.tsx`, `../frontend/src/App.tsx` | Modify | Use application entry and relocated global stylesheet. |
| `../frontend/src/styles.css` | Move | Preserve the existing styles under `styles/global.css`. |
| `../frontend/vite.config.ts`, `../frontend/tsconfig.app.json` | Modify | Configure aligned `@/` aliases. |
| `../frontend/src/**/*.test.tsx` | Move or modify | Follow moved modules and retain behavior coverage. |

## Testing Approach

- Run the current focused API, auth hook, application-flow, and accessibility tests before and after migration.
- Verify every documented route still renders expected API content and all loading, empty, error, auth, and canvas branches remain covered.
- Run `npm run verify` from `../frontend`, including format, lint, strict typecheck, coverage, build, and Playwright/axe E2E checks.

## Cross-Cutting Concerns

| Concern | Decision Or N/A | Verification |
| --- | --- | --- |
| State, cache, and invalidation | Retain local hook state and no cache; current behavior has no shared server-state requirement. | Existing flow tests and no additional fetches in review. |
| Validation, errors, retry, and offline | Keep HTTP error normalization and retryable resource state; keep auth inline errors. | Client, flow, and error-boundary tests. |
| Security, privacy, auth, and permissions | Retain memory-only access token, credentialed refresh, and server-owned authorization. | Client/auth tests and source review. |
| Accessibility | Preserve semantic landmarks, labels, skip link, dialog labels, and canvas label. | axe unit test and Playwright suite. |
| Performance and reliability | No new dependency, request, cache, or memoization layer. | Build output and focused code review. |
| Compatibility, migration, and release | Stable routes, endpoint paths, and environment variable. | E2E suite, README/doc check, and build. |

## Docs Impact Plan

- **Precheck result**: Not impacted.
- **Canonical docs checked**: `specs/003-frontend-learner-shell/spec.md`, `specs/001-chinese-learning-mvp/spec.md`, `specs/001-chinese-learning-mvp/contracts/api-v1.md`, and `../frontend/README.md`.
- **Canonical docs to sync**: None.
- **Project-wide traceability**: `specs/001-chinese-learning-mvp/tasks.md` is not impacted because its requirement-to-test mappings and behavior remain unchanged.
- **Sync task**: T009.
- **Required verifier/owner**: Reviewer-QA.
- **Completion evidence**: Record the actual-diff no-impact conclusion in `review.md` and `converge.md`.

## Risks And Mitigations

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Moving components changes a route or accessibility relationship. | User-flow regression. | Preserve existing JSX and run current flow, axe, and Playwright coverage. |
| Split API adapters alter request behavior. | Auth/content failure. | Extract transport first; retain endpoint strings and add adapter tests. |
| Alias mismatch between Vite and TypeScript. | Build or test failure. | Configure both together and run typecheck/build immediately. |
| Existing uncommitted frontend work is overwritten. | Loss of user work. | Treat the current worktree as baseline and modify only the modules covered by this approved refactor. |

## Supporting Docs

- `research.md`: Not required; the repository contains the current source, tests, and contract evidence.
- `data-model.md`: Not required; no API or persisted-data shape changes.
- `contracts/`: Not required; public contracts remain stable.

## Complexity Exceptions

None.
