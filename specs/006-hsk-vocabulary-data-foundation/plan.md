# Implementation Plan: HSK Vocabulary Data Foundation

**Work Item ID**: `006-hsk-vocabulary-data-foundation`
**Date**: `2026-08-27`
**Spec**: `spec.md`

## Summary

Create a reproducible offline data pipeline that converts pinned HSK, CVDICT, curated Hán–Việt, and Unihan files into a normalized licensed dataset and idempotently imports it into new backend tables. Keep current curriculum content separate. Add the required visible attribution, but defer HSK learner UI and writing modes.

## Technical Context

- **Language/Version**: TypeScript 5.7, Node 22+, NestJS 11, React 19.
- **Primary Dependencies**: Existing TypeORM, PostgreSQL, Vitest, Jest, and React Testing Library; no new runtime dependency.
- **Storage**: Versioned normalized JSON, source lock metadata, and PostgreSQL tables.
- **Testing**: Jest parser/import unit tests, existing backend checks, frontend verification.
- **Target Platform**: Backend CLI import and browser footer.
- **Project Type**: Cross-repository data foundation.
- **Performance Goals**: Process 10,969 entries without an external runtime request; batched database upserts inside one bounded transaction.
- **Constraints**: Preserve source attribution, licensing, source order, multiple senses, and unmatched records; do not change existing learner endpoints.
- **Scale/Scope**: HSK 1–6 and combined 7–9 only.

## Constitution Check

- [x] Specification first
- [x] Clarification gate completed
- [x] Material persona lens decisions recorded where they affect the plan
- [x] Requirements are traceable
- [x] Simplicity preserved
- [x] Testing strategy defined
- [x] Existing patterns followed
- [x] Review and convergence planned

## Persona Lens Notes

- **Architect**: Keep source HSK data in dedicated tables rather than creating published `Level`, `Unit`, `Lesson`, and `Vocabulary` rows. This preserves import provenance and avoids asserting curriculum sequencing that the flat source does not provide.
- **Architect**: Commit a normalized HSK slice and a lockfile, not full upstream repositories. This keeps the app offline at runtime and gives reviewers a stable, license-aware snapshot.
- **Planner**: Validate normalization before migration/seed work; the database import and frontend attribution proceed only after source count, parsing, and license fixtures are stable.

## Architecture And Data Flow

```text
Pinned upstream snapshots
  → source-lock verification
  → pure parser/normalizer
  → versioned local HSK JSON + review report + DATA_LICENSE.md
  → idempotent transactional backend import
  → future HSK API and learner UI
```

The data builder is a maintainer command, never a request-path dependency. CVDICT rows are keyed by simplified word and retain all matching senses. Character readings prefer the curated source and fall back to Unihan `kVietnamese`. No match is turned into a generated answer.

## Project Structure

### Source Code

```text
../backend/
  data/hsk/
    hsk-3.0-vi.json
    sources.lock.json
    DATA_LICENSE.md
  src/modules/hsk-data/
    application/
    domain/
    infrastructure/
  src/infrastructure/database/migrations/
  src/infrastructure/database/import-hsk.ts
  scripts/build-hsk-dataset.ts

../frontend/src/layouts/AppLayout.tsx
```

## File-Level Impact

| File                                                   | Action | Responsibility                                                   |
| ------------------------------------------------------ | ------ | ---------------------------------------------------------------- |
| `../backend/data/hsk/sources.lock.json`                | Create | Pinned source revisions, URLs, checksums, and licenses.          |
| `../backend/data/hsk/hsk-3.0-vi.json`                  | Create | Generated normalized HSK snapshot.                               |
| `../backend/data/hsk/DATA_LICENSE.md`                  | Create | Attribution, modifications, and ShareAlike requirements.         |
| `../backend/scripts/build-hsk-dataset.ts`              | Create | Source parser, validation, normalization, and report generation. |
| `../backend/src/modules/hsk-data/*`                    | Create | Data types and import service.                                   |
| `../backend/src/infrastructure/database/migrations/*`  | Create | HSK source tables and constraints.                               |
| `../backend/src/infrastructure/database/import-hsk.ts` | Create | Explicit idempotent transactional data import command.           |
| `../backend/package.json`                              | Modify | Add data build/import scripts only.                              |
| `../frontend/src/layouts/AppLayout.tsx`                | Modify | Display accessible data attribution.                             |
| `../frontend/src/app/*.test.tsx`                       | Modify | Cover footer attribution.                                        |
| Canonical specs and READMEs                            | Modify | Document data scope, operation, attribution, and traceability.   |

## Testing Approach

- Unit-test CVDICT parsing, source-lock validation, multiple-sense retention, unmatched handling, source-order preservation, and Hán–Việt fallback.
- Test database import idempotency and database constraints using the existing backend test approach; run migration status/integration check if a disposable database is available.
- Add frontend footer attribution test and run existing `npm run verify`.
- Run backend format, lint, typecheck, unit tests, build, and integration tests.

## Cross-Cutting Concerns

| Concern                                  | Decision Or N/A                                                                              | Verification                            |
| ---------------------------------------- | -------------------------------------------------------------------------------------------- | --------------------------------------- |
| State, cache, and invalidation           | N/A; no learner API is added.                                                                | No new controller or request path.      |
| Validation, errors, retry, and offline   | Source locks/checksums validate before writes; malformed or unmatched rows produce a report. | Parser tests.                           |
| Security, privacy, auth, and permissions | No secret and no external call during app runtime; import remains an operator command.       | Source/config review.                   |
| Accessibility                            | Footer attribution is semantic text with accessible links.                                   | RTL and axe coverage.                   |
| Performance and reliability              | Local generated file and batched transaction; no per-word source network calls.              | Import test and source review.          |
| Compatibility, migration, and release    | Add-only tables and scripts; current API routes remain stable.                               | Migration check and existing API tests. |

## Docs Impact Plan

- **Precheck result**: Impacted.
- **Canonical docs checked**: `specs/001-chinese-learning-mvp/spec.md`, `specs/001-chinese-learning-mvp/data-model.md`, `specs/001-chinese-learning-mvp/contracts/api-v1.md`, `specs/001-chinese-learning-mvp/tasks.md`, `../backend/README.md`, and `../frontend/README.md`.
- **Canonical docs to sync**: All checked docs except the public API contract, which is checked for no HTTP change; add `../backend/data/hsk/DATA_LICENSE.md`.
- **Project-wide traceability**: Update `specs/001-chinese-learning-mvp/tasks.md` to connect HSK data foundation to FR-014 and NFR-008 without marking quiz/admin work complete.
- **Sync task**: T009.
- **Required verifier/owner**: Reviewer-QA.
- **Completion evidence**: Source revisions, data counts, data-license review, and tests recorded in `review.md` and `converge.md`.

## Risks And Mitigations

| Risk                                            | Impact                              | Mitigation                                                                                              |
| ----------------------------------------------- | ----------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Source data changes or disappears.              | Non-reproducible import.            | Pin commits and checksums; commit normalized output.                                                    |
| CVDICT has incorrect or ambiguous translations. | Learner content quality risk.       | Preserve senses, mark source provenance, and create review status without claiming manual verification. |
| Derived-data licensing is omitted.              | Legal and community-license breach. | Versioned data license, lockfile, footer attribution, and review gate.                                  |
| Large import partially writes.                  | Data integrity failure.             | Validate first and import in a transaction with idempotent constraints.                                 |
| Hán–Việt source lacks a character.              | Misleading reading.                 | Unihan fallback; otherwise null and `needs_review`.                                                     |

## Supporting Docs

- `research.md`: Required for source audit, counts, and licensing decisions.
- `data-model.md`: Required for new table ownership and invariants.
- `contracts/`: Not required; no public HTTP contract changes.

## Complexity Exceptions

None.
