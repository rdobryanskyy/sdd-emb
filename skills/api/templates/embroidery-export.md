<!-- Template for `api` — copied to docs/features/<slug>/contracts/embroidery-export.md ONLY when -->
<!-- the feature's spec.md AC describe producing a machine embroidery file. Additive: this rides -->
<!-- alongside whichever primary contract kind the declared surface already produces (openapi.yaml, -->
<!-- public-api.md, ...) — it does not replace it. Fields trace to data-model.md columns; numeric -->
<!-- bounds trace to docs/domain/embroidery/machine-constraints.md when that reference exists, or -->
<!-- are marked <!-- TBD: verify --> when they don't. Delete this file if the feature never emits a -->
<!-- machine-readable stitch file. -->
---
status: Draft
owner: "<Backend Lead>"
reviewers: []
updated_at: "<YYYY-MM-DD>"
feature_size: M
---

# Embroidery file export — <feature>

Derived contract for the machine-format file(s) this feature produces. Like the OpenAPI contract,
this is **generated from `data-model.md`** (the StitchBlock/ColorStop/Design-shaped entities) plus
the domain constraints in `docs/domain/embroidery/machine-constraints.md` and
`docs/domain/embroidery/file-formats.md` — never hand-typed. A field or bound with no traceable
origin gets `<!-- TBD: verify -->`, not an invented number.

## Target format(s)

<!-- One block per format the AC require. A feature may need to emit several. -->

### `<DST | PES | EXP | JEF | VP3 | HUS | XXX | ART | EMB>`

- **Encoding model:** relative | absolute stitch coordinates — from `docs/domain/embroidery/file-formats.md`.
- **Source entities:** `<data-model.md entity>` → this format's stitch-record stream.
- **Header/metadata fields written:** `<design name, stitch count, color count, extents — whichever the format supports>`.
- **Color-change / trim / jump / end command mapping:** `<how a data-model ColorStop/Jump/Trim maps to this format's command bytes>`.

## Per-field bounds (validated before export)

| Field | Source (data-model column / domain doc) | Bound | Enforcement |
|---|---|---|---|
| stitch length | `docs/domain/embroidery/machine-constraints.md` | `<value or range — TBD if unverified>` | reject / clamp / split-into-jumps (pick one, per format's own convention) |
| jump length | `docs/domain/embroidery/machine-constraints.md` | `<value — TBD if unverified>` | forced trim above this length |
| hoop extents | feature's declared target hoop | `<width x height>` | reject a design exceeding the hoop |
| color/needle count | `<data-model.md entity>` | `<machine's needle count, if a specific machine is targeted>` | warn or reject |

## Round-trip validation (the export's self-check)

<!-- This is the export contract's equivalent of api's bidirectional drift check. -->

- **Stitch count preserved:** re-parsing the exported file yields the same stitch count as the source design.
- **No stitch exceeds this format's/machine's max length** (see the bounds table above).
- **Color-stop count matches** the source design's color-block count.
- **Re-import round-trip** (if a parser for this format is available, e.g. `pyembroidery`/`libembroidery`): export → re-parse → compare stitch coordinates within a tolerance; a mismatch is a real bug, not a rounding footnote.

## Drift check

Same discipline as `api`'s OpenAPI drift check, applied here: does every field in the exported
format trace to a `data-model.md` column? Does every numeric bound trace to
`docs/domain/embroidery/machine-constraints.md` (or an explicit `<!-- TBD -->`)? A field or bound
with no origin is surfaced, not silently invented.
