# Implementation Plan: HSK Learner UI And Writing Practice

**Work Item ID**: 007-hsk-learner-writing
**Date**: 2026-08-27

## Architecture

The backend exposes a new public read-only HSK query boundary. It reads the
already imported tables through an HSK-owned repository and use cases; it does
not reuse curriculum repositories or change curriculum responses.

The frontend owns a new HSK feature. It uses its own API client, types, routes,
pages, and writing components. Local HanziWriter assets are served from the
application origin through a custom character-data loader, preventing the
library default CDN behaviour.

~~~text
HSK tables
  → HSK repository and use cases
  → public HSK API
  → frontend HSK feature
  → local HanziWriter character JSON
  → three writing modes
~~~

## Persona Lens Notes

- Architect: Keep HSK queries in the HSK module instead of extending
  curriculum, because imported HSK data is not administrator-published lesson
  content.
- Architect: Use a local asset loader with selected character JSON files over
  a CDN or all-character bundle; the risk is build-size growth, mitigated by
  extracting only characters present in the imported snapshot.
- Planner: Build and test the public read model before frontend routing, then
  add local stroke assets before wiring HanziWriter so offline behavior is
  testable.

## File-Level Impact

| Area | Action | Responsibility |
| --- | --- | --- |
| Backend HSK application, persistence, presentation | Add | Read-only bands/list/detail contract, validation, use cases, repository, and tests. |
| Backend HSK module | Modify | Register controller and providers. |
| Backend README | Modify | Document new public endpoints. |
| Frontend HSK feature | Add | API, types, pages, browse components, writing panel, and tests. |
| Frontend router/layout | Modify | Add HSK deep links and navigation entry. |
| Frontend public HSK stroke assets | Add | Pinned selected character JSON, asset lock, and Arphic notice. |
| Frontend package and scripts | Modify | Add HanziWriter and deterministic asset builder. |
| Canonical docs | Modify | Synchronize API, data model, tasks, operations, and attribution. |

## API Contract

- GET /hsk/bands returns all display bands and imported vocabulary counts.
- GET /hsk/vocabulary accepts band, query, cursor, and limit; it returns items
  ordered by band source order plus an optional next cursor.
- GET /hsk/vocabulary/:id returns a word, all ordered senses, and source status.

All three endpoints are public and read-only. Query input is bounded and
whitelisted. No migration, write, background job, external request, or
learner-state side effect is introduced.

## Test Strategy

- Backend unit tests cover cursor parsing, filter mapping, response shaping,
  empty results, and detail-not-found behavior.
- Backend integration-style controller tests cover validated query input and
  public routing.
- Frontend tests cover browse state, word detail, character ordering, all
  three modes, unavailable local stroke data, and attribution text.
- Run backend and frontend format, lint, typecheck, tests, build, and targeted
  accessibility checks. Manually verify the writing panel at narrow and wide
  widths because pointer drawing cannot be fully asserted in jsdom.

## Docs Impact

- **Precheck result**: Impacted.
- **Canonical docs**: MVP API contract, data model, tasks, backend README,
  frontend README, existing data license, and new stroke-data notice.
- **Project-wide traceability**: Update MVP tasks to link the HSK learner
  feature without marking quiz, progress, admin, or SRS tasks complete.
