# Tasks: Frontend Learner Shell

**Work Item ID**: `003-frontend-learner-shell`
**Status**: Complete

| ID | Task | Verification | Status |
| --- | --- | --- | --- |
| T1 | Audit the existing frontend and foundation API boundary. | Compare source, backend runtime, and contract. | DONE |
| T2 | Define the API client, auth session handling, and typed content projections. | Typecheck and API-client unit tests. | DONE |
| T3 | Implement auth, catalogue, lesson, audio, and writing routes. | Component and browser flow tests. | DONE |
| T4 | Apply responsive and accessibility review. | Axe and viewport E2E checks. | DONE |
| T5 | Update documentation and parent traceability. | Markdown review and diff check. | DONE |

## Completion Evidence

- `npm run verify` passed: 18 unit tests, per-file coverage thresholds, build,
  and two Playwright/axe flows.
- The learner flow was checked at desktop and 390 px mobile width.
- The final green sidebar layout keeps the previous visual hierarchy while
  replacing all browser-only ZenLingo state with the backend API boundary.
