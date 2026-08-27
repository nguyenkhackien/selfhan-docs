# Data Model: HSK Vocabulary Data Foundation

**Work Item ID**: `006-hsk-vocabulary-data-foundation`
**Status**: Approved

## Ownership

The existing `vocabulary` table remains lesson-owned content. HSK source data is stored separately so an import can be repeated, audited, reviewed, and later mapped into lessons without turning all imported records into published curriculum.

## Entities

### HskVocabulary

One imported HSK word in the source's exact order and band.

| Field            | Type             | Rules                                                     |
| ---------------- | ---------------- | --------------------------------------------------------- |
| `id`             | UUID             | Primary key.                                              |
| `hskBand`        | small integer    | `1` through `7`; band `7` is displayed as `HSK 7–9`.      |
| `sourceOrder`    | integer          | Positive and unique within one band.                      |
| `simplified`     | text             | Required source word.                                     |
| `traditional`    | text nullable    | Source form when present.                                 |
| `pinyin`         | text             | Required pinyin with tone marks.                          |
| `frequency`      | integer nullable | Source frequency when present.                            |
| `sinoViet`       | text nullable    | Character readings in word order; no value is fabricated. |
| `importStatus`   | enum             | `ready` or `needs_review`.                                |
| `sourceRevision` | text             | Immutable HSK source revision.                            |
| timestamps       | timestamptz      | Import lifecycle audit.                                   |

Unique constraint: `hskBand + simplified`.

### HskVocabularySense

An ordered Vietnamese definition candidate imported from CVDICT.

| Field                 | Type    | Rules                                               |
| --------------------- | ------- | --------------------------------------------------- |
| `id`                  | UUID    | Primary key.                                        |
| `hskVocabularyId`     | UUID    | Required foreign key; cascade delete.               |
| `sourceOrder`         | integer | Positive and unique within the vocabulary word.     |
| `meaningVi`           | text    | Required source-imported Vietnamese sense.          |
| `sourceTraditional`   | text    | CVDICT traditional form for attribution and review. |
| `sourcePinyinNumeric` | text    | CVDICT numeric pinyin for review.                   |

### HskCharacterReading

One reading per character, used to build an HSK word's `sinoViet` value and support later writing practice.

| Field            | Type | Rules                                   |
| ---------------- | ---- | --------------------------------------- |
| `hanzi`          | text | Primary key; one Unicode Han character. |
| `sinoViet`       | text | Required source reading.                |
| `source`         | enum | `kai_hanzi` or `unihan`.                |
| `sourceRevision` | text | Immutable source version/revision.      |

## Import Invariants

- The importer only reads lockfile-pinned files and rejects an unexpected revision or checksum.
- Every selected HSK word is inserted or updated idempotently in a transaction.
- A word with no CVDICT match has no sense records and `needs_review`; it is never assigned a guessed definition.
- CVDICT senses remain separate rather than being concatenated or collapsed.
- Hán–Việt is constructed by traversing each word's characters from left to right; any missing character reading yields a null `sinoViet` and `needs_review`.
