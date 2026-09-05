# Work Item: Backend Request Logger

**Work Item ID**: `014-backend-request-logger`
**Work Type**: `change-request`
**Status**: Complete

## Scope

- **Outcome**: Upgrade the existing NestJS JSON logger into a request-correlated logging system. Every completed HTTP request and handled failure will include a safe request ID, method, route, status, and duration. The logger will support `trace`, `debug`, `verbose`, `info`, `warn`, `error`, and `fatal`; it will render coloured, readable records in development and structured JSON in production.
- **Non-goals**: Add a third-party logging transport, change HTTP response/error contracts, persist logs, add tracing/metrics backends, log request bodies, queries, headers, cookies, tokens, or change authentication/database behavior.
- **Assumptions and risks**: The existing `x-request-id` middleware and safe error envelope remain authoritative. `AsyncLocalStorage` will make the current request ID available to nested async logs without passing it through every method. Structured log objects continue to redact sensitive keys, and failure details sent to clients remain unchanged.

## Technical Plan And Tasks

| ID | Task | Verification | Status |
| -- | ---- | ------------ | ------ |
| T1 | Add validated logging configuration and request-scoped correlation storage; upgrade the shared logger with level filtering, trace support, secret redaction, production JSON, and development ANSI colours. | Unit tests for levels, threshold, redaction, request ID propagation, and both render formats. | COMPLETED |
| T2 | Extend request-ID middleware and the global exception filter to record request completion and a single contextual failure record without logging sensitive request data. | Middleware/filter unit tests for request ID, duration, 4xx warning, 5xx error, and safe response compatibility. | COMPLETED |
| T3 | Document runtime logging configuration and run the backend quality gates, review, and convergence. | Format, lint, typecheck, unit tests, build, diff check, and documentation review. | COMPLETED |

## Architecture Decision

- **Architect**: Extend the established built-in logger rather than introduce Pino/Winston because the current application already owns its log schema and needs only local console output. A config-backed level/format policy and `AsyncLocalStorage` provide correlation without a transport dependency; the risk of lost context is mitigated by keeping the request ID explicitly on completion and exception records.

## Verification Diagnosis

- **Observed failure**: The first focused test run compiled the logger tests but rejected three strict test doubles: an overly broad exception-filter logger mock, untyped response locals on an `EventEmitter`, and an overbroad fake ConfigService key. The first fixture correction still made the development-format test read the host process environment.
- **Cause and localization**: The production logger depends only on `warn` and `error` in the exception filter, while the test constructor demanded the full class. The other failures are limited to strict TypeScript inference in the test fixtures. Nest's in-memory `ConfigService` intentionally falls back to process environment, so it is unsuitable for asserting an absent `LOG_FORMAT`.
- **ROOT_CAUSE_GATE**: Confirmed. Narrow the filter dependency to the required logger methods; explicitly type the test response locals; give the options factory a minimal configuration-reader interface and test it with an isolated implementation. Production still injects Nest's `ConfigService` through that interface.
- **Fix attempt**: 2. Passed: all five focused suites (15 tests) pass after isolating the configuration reader.

## Docs Impact

- **Precheck**: Impacted.
- **Canonical docs checked or to update**: `../backend/README.md` and `../backend/.env.example` must document the logging levels, output format, request correlation, and safe operational controls. `specs/001-chinese-learning-mvp/contracts/api-v1.md` and `specs/008-mvp-learning-admin/contracts/learning-admin-v1.md` describe the unchanged request-ID/error envelope and are expected to remain accurate.
- **DOCS_IMPACT_FINAL_CHECK**: Passed. `../backend/README.md` and `../backend/.env.example` now document level/format configuration, colour and JSON behavior, request correlation, redaction controls, and global injection. The API contracts at `specs/001-chinese-learning-mvp/contracts/api-v1.md` and `specs/008-mvp-learning-admin/contracts/learning-admin-v1.md` were checked and remain accurate because headers and error envelopes are unchanged.
- **Canonical Docs State**: Synced — verifier reviewed the actual diff and the resulting request/error behavior.

## Completion Evidence

- **Verification**: Focused logger suite: 5 suites / 15 tests passed. Final backend quality gates passed: `npm run format:check`, `npm run lint`, `npm run typecheck`, `npm test` (22 suites / 67 tests), and `npm run build`. Final `git diff --check` passed for backend and source-trust work item.
- **Technical review**: One medium finding: registering logger providers only in `AppModule` prevented feature modules from injecting the shared logger. Resolved by adding a global `LoggingModule` that exports `JsonLoggerService` and `RequestContextService`; no Critical, High, or remaining Medium findings.
- **Scoped re-review**: Passed. The new module owns the options factory and exports its dependencies; static checks, all unit tests, and production build pass after the correction.
- **Convergence**: Converged. Levels, colour/JSON formats, automatic request correlation, completion/error records, redaction, configuration validation, and documentation each map to implementation and verification evidence.
