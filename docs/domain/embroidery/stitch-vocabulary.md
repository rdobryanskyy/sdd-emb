# Digitizing Stitch Vocabulary

> Compiled through web research on 2026-07. Before you use a number from this file as a hard constraint in code (for example, a database CHECK constraint), compare it with the authoritative manufacturer or format specifications.

This document defines the core digitizing vocabulary that a data model or a QA rule set for embroidery designs needs. It covers the stitch types, the underlay strategy, the density and compensation parameters, and special cases such as appliqué, sequins and lettering.

## Core stitch types

### Running / straight stitch

A single stitch between two points. It is the simplest primitive. Use it for outlines, light detail, and as a connector or travel stitch between design areas [Source: Wikipedia — Machine embroidery](https://en.wikipedia.org/wiki/Machine_embroidery); [Source: Absolute Digitizing — Run Stitch, Satin Stitch, and Fill Stitch](https://absolutedigitizing.com/understanding-run-stitch-satin-stitch-and-fill-stitch-in-machine-embroidery/).

### Satin stitch (column stitch)

Zigzag stitches with very small spaces, which lay thread from side to side across a defined width. They make a bold, glossy, raised line. Use them for borders, lettering, and to cover the raw edges of appliqué [Source: EduTechWiki — Embroidery stitch type](https://edutechwiki.unige.ch/en/Embroidery_stitch_type); [Source: Absolute Digitizing — Run/Satin/Fill](https://absolutedigitizing.com/understanding-run-stitch-satin-stitch-and-fill-stitch-in-machine-embroidery/). The trade uses "column stitch" and "satin stitch" as the same term [Source: search-aggregated digitizing glossary, e.g. embroiderylegacy — Satin Stitch Explained](https://embroiderylegacy.com/satin-stitch-embroidery-digitizing/).

- Satin stitch has a width limit. Above a certain column width, the stitch can snag and gives bad coverage. For this reason, a digitizer usually changes very wide "satin" areas to a fill. For the numeric side of this trade-off, see stitch density below.

### Fill stitch (tatami and variants)

A series of run stitches that cover a broad area. Usually the stitches are in parallel rows with an offset, as in a brick wall. The name "tatami" comes from the woven structure of Japanese tatami mats. The offset prevents one weak seam line through the fill [Source: EduTechWiki — Embroidery stitch type](https://edutechwiki.unige.ch/en/Embroidery_stitch_type); [Source: Absolute Digitizing — Run/Satin/Fill](https://absolutedigitizing.com/understanding-run-stitch-satin-stitch-and-fill-stitch-in-machine-embroidery/).

- Fill can go in one direction, or in different directions and angles across sub-regions of a shape. This makes texture or decreases fabric distortion.
- "Tatami" is the classic, default fill algorithm. Digitizing software usually gives fill *variants* as alternatives to plain tatami. Examples: contour fill, which follows the outline of a shape, and motif fill (see below).

### Motif / decorative fill

A fill technique that repeats a small decorative shape (a "motif") along a path or across an area. Examples of a motif: a scallop, a cross-hatch, a leaf. It does not lay plain parallel rows. Use it for ornamental texture, not for flat coverage [Source: search-aggregated digitizing software documentation on motif/decorative fill objects, e.g. Embrilliance StitchArtist product docs](https://embrilliance.com/Help/Platform%20Win%201148/productstitchartist.htm).

## Underlay

Underlay is a low-density foundation layer of stitches. The machine sews it *before* the visible top stitching. It has these functions: it attaches the fabric to the backing, it gives a flatter and firmer surface for the top layer, it decreases the show-through of the base fabric color, and it decreases the pull and distortion from the dense top stitching [Source: theembroiderycoach — Embroidery Underlay Stitching Is Important](https://theembroiderycoach.com/embroidery-underlay-stitching-is-important/); [Source: search-aggregated digitizing underlay guides](https://www.cre8iveskill.com/blog/essential-guide-to-underlay-stitches-for-embroidery-digitizing-service).

Common underlay types:

| Underlay type | Pattern | Typical use |
|---|---|---|
| **Edge walk / contour** | One running-stitch line just inside the edge of the object | Makes a stable "rollover edge" for letters and small objects [Source: search-aggregated underlay guides](https://www.cre8iveskill.com/blog/essential-guide-to-underlay-stitches-for-embroidery-digitizing-service) |
| **Zigzag** | A zigzag with wide spaces along the length of a satin column, at a density much lower than the top stitch | Standard underlay for satin columns. It attaches the column to the backing. It makes the stitch count about two times larger, at low density [Source: search-aggregated underlay guides](https://www.cre8iveskill.com/blog/essential-guide-to-underlay-stitches-for-embroidery-digitizing-service) |
| **Lattice / net (mesh)** | A low-density fill at 45°/90° to the top stitching (or two passes, one on the other, at opposite 45°/135° angles) | Standard underlay for fill/tatami areas. The crossed angles make a mesh under the top fill. This gives better coverage and decreases show-through [Source: search-aggregated underlay guides](https://www.cre8iveskill.com/blog/essential-guide-to-underlay-stitches-for-embroidery-digitizing-service) |

The top-stitch type and the object size set the underlay density and type. Small lettering usually needs a lighter centerline-style underlay, so that the letters do not look "inflated". Large fills get a better result with a full lattice underlay [Source: search-aggregated small-lettering digitizing guides, e.g. Digitizing Buddy — How to Digitize Small Letters](https://digitizingbuddy.com/digitize-small-letters-for-embroidery/).

## Stitch density

Density tells how near the stitches are to each other in a satin column or a fill area. There are two units of measurement: **spacing** (the distance between adjacent stitch rows) or **stitches per unit of length or area**:

- A frequently cited default row spacing for satin and fill is about **3.0–4.0**, in the density units of the digitizing software (usually points or hundredths of a mm, as the tool sets). Adjust it for the thread weight [Source: search-aggregated digitizing density guides, e.g. Hooping Station — Embroidery Digitizing Cheat Sheet](https://hoopingstation.com/blogs/articles/the-ultimate-embroidery-digitizing-cheat-sheet-density-underlay-pull-comp-and-objects-explained-for-real-stitch-out-results).
- Sources also give about **6–8 stitches per mm** as a coverage benchmark for fill stitches <!-- TBD: verify — this figure appeared once in aggregated search results without a clearly primary source; treat as approximate -->.
- An approximate heuristic for stitch-count planning (not a density specification, but useful to estimate the output size and the production time, see `production.md`): dense or filled designs have about **4,000–6,000 stitches per square inch**. Light or open designs have about **2,000–3,000 stitches per square inch** [Source: search-aggregated production-time guides, e.g. embroidery time calculators](https://jpgtodst.com/embroidery-time-calculator/).

## Pull compensation

Dense stitching mechanically **pulls fabric inward**. After the stitch-out, a digitized shape becomes smaller than its intended size. Pull compensation corrects this. Digitizers deliberately draw shapes a little larger or wider than the target artwork. They expect that the finished stitch-out pulls back to the correct size [Source: Embroiderylegacy — Push and Pull Compensation](https://embroiderylegacy.com/push-pull-compensation-embroidery-digitizing/).

- A frequently cited starting value for compensation is **~0.17–0.20 mm** [Source: search-aggregated digitizing guides, e.g. Hooping Station cheat sheet](https://hoopingstation.com/blogs/articles/the-ultimate-embroidery-digitizing-cheat-sheet-density-underlay-pull-comp-and-objects-explained-for-real-stitch-out-results).
- Digitizing software usually shows compensation on a scale with limits. Examples from the unit system of some tools: **0–30 for satin** and **0–20 for fill** areas. These are software-specific scales, not universal millimeter values. Do not think that they are comparable across products [Source: search-aggregated digitizing guides](https://hoopingstation.com/blogs/articles/the-ultimate-embroidery-digitizing-cheat-sheet-density-underlay-pull-comp-and-objects-explained-for-real-stitch-out-results).
- A frequent rule of thumb for manual digitizing: put the input points about one stitch (~0.4 mm) inside the artwork boundary, to compensate in advance [Source: search-aggregated digitizing guides](https://hoopingstation.com/blogs/articles/the-ultimate-embroidery-digitizing-cheat-sheet-density-underlay-pull-comp-and-objects-explained-for-real-stitch-out-results).
- Compensation depends on the fabric and the stabilizer. A stretch knit needs more compensation than a stable woven fabric. For this reason, digitizing software uses a value for each object that the operator can tune, not a fixed constant.

## Appliqué

Appliqué attaches a separate piece of fabric to the base garment as part of the design. It is digitized as a specific multi-step stitch sequence. Usually the steps have separate colors or stops, so that the operator can do manual work between the phases [Source: search-aggregated appliqué tutorials, e.g. Karlie Belle — Applique Tutorial](https://karliebelle.com/applique-tutorial-for-machine-embroidery-start-to-finish/); [Source: Ink/Stitch — Applique tutorial](https://inkstitch.org/tutorials/applique/).

Typical sequence:

1. **Placement line** — the machine first sews a running-stitch outline on the base garment. It shows exactly where to put the appliqué fabric.
2. **Fabric placement** — the operator puts the appliqué fabric on the placement line. Usually the machine stops here (see the `STOP` semantics in `machine-constraints.md`).
3. **Tack-down stitch** — a running or light zigzag stitch sews through the appliqué fabric and the base along the same outline. It attaches the fabric.
4. **Trim** — the operator cuts away the unwanted appliqué fabric outside the tack-down line. This is frequently another `STOP` point.
5. **Cover stitch** — a satin (column) stitch goes over the tack-down line. It seals the raw fabric edge and gives the finished decorative border [Source: search-aggregated appliqué tutorials, e.g. Kimberbell — Mastering Applique Techniques](https://kimberbell.com/blogs/thekimberbellablog/mastering-applique-techniques-with-machine-embroidery); [Source: OESD — Trim in Place Applique Tutorial](https://support.oesd.com/article/35-trim-in-place-applique-tutorial).

Usually the placement, tack-down and cover stitches are digitized as **separate colors or color stops**. The only reason is that the machine must pause between the phases for manual work. The thread color does not change at each step for aesthetic reasons [Source: search-aggregated appliqué tutorials](https://karliebelle.com/applique-tutorial-for-machine-embroidery-start-to-finish/).

## Sequins

Sequin attachment is a machine-hardware feature (a sequin feeder or attachment device), not a universal file-format feature. At the file-format level:

- Sources most frequently give DST as the format with first-class sequin support. A `SEQUIN_MODE` toggle changes how the machine interprets the next jump-class commands. In effect, a `JUMP` becomes a `SEQUIN_EJECT` at the destination coordinate of the jump [Source: pyembroidery README / project docs](https://github.com/EmbroidePy/pyembroidery).
- Other formats usually have no native sequin command. Conversion tools use a configurable fallback ("sequin contingency"): they change the sequin instruction into a plain jump or a plain stitch, they drop it, or they use a format-specific equivalent [Source: pyembroidery README](https://github.com/EmbroidePy/pyembroidery).
- Tajima introduced the first commercial sequin embroidery machine in 1986 [Source: Wikipedia — Machine embroidery](https://en.wikipedia.org/wiki/Machine_embroidery).
- Digitizing for sequins must include the sequin size and the feed pitch and timing. Then the attachment device puts the discs at the correct spacing [Source: search-aggregated sequin digitizing guide, e.g. DigitEMB — Complete Guide of Sequin Embroidery Digitizing](https://www.digitemb.com/blog/sequin-embroidery-digitizing/).

## Lettering / monogramming

Lettering and monogramming are a special digitizing case. If you use generic satin or fill settings again for small text, the risk of stitch-quality failures (bridging, bad legibility, thread breaks) is much higher:

- General guidance: capital letters must be **at least ~6.4 mm** high. Mixed-case (sentence or title case) text must be **at least ~5 mm**. All-caps text can go down to **~3.8 mm**. Below that, the legibility and stitch-quality risk increases quickly [Source: search-aggregated small-lettering digitizing guides, e.g. Digitizing Buddy — How to Digitize Small Letters](https://digitizingbuddy.com/digitize-small-letters-for-embroidery/).
- Monogramming is most reliable in the height range **~1.3–5 cm (½ inch – 2 inch)**. In this range, satin-stitch letter shapes stay durable and well-formed [Source: search-aggregated monogram digitizing guides](https://digitizingbuddy.com/digitize-small-letters-for-embroidery/).
- Sources frequently give about **0.8 mm** as the minimum satin column width (the width of one letter stroke). Below that, the column becomes unstable and the pull-compensation trade-offs become severe [Source: search-aggregated small-lettering digitizing guides](https://digitizingbuddy.com/digitize-small-letters-for-embroidery/).
- The internal counters of a letter (the closed space in a loop, for example in an "o" or an "e") must stay at least **~0.9 mm** in diameter. If not, they fill in [Source: search-aggregated small-lettering digitizing guides](https://digitizingbuddy.com/digitize-small-letters-for-embroidery/).
- The recommended space between letters ("walking distance") is about **0.5–1.0 mm**. Less space looks cluttered. More space looks disconnected [Source: search-aggregated small-lettering digitizing guides](https://digitizingbuddy.com/digitize-small-letters-for-embroidery/).
- For small text, the underlay is deliberately light. A light centerline-style underlay is best under ~1.5 cm letters. Then the underlay does not make thin strokes look bulky [Source: search-aggregated small-lettering digitizing guides](https://digitizingbuddy.com/digitize-small-letters-for-embroidery/).
- A common technique for small letters is a slightly lower satin density (row spacing about **0.30–0.40 mm**). This prevents too much bulk on thin strokes [Source: search-aggregated small-lettering digitizing guides](https://digitizingbuddy.com/digitize-small-letters-for-embroidery/).

All numeric ranges in this section come from digitizing education and tutorial sources, not from one manufacturer specification. Use them as **starting points that you show as tunable settings**, not as fixed validation thresholds.
