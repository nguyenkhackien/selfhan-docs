# Clarifications: HSK Learner UI And Writing Practice

## Accepted Decisions

- Priority is HSK learner UI and writing practice before quiz, progress, admin,
  SRS, and dashboard work.
- Vocabulary scope is the imported HSK 1–6 and combined HSK 7–9 snapshot.
- The writing unit is each character in the selected word's display order.
- The three required modes are guidance, background character, and white paper.
- Writing remains browser-only practice: no recognition, score, or saved
  strokes.
- Stroke data must be locally bundled and may not fall back to a remote CDN.

## Assumptions

- A read-only HSK browse API is public like the existing curriculum catalogue.
- Search is limited to simplified Chinese, Pinyin, Hán–Việt, and Vietnamese
  senses, and pagination uses a small bounded page size.
- Existing vocabulary attribution remains visible in the footer; a separate
  local stroke-data notice records the HanziWriter and character-data license.
