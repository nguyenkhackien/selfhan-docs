# Review: HSK Vocabulary Data Foundation

**Work Item ID**: `006-hsk-vocabulary-data-foundation`
**Date**: `2026-08-27`
**Reviewer**: Agent

## Scope Reviewed

- Spec: `spec.md`
- Plan: `plan.md`
- Tasks: `tasks.md`
- Code: backend HSK data builder, entities, migration, import command, and
  frontend footer attribution.
- Tests: backend parser/snapshot tests and frontend application/accessibility
  tests.
- Verification: backend and frontend quality gates, TypeORM migration, two live
  imports, and a duplicate-key count query.

## Reviewer/QA Lens Notes

- The HSK data stays separate from the published curriculum tables and adds no
  learner HTTP route. This preserves the existing contract while making later
  HSK UI/API work possible.
- Missing source information remains explicit review work. The importer never
  creates a Vietnamese meaning or Hán–Việt reading that is not in an approved
  local source.

## Findings

| ID    | Severity | Location                       | Finding                                                                                              | Recommendation                                                                                | Disposition |
| ----- | -------- | ------------------------------ | ---------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- | ----------- |
| R-001 | Medium   | `scripts/build-hsk-dataset.ts` | Unihan provenance was initially saved as the extracted-file checksum instead of the Unicode version. | Store the locked Unicode version as source revision and retain the checksum only in the lock. | Fixed       |

## Review Cycle

- **Primary review verdict**: Approved
- **Scoped re-review used**: Yes
- **Scoped re-review boundary**: R-001; rebuilt snapshot, reran parser/snapshot
  tests, lint, and typecheck after correcting the revision value.
- **Scoped re-review result**: All in scope closed.

## Test Gaps

- The importer's idempotency evidence is a local PostgreSQL smoke test rather
  than an isolated disposable-database Jest test. It ran two imports and a
  read-only count query, so this is a follow-up test-strengthening opportunity,
  not an acceptance gap.

## Security And Data Risks

- No runtime source call or translation API was added. Source caches are
  ignored by Git; generated data records locks and attribution.
- CVDICT-derived meanings and incomplete Hán–Việt coverage remain
  source-imported content. The generated review report contains 2,563
  needs-review entries and must be resolved by a later content-review workflow.

## Performance Risks

- The 7.3 MB normalized JSON is a maintainer/import artifact, not loaded in a
  browser request path. The importer processes it in batches of 250 entries.

## Cross-Cutting Concerns

| Concern                                  | Reviewed Evidence                                                                                   | Finding Or None |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------- | --------------- |
| State, cache, and invalidation           | No controller, cache, or learner request path added.                                                | None            |
| Validation, errors, retry, and offline   | SHA-256 verification, parser fixtures, review report, and local-only source cache.                  | None            |
| Security, privacy, auth, and permissions | Explicit operator CLI; no secret added to tracked files.                                            | None            |
| Accessibility                            | Semantic footer links, RTL check, and existing axe suite.                                           | None            |
| Performance, reliability, and release    | Add-only migration, database constraints, batched upsert, two import runs, and zero duplicate keys. | None            |

## Docs Drift

- None found. The MVP specification, data model, tasks, backend/frontend
  operations docs, license record, and work-item artifacts describe the
  resulting data-only scope. The API contract was checked and remains correct
  because no public route was introduced.

## Docs Impact Verification

- **Precheck result reviewed**: Impacted
- **Canonical docs checked**: `specs/001-chinese-learning-mvp/spec.md`,
  `specs/001-chinese-learning-mvp/data-model.md`,
  `specs/001-chinese-learning-mvp/tasks.md`,
  `specs/001-chinese-learning-mvp/contracts/api-v1.md`,
  `../backend/README.md`, and `../frontend/README.md`.
- **Actual diff impact**: Licensed local HSK data, persistence schema/import,
  source attribution, and frontend footer.
- **Project-wide traceability**: Synced in
  `specs/001-chinese-learning-mvp/tasks.md`.
- **Canonical docs state**: Synced
- **Verifier**: Reviewer-QA
- **Evidence or blocker**: Updated docs, source lock, data license, review
  report, tests, migration execution, and database counts; no blocker.

## Verdict

Approved.
