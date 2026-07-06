---
status: Living
updated_at: "2026-07-06"
---

# Domain Context — sdd-emb

## Glossary

<!-- Seeded from docs/domain/embroidery/*.md (research pass, 2026-07-06) rather than an interactive
     per-term dialogue — batch-authored as content prep per the project owner's instruction. Treat
     each definition as a first draft: correct or tighten any entry that doesn't match how the term
     is actually used once real embroidery features are specified against it. -->

- appliqué — attaching a separate piece of fabric to the base garment as part of a design, digitized as a placement → fabric-placement → tack-down → trim → cover-stitch sequence. NOT a stitch type itself — it's a multi-step technique built from running and satin stitches.
- color block — a contiguous run of stitches in one thread between two color changes.
- color stop / color change — a marker in a stitch file indicating the machine should advance to a different thread/needle. NOT a guarantee of a specific resolved color — most file formats store color-change markers only, not thread names or RGB values.
- digitizing — the process of converting artwork or a design brief into a machine-readable stitch file (stitch types, paths, order, underlay, density). NOT vector-tracing alone — digitizing produces an executable stitch sequence, not just an outline.
- fill stitch (tatami) — rows of running stitches laid offset like bricks to cover a broad area without a single weak seam line running through it. NOT the same as a motif fill, which repeats a decorative shape instead of plain parallel rows.
- hoop — the frame that holds fabric taut during stitching. Nominal sizes (4x4, 5x7, 6x10 in, etc.) are industry-conventional, not standardized to one millimeter figure. NOT the same as the usable stitching field, which is always somewhat smaller than the nominal hoop size.
- jump stitch — a needle-up travel move between two stitch points with no thread caught in the fabric. NOT the same as a trim — a jump can occur without the thread being cut.
- motif fill — a fill technique that repeats a small decorative shape along a path or across an area for ornamental texture. NOT plain tatami fill.
- multi-head machine — an industrial machine where every head stitches the same design in parallel (one per garment/item) in a single run. NOT the same as a multi-needle machine, which describes needle count within a single head, not head count.
- pull compensation — deliberately oversizing a digitized shape to counteract the fabric distortion ("pull") that dense stitching causes, so the finished stitch-out matches the intended size. NOT the same as underlay, though both exist to manage stitching-induced distortion.
- running stitch — a single stitch line used for outlines, light detail, or as a travel connector between design areas. NOT the same as a satin or fill stitch.
- satin stitch (column stitch) — closely spaced zigzag stitches laid back and forth across a defined width, producing a bold, raised line; used for borders, lettering, and covering appliqué edges. NOT suited to very wide areas — those are conventionally re-digitized as fill instead.
- sequin — a decorative disc attached via a machine's sequin-feeder hardware, encoded in some file formats (notably DST) as its own stitch-file command. NOT supported natively by every format — most fall back to a jump/stitch/drop contingency when exporting.
- stabilizer — a backing material hooped or placed behind fabric during stitching to prevent distortion. Type (cutaway / tearaway / water-soluble / adhesive) is chosen by fabric type and whether residue may remain visible. NOT part of the digitized design itself — it's a physical production input, not a stitch-file property.
- stitch — a single thread pass between two needle-penetration points; the atomic unit of an embroidery design. NOT a "stitch type" — satin/fill/running are stitch *types*, not stitches themselves.
- stitch density — how tightly stitches are packed within a satin or fill area, measured as row spacing or stitches per unit length/area. NOT the same as stitch count (the total number of stitches in a whole design).
- trim — a command that cuts the thread at the current position, typically emitted before a long jump or a color change. NOT always distinct from a color change — the two are frequently the same underlying machine command, distinguished by whether the target needle/color slot differs from the current one.
- underlay — a low-density foundation stitch layer sewn before the visible top stitching, to stabilize fabric and improve top-stitch coverage. NOT the top/visible stitching itself.
