<!-- Template for `embroidery-digitize` — copied to docs/embroidery/<design>/stitch-plan.md. -->
<!-- The paired stitch-plan.json (same directory) is the machine-readable twin consumed by -->
<!-- embroidery-optimize / embroidery-export / embroidery-qa. Keep the two in agreement. -->
---
status: Draft
updated_at: "<YYYY-MM-DD>"
---

# Stitch plan — <design>

## Production inputs

- **Fabric:** `<fabric type>`
- **Stabilizer:** `<cutaway | tearaway | water-soluble | adhesive>` — `<why, citing production.md if used>`
- **Hoop:** `<nominal hoop size>`

## Regions

<!-- One subsection per region, in color-block sequence order. -->

### R1 — <descriptive name>

- **Stitch type:** `<satin | fill | running | motif>`
- **Underlay:** `<edge-walk | zigzag | lattice | none>` — `<reason if none>`
- **Density:** `<value + unit>` — source: `<doc:<file> | user-confirmed>`
- **Pull compensation:** `<value in mm>` — source: `<doc:<file> | user-confirmed>`
- **Color / thread block:** `<id>`

## Sequencing notes

<!-- Any hard ordering the design requires (e.g. appliqué's placement → tack-down → trim → cover),
     and the color-grouping rationale for the rest. -->

## Open gaps

<!-- Anything left <!-- N/A: reason --> or sourced from a domain doc's <!-- TBD: verify --> marker,
     so embroidery-optimize/export/qa (and the user) don't silently trust it as verified. -->
