# Convergence Check: HSK Vocabulary Data Foundation

**Work Item ID**: `006-hsk-vocabulary-data-foundation`
**Date**: `2026-08-27`

## Requirement Coverage

| Requirement | Implemented? | Evidence                                                   | Notes                                                                                     |
| ----------- | ------------ | ---------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| FR-001      | Yes          | Source lock, deterministic builder, and generated snapshot | All inputs are local and checksum-verified before build output.                           |
| FR-002      | Yes          | Snapshot Jest test and generated data                      | 10,969 entries preserve source order; band 7 displays as HSK 7–9.                         |
| FR-003      | Yes          | Builder fixture tests and review report                    | All CVDICT senses are retained; missing data is needs_review.                             |
| FR-004      | Yes          | Add-only migration, entities, import CLI, and live import  | Two imports produced zero duplicate band/word keys.                                       |
| FR-005      | Yes          | Source lock and DATA_LICENSE.md                            | Licenses, source revisions, checksums, modifications, and export obligation are recorded. |
| FR-006      | Yes          | AppLayout footer and RTL/axe suite                         | CVDICT, CC-CEDICT, and CC BY-SA 4.0 are visible semantic links.                           |
| NFR-001     | Yes          | No controller/source client added                          | Browser and learner API paths make no runtime source request.                             |
| NFR-002     | Yes          | Parser validation, checksum verification, review report    | Malformed source fails build; missing facts are not synthesized.                          |
| NFR-003     | Yes          | Constraints and batched transactions                       | Import processes chunks of 250 in transactions.                                           |
| NFR-004     | Yes          | Locked revisions and SHA-256 checks                        | Source cache is ignored and generated output is versioned.                                |

## Acceptance Scenario Coverage

| Scenario                                        | Covered By                            | Result |
| ----------------------------------------------- | ------------------------------------- | ------ |
| Build locked HSK bands with source order        | Snapshot test and hsk:build output    | Pass   |
| Preserve multiple CVDICT senses                 | Parser fixture test                   | Pass   |
| Mark unmatched or incomplete records for review | Parser fixture test and review report | Pass   |
| Display attribution in the footer               | App RTL test and axe suite            | Pass   |
| Import repeatedly without duplicates            | Two local imports and count query     | Pass   |

## Verification Commands

| Command                                                                                                  | Result | Notes                                                                               |
| -------------------------------------------------------------------------------------------------------- | ------ | ----------------------------------------------------------------------------------- |
| `npm run hsk:build`                                                                                      | Pass   | Built 10,969 entries: 8,406 ready and 2,563 needs review.                           |
| `npm test` in backend                                                                                    | Pass   | 15 suites, 48 tests.                                                                |
| `npm run format:check`, `npm run lint`, `npm run typecheck`, and `npm run build` in backend              | Pass   | Final static and build checks pass.                                                 |
| `npm run migration:status` with local environment                                                        | Pass   | Both migrations are applied.                                                        |
| `npm run hsk:import` twice                                                                               | Pass   | Each run imported 10,969 vocabulary entries and 2,191 character readings.           |
| Local TypeORM count query                                                                                | Pass   | 10,969 vocabulary rows, 32,329 senses, 2,191 readings, 0 duplicate vocabulary keys. |
| `npm run format:check`, `npm run lint`, `npm run typecheck`, `npm test`, and `npm run build` in frontend | Pass   | 7 suites, 24 tests; production build succeeds.                                      |

## Docs And Code Alignment

- The source MVP contract now distinguishes the imported HSK data foundation
  from administrator-published curriculum and does not claim a new learner API.
- Backend operations and legal provenance are documented with the local
  snapshot. The frontend README explains the attribution and deferred HSK UI.

## Canonical Docs State

- **Synced**: Verified by Reviewer-QA in
  `review.md`. Updated
  `specs/001-chinese-learning-mvp/spec.md`,
  `specs/001-chinese-learning-mvp/data-model.md`,
  `specs/001-chinese-learning-mvp/tasks.md`,
  `../backend/README.md`, `../backend/data/hsk/DATA_LICENSE.md`, and
  `../frontend/README.md`. Checked
  `specs/001-chinese-learning-mvp/contracts/api-v1.md` needs no change
  because no public route was added.

**DOCS_IMPACT_FINAL_CHECK evidence**: Reviewed the actual backend schema,
snapshot, import command, footer, tests, and generated review report against
the work-item specification and all impacted canonical docs.

**Project-wide traceability**: Synced in
`specs/001-chinese-learning-mvp/tasks.md`.

## Persona Lens Closure

| Applied Lens | Decision Artifact            | Closure Evidence                                                           |
| ------------ | ---------------------------- | -------------------------------------------------------------------------- |
| Architect    | `plan.md`                    | Dedicated HSK tables leave current curriculum and API contracts unchanged. |
| Planner      | `tasks.md`                   | Locked build and tests precede import; verification and docs are recorded. |
| Engineer     | Backend and frontend changes | Local build/import runs, constraints, and footer attribution are complete. |
| Reviewer-QA  | `review.md`                  | R-001 fixed and approval recorded.                                         |

## Remaining Work

- Content review tooling and manual resolution of the 2,563 needs-review
  entries.
- HSK learner browsing, export metadata, writing modes, quizzes, and progress
  remain deliberately out of scope for this data-foundation work item.

## Final Status

Converged.
