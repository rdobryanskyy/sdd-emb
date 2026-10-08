<!-- Template for `api` — copied to docs/features/<slug>/contracts/embroidery-export.md ONLY when -->
<!-- the AC in the spec.md of the feature describe a machine embroidery file as output. Additive: -->
<!-- this file goes together with the primary contract kind of the declared surface (openapi.yaml, -->
<!-- public-api.md, ...). It does not replace it. Fields trace to data-model.md columns. Numeric -->
<!-- bounds trace to docs/domain/embroidery/machine-constraints.md when that reference exists. If -->
<!-- not, mark them <!-- TBD: verify -->. If the feature never writes a machine-readable stitch -->
<!-- file, delete this file. -->
---
status: Draft
owner: "<Backend Lead>"
reviewers: []
updated_at: "<YYYY-MM-DD>"
feature_size: M
---

# Embroidery file export — <feature>

This is the derived contract for the machine-format files that this feature makes. The same as the
OpenAPI contract, it is **generated from `data-model.md`** (the StitchBlock/ColorStop/Design-shaped
entities) and from the domain constraints in `docs/domain/embroidery/machine-constraints.md` and
`docs/domain/embroidery/file-formats.md`. It is never typed by hand. A field or bound without a
traceable origin gets `<!-- TBD: verify -->`, not an invented number.

## Target format(s)

<!-- One block for each format that the AC need. A feature can write more than one format. -->

### `<DST | PES | EXP | JEF | VP3 | HUS | XXX | ART | EMB>`

- **Encoding model:** relative | absolute stitch coordinates — from `docs/domain/embroidery/file-formats.md`.
- **Source entities:** `<data-model.md entity>` → the stitch-record stream of this format.
- **Header/metadata fields written:** `<design name, stitch count, color count, extents — the fields that the format supports>`.
- **Color-change / trim / jump / end command mapping:** `<how a data-model ColorStop/Jump/Trim maps to the command bytes of this format>`.

## Per-field bounds (validated before export)

| Field | Source (data-model column / domain doc) | Bound | Enforcement |
|---|---|---|---|
| stitch length | `docs/domain/embroidery/machine-constraints.md` | `<value or range — TBD if unverified>` | reject / clamp / split-into-jumps (select one, per the convention of the format) |
| jump length | `docs/domain/embroidery/machine-constraints.md` | `<value — TBD if unverified>` | forced trim above this length |
| hoop extents | the declared target hoop of the feature | `<width x height>` | reject a design that is larger than the hoop |
| color/needle count | `<data-model.md entity>` | `<the needle count of the machine, if the target is a specific machine>` | warn or reject |

## Round-trip validation (the export's self-check)

<!-- This is the export-contract equivalent of the bidirectional drift check of api. -->

- **Stitch count preserved:** when you parse the exported file again, the stitch count is the same as in the source design.
- **No stitch is longer than the maximum length of this format or machine** (see the bounds table above).
- **Color-stop count matches** the color-block count of the source design.
- **Re-import round-trip** (if a parser for this format is available, for example `pyembroidery`/`libembroidery`): export → parse again → compare the stitch coordinates within a tolerance. A mismatch is a real bug, not a rounding footnote.

## Drift check

Use the same discipline as the OpenAPI drift check of `api`. Does each field in the exported format
trace to a `data-model.md` column? Does each numeric bound trace to
`docs/domain/embroidery/machine-constraints.md` (or to an explicit `<!-- TBD -->`)? Show each field
or bound without an origin. Do not silently invent it.
