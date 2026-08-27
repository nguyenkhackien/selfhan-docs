# Data Model: HSK Learner UI And Writing Practice

## API Read Models

| Model | Fields | Invariants |
| --- | --- | --- |
| HskBandSummary | band, displayBand, count | Bands are 1 through 7; 7 displays as HSK 7–9. |
| HskVocabularySummary | id, band, order, simplified, pinyin, sinoViet, primaryMeaning, importStatus | One row per imported vocabulary record. |
| HskVocabularyDetail | summary fields, traditional, frequency, senses | Senses retain the imported CVDICT order. |
| HskVocabularyPage | items, nextCursor, limit | Limit is validated and bounded; cursor order is stable. |

## Writing State

| State | Meaning |
| --- | --- |
| mode | guidance, background, or white-paper |
| characterIndex | Zero-based index in the selected word's simplified characters |
| strokeDataState | loading, ready, or unavailable for the active character |

The drawing buffer is transient browser state. Changing character or mode
clears only the local drawing; no request stores it.
