# Research: HSK Vocabulary Data Foundation

**Work Item ID**: `006-hsk-vocabulary-data-foundation`
**Date**: `2026-08-27`

## Audited Sources

| Source                                  | Pinned Revision                                                   | Purpose                                                                   | License         | Integration Decision                                                  |
| --------------------------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------------- | --------------- | --------------------------------------------------------------------- |
| `drkameleon/complete-hsk-vocabulary`    | `7ac65bf1a6387d35f1ade478906172a19311c7f9`                        | HSK 3.0 word order, simplified/traditional forms, pinyin, and level bands | MIT             | Import only `wordlists/exclusive/new/1.json` through `7.json`.        |
| `ph0ngp/CVDICT`                         | `c379d909e308343a247e51619f7839a2060a271c`                        | Vietnamese dictionary senses                                              | CC BY-SA 4.0    | Parse locally and preserve all senses for a matching simplified word. |
| `binhbuithithanh/hanzi-sino-vietnamese` | `49e5d28ce44ae737bcfc8c1de8f1b26cfdf6dbbd`                        | Curated Hán–Việt readings and learning metadata                           | CC BY 4.0       | Use `sinoViet` where a character exists in the source.                |
| Unicode Unihan                          | Unicode current release at import time, recorded in lock metadata | Hán–Việt fallback per character                                           | Unicode License | Use only `kVietnamese`; do not treat it as a Vietnamese definition.   |

## Schema Findings

- The selected HSK files contain 506, 750, 953, 972, 1,059, 1,123, and 5,606 exclusive entries for bands 1 through 7 respectively: 10,969 total entries.
- The source describes band 7 as HSK 7–9 rather than distinct 7, 8, and 9 lists.
- CVDICT uses CEDICT-style rows: traditional form, simplified form, numeric pinyin, and one or more slash-delimited Vietnamese senses.
- The pinned CVDICT snapshot matches 10,941 HSK simplified forms (99.74%). The remaining 28 records must be retained with `needs_review`; the importer must not synthesize a definition.
- A simplified-form lookup can return more than one CVDICT row. The pinned input has 859 such ambiguous lookups, so the importer must retain all candidate senses and their source order.

## License And Attribution Plan

- Keep source-code licensing separate from the imported vocabulary dataset.
- Add `DATA_LICENSE.md` beside the generated dataset. It names all sources, immutable revisions, license URLs, required attribution, modifications, and the derivative-data CC BY-SA 4.0 notice.
- Add a visible footer attribution with links to CVDICT, CC-CEDICT, and CC BY-SA 4.0.
- Generated export metadata is deferred until an export feature exists; any future exporter must preserve this attribution metadata.

## Rejected Alternatives

- External machine translation: rejected because the accepted design is offline at runtime and source translations are available through CVDICT.
- Treating Hán–Việt readings as Vietnamese definitions: rejected because pronunciation/reading does not express the word's Vietnamese meaning.
- Fetching vocabulary from source repositories in the learner app: rejected because it weakens reproducibility, privacy, availability, and license control.
