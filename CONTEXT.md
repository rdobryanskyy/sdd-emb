---
status: Living
updated_at: "2026-07-06"
---

# Domain Context — sdd-emb

## Glossary

<!-- These terms come from docs/domain/embroidery/*.md (research pass, 2026-07-06), not from an
     interactive dialogue for each term. The project owner told us to write them as one batch of
     content preparation. Each definition is a first draft. When real embroidery features use a
     term in a different way, correct the entry or make it more precise. -->

- appliqué — a separate piece of fabric that a design attaches to the base garment. It is digitized as a placement → fabric-placement → tack-down → trim → cover-stitch sequence. NOT a stitch type — it is a multi-step technique made of running and satin stitches.
- color block — a continuous sequence of stitches in one thread between two color changes.
- color stop / color change — a marker in a stitch file that tells the machine to go to a different thread or needle. NOT a guarantee of a specific color — most file formats keep only color-change markers, not thread names or RGB values.
- digitizing — the process that changes artwork or a design brief into a machine-readable stitch file (stitch types, paths, order, underlay, density). NOT only vector tracing — digitizing makes an executable stitch sequence, not only an outline.
- fill stitch (tatami) — rows of running stitches with an offset, as in a brick wall, that cover a broad area without one weak seam line through it. NOT the same as a motif fill, which repeats a decorative shape in place of plain parallel rows.
- hoop — the frame that keeps the fabric tight during stitching. Nominal sizes (4x4, 5x7, 6x10 in, etc.) are industry conventions, not standardized to one millimeter value. NOT the same as the usable stitching field, which is always a little smaller than the nominal hoop size.
- jump stitch — a needle-up move between two stitch points without thread in the fabric. NOT the same as a trim — a jump can occur without a cut of the thread.
- motif fill — a fill technique that repeats a small decorative shape along a path or across an area, for ornamental texture. NOT plain tatami fill.
- multi-head machine — an industrial machine in which each head stitches the same design at the same time (one head for each garment or item) in one run. NOT the same as a multi-needle machine, which is about the number of needles in one head, not the number of heads.
- pull compensation — a deliberate increase of the size of a digitized shape. It compensates for the fabric distortion ("pull") that dense stitching causes, so that the finished stitch-out has the intended size. NOT the same as underlay, but both control the distortion that stitching causes.
- running stitch — a single stitch line for outlines, light detail, or travel between design areas. NOT the same as a satin or fill stitch.
- satin stitch (column stitch) — zigzag stitches with small spaces, laid from side to side across a defined width. They make a bold, raised line for borders, lettering, and the edges of appliqué. NOT applicable to very wide areas — a digitizer usually changes those areas to fill.
- sequin — a decorative disc that the sequin-feeder hardware of a machine attaches. Some file formats (for example, DST) encode it as a separate stitch-file command. NOT supported in all formats — most formats use a jump/stitch/drop alternative at export.
- stabilizer — a backing material in the hoop or behind the fabric during stitching, to prevent distortion. Select the type (cutaway / tearaway / water-soluble / adhesive) from the fabric type and from whether residue can stay visible. NOT part of the digitized design — it is a physical production input, not a property of the stitch file.
- stitch — one thread pass between two needle-penetration points. It is the smallest unit of an embroidery design. NOT a "stitch type" — satin/fill/running are stitch *types*, not stitches.
- stitch density — how near the stitches are to each other in a satin or fill area. It is measured as row spacing or as stitches per unit of length or area. NOT the same as stitch count (the total number of stitches in a full design).
- trim — a command that cuts the thread at the current position. Usually it occurs before a long jump or a color change. NOT always different from a color change — frequently the two are the same machine command. The difference is whether the target needle or color slot is different from the current one.
- underlay — a low-density foundation layer of stitches before the visible top stitching. It makes the fabric stable and gives better coverage of the top stitches. NOT the top (visible) stitching.
