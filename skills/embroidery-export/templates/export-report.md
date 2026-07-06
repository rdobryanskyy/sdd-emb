<!-- Template for `embroidery-export` — copied to -->
<!-- docs/embroidery/<design>/_export/export-report-<format>-<date>.md -->
---
status: Final
updated_at: "<YYYY-MM-DD>"
---

# Export report — <design> → <FORMAT>

## Pre-export validation

- **Oversized stitches split:** `<count, or "none">`
- **Hoop-fit check:** `<pass | fail — design extents vs. declared hoop>`
- **Needle-count flag:** `<n/a | "N color blocks vs. M needles — a manual re-thread will be needed">`

## Serialization

- **Library used:** `<pyembroidery | libembroidery | other — name it>` — `<version if known>`
- **Fallback path taken?** `<no | yes — no library available; a spec-only document was produced instead of a binary file>`

## Round-trip validation

| Check | Source design | Re-parsed export | Match? |
|---|---|---|---|
| Stitch count | `<n>` | `<n>` | `<yes/no>` |
| Color-stop count | `<n>` | `<n>` | `<yes/no>` |
| Extents (bounding box) | `<w x h>` | `<w x h>` | `<yes/no, tolerance used>` |

## Contract check (if applicable)

<!-- Only if docs/features/<slug>/contracts/embroidery-export.md exists for this feature. -->

- `<field/bound>` — `<pass | drift, described>`

## Domain-doc facts relied upon

<!-- Every <!-- TBD: verify --> marked fact this export leaned on, named explicitly so it isn't
     silently trusted as verified. -->
