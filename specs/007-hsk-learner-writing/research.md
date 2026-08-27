# Research: HSK Learner UI And Writing Practice

## Existing Boundaries

- HSK vocabulary is imported in dedicated backend tables and is separate from
  published curriculum content.
- The current frontend is React and Vite with feature-owned API modules,
  pages, and shared remote-resource state.
- The existing browser canvas already supports pointer drawing but has no
  stroke-order data or mode model.

## Stroke Data Decision

HanziWriter source code is MIT licensed. Its character data originates from
Make Me a Hanzi and is distributed under the Arphic Public License. HanziWriter
documents a custom local character-data loader and warns that its default is a
CDN fetch. SelfHan will pin the data source, extract only characters used by
the imported HSK vocabulary, host them as local application assets, and record
the required notice.

## Rejected Alternatives

- Default HanziWriter configuration: rejected because it fetches from a CDN.
- One all-character data bundle: rejected because the documented bundle is
  materially larger than individual local character files.
- Font-based fake animation: rejected because it cannot provide true stroke
  order.
