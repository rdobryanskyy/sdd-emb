# Digitizing Stitch Vocabulary

> Compiled via web research on 2026-07 — verify against authoritative manufacturer/format specs before treating any number here as a hard constraint in code (e.g. a database CHECK constraint).

This document defines the core digitizing vocabulary a data model or QA rule set for embroidery designs needs: the stitch types, underlay strategy, density/compensation parameters, and specialized cases like appliqué, sequins, and lettering.

## Core stitch types

### Running / straight stitch

A single stitch formed between two points — the simplest primitive, used for outlines, light detail, and as connector/travel stitches between design areas [Source: Wikipedia — Machine embroidery](https://en.wikipedia.org/wiki/Machine_embroidery); [Source: Absolute Digitizing — Run Stitch, Satin Stitch, and Fill Stitch](https://absolutedigitizing.com/understanding-run-stitch-satin-stitch-and-fill-stitch-in-machine-embroidery/).

### Satin stitch (column stitch)

Very closely spaced zigzag stitches that lay thread back and forth across a defined width, producing a bold, glossy, raised line — used for borders, lettering, and covering raw appliqué edges [Source: EduTechWiki — Embroidery stitch type](https://edutechwiki.unige.ch/en/Embroidery_stitch_type); [Source: Absolute Digitizing — Run/Satin/Fill](https://absolutedigitizing.com/understanding-run-stitch-satin-stitch-and-fill-stitch-in-machine-embroidery/). "Column stitch" and "satin stitch" are used interchangeably in the trade [Source: search-aggregated digitizing glossary, e.g. embroiderylegacy — Satin Stitch Explained](https://embroiderylegacy.com/satin-stitch-embroidery-digitizing/).

- Satin stitch is width-limited: past a certain column width the stitch is prone to snagging and poor coverage, which is why very wide "satin" areas are conventionally re-digitized as a fill instead — see stitch density below for the numeric side of this trade-off.

### Fill stitch (tatami and variants)

A series of run stitches arranged to cover a broad area, typically laid in parallel rows offset like bricks (hence "tatami," named after the woven structure of Japanese tatami mats) to avoid a single weak seam line running through the fill [Source: EduTechWiki — Embroidery stitch type](https://edutechwiki.unige.ch/en/Embroidery_stitch_type); [Source: Absolute Digitizing — Run/Satin/Fill](https://absolutedigitizing.com/understanding-run-stitch-satin-stitch-and-fill-stitch-in-machine-embroidery/).

- Fill can run in a single direction or in multiple directions/angles across sub-regions of a shape to create texture or reduce fabric distortion.
- "Tatami" is the classic/default fill algorithm; digitizing software typically offers fill *variants* (e.g. contour fill that follows a shape's outline, or motif fill — see below) as alternates to plain tatami.

### Motif / decorative fill

A fill technique that repeats a small decorative shape (a "motif") — e.g. a scallop, a cross-hatch, a leaf — along a path or across an area instead of laying plain parallel rows, used for ornamental texture rather than flat coverage [Source: search-aggregated digitizing software documentation on motif/decorative fill objects, e.g. Embrilliance StitchArtist product docs](https://embrilliance.com/Help/Platform%20Win%201148/productstitchartist.htm).

## Underlay

Underlay is a low-density foundation layer of stitching sewn *before* the visible top stitching, whose job is to stabilize fabric against the backing, provide a flatter/firmer surface for the top layer, minimize show-through of the base fabric color, and reduce pull/distortion from the dense top stitching [Source: theembroiderycoach — Embroidery Underlay Stitching Is Important](https://theembroiderycoach.com/embroidery-underlay-stitching-is-important/); [Source: search-aggregated digitizing underlay guides](https://www.cre8iveskill.com/blog/essential-guide-to-underlay-stitches-for-embroidery-digitizing-service).

Common underlay types:

| Underlay type | Pattern | Typical use |
|---|---|---|
| **Edge walk / contour** | A single running-stitch line just inside the object's edge | Creates a stable "rollover edge" for letters and small objects [Source: search-aggregated underlay guides](https://www.cre8iveskill.com/blog/essential-guide-to-underlay-stitches-for-embroidery-digitizing-service) |
| **Zigzag** | A widely spaced zigzag running the length of a satin column, well below top-stitch density | Standard underlay for satin columns — anchors the column to the backing while roughly doubling stitch count at low density [Source: search-aggregated underlay guides](https://www.cre8iveskill.com/blog/essential-guide-to-underlay-stitches-for-embroidery-digitizing-service) |
| **Lattice / net (mesh)** | A low-density fill run at 45°/90° (or two stacked passes at opposing 45°/135°) to the top stitching | Standard underlay for fill/tatami areas — the crossing angles form a mesh the top fill "rests on," improving coverage and reducing show-through [Source: search-aggregated underlay guides](https://www.cre8iveskill.com/blog/essential-guide-to-underlay-stitches-for-embroidery-digitizing-service) |

Underlay density and type are chosen based on the top-stitch type and object size: small lettering typically wants a lighter centerline-style underlay so the letters don't visually "inflate," while large fills benefit from full lattice underlay [Source: search-aggregated small-lettering digitizing guides, e.g. Digitizing Buddy — How to Digitize Small Letters](https://digitizingbuddy.com/digitize-small-letters-for-embroidery/).

## Stitch density

Density describes how tightly stitches are packed within a satin column or fill area, and is measured either as **spacing** (distance between adjacent stitch rows) or as **stitches-per-unit-length/area**:

- A commonly cited default satin/fill row spacing is around **3.0–4.0** (in the density units digitizing software exposes — typically points or hundredths-of-mm depending on the tool), adjusted for thread weight [Source: search-aggregated digitizing density guides, e.g. Hooping Station — Embroidery Digitizing Cheat Sheet](https://hoopingstation.com/blogs/articles/the-ultimate-embroidery-digitizing-cheat-sheet-density-underlay-pull-comp-and-objects-explained-for-real-stitch-out-results).
- Fill stitches are also commonly quoted at roughly **6–8 stitches per mm** as a coverage benchmark <!-- TBD: verify — this figure appeared once in aggregated search results without a clearly primary source; treat as approximate -->.
- As a rough stitch-count planning heuristic (not a density spec, but useful for estimating output size and production time — see `production.md`): dense/filled designs run roughly **4,000–6,000 stitches per square inch**, while light/open designs run closer to **2,000–3,000 stitches per square inch** [Source: search-aggregated production-time guides, e.g. embroidery time calculators](https://jpgtodst.com/embroidery-time-calculator/).

## Pull compensation

Pull compensation corrects for the fact that dense stitching mechanically **pulls fabric inward**, shrinking a digitized shape below its intended size once stitched out — so digitizers deliberately draw shapes slightly larger/wider than the target artwork, expecting the finished stitch-out to pull back to the correct size [Source: Embroiderylegacy — Push and Pull Compensation](https://embroiderylegacy.com/push-pull-compensation-embroidery-digitizing/).

- A commonly cited starting-point compensation value is **~0.17–0.20 mm** [Source: search-aggregated digitizing guides, e.g. Hooping Station cheat sheet](https://hoopingstation.com/blogs/articles/the-ultimate-embroidery-digitizing-cheat-sheet-density-underlay-pull-comp-and-objects-explained-for-real-stitch-out-results).
- Digitizing software commonly exposes compensation on a bounded scale, e.g. **0–30 for satin** and **0–20 for fill** areas in some tools' unit system — these are software-specific unit scales, not universal millimeter values, so do not assume they're comparable across products [Source: search-aggregated digitizing guides](https://hoopingstation.com/blogs/articles/the-ultimate-embroidery-digitizing-cheat-sheet-density-underlay-pull-comp-and-objects-explained-for-real-stitch-out-results).
- A commonly quoted rule of thumb for manual digitizing is to place input points roughly one stitch short (~0.4 mm) inside the artwork boundary to pre-compensate [Source: search-aggregated digitizing guides](https://hoopingstation.com/blogs/articles/the-ultimate-embroidery-digitizing-cheat-sheet-density-underlay-pull-comp-and-objects-explained-for-real-stitch-out-results).
- Compensation is fabric- and stabilizer-dependent (a stretch knit needs more compensation than a stable woven), which is why digitizing software treats it as a per-object, operator-tunable value rather than a fixed constant.

## Appliqué

Appliqué attaches a separate piece of fabric to the base garment as part of the design, and is digitized as a specific multi-step stitch sequence, conventionally split across separate colors/stops so the operator can physically intervene between phases [Source: search-aggregated appliqué tutorials, e.g. Karlie Belle — Applique Tutorial](https://karliebelle.com/applique-tutorial-for-machine-embroidery-start-to-finish/); [Source: Ink/Stitch — Applique tutorial](https://inkstitch.org/tutorials/applique/).

Typical sequence:

1. **Placement line** — a running-stitch outline is sewn on the base garment first, showing exactly where to position the appliqué fabric.
2. **Fabric placement** — the operator lays the appliqué fabric over the placement line (machine is typically stopped here — see `STOP` semantics in `machine-constraints.md`).
3. **Tack-down stitch** — a running or light zigzag stitch sews through the appliqué fabric and the base along the same outline, securing the fabric in place.
4. **Trim** — excess appliqué fabric outside the tack-down line is trimmed away (often another `STOP` point).
5. **Cover stitch** — a satin (column) stitch runs over the tack-down line, sealing the raw fabric edge and giving the finished decorative border [Source: search-aggregated appliqué tutorials, e.g. Kimberbell — Mastering Applique Techniques](https://kimberbell.com/blogs/thekimberbellablog/mastering-applique-techniques-with-machine-embroidery); [Source: OESD — Trim in Place Applique Tutorial](https://support.oesd.com/article/35-trim-in-place-applique-tutorial).

Placement, tack-down, and cover stitches are conventionally digitized as **separate colors/color-stops**, purely so the machine pauses between phases for manual work — not because thread color changes for aesthetic reasons at each step [Source: search-aggregated appliqué tutorials](https://karliebelle.com/applique-tutorial-for-machine-embroidery-start-to-finish/).

## Sequins

Sequin attachment is a machine-hardware feature (a sequin feeder/attachment device) rather than a universal file-format feature. At the file-format level:

- DST is the format most commonly cited as having first-class sequin support: a `SEQUIN_MODE` toggle changes how subsequent jump-class commands are interpreted, effectively turning a `JUMP` into a `SEQUIN_EJECT` at the jump's destination coordinate [Source: pyembroidery README / project docs](https://github.com/EmbroidePy/pyembroidery).
- Other formats generally have no native sequin command; conversion tools apply a configurable fallback ("sequin contingency") — treat the sequin instruction as a plain jump, a plain stitch, drop it, or a format-specific equivalent [Source: pyembroidery README](https://github.com/EmbroidePy/pyembroidery).
- Tajima introduced the first commercial sequin embroidery machine in 1986 [Source: Wikipedia — Machine embroidery](https://en.wikipedia.org/wiki/Machine_embroidery).
- Digitizing for sequins requires accounting for sequin size and feed pitch/timing so the attachment device places discs at the correct spacing [Source: search-aggregated sequin digitizing guide, e.g. DigitEMB — Complete Guide of Sequin Embroidery Digitizing](https://www.digitemb.com/blog/sequin-embroidery-digitizing/).

## Lettering / monogramming

Lettering and monogramming are treated as a specialized digitizing case because small text is disproportionately prone to stitch-quality failures (bridging, illegibility, thread breaks) if generic satin/fill settings are reused:

- General guidance: capital letters should be **at least ~6.4 mm** tall; mixed-case (sentence/title case) text should be **at least ~5 mm**; all-caps text can go as small as **~3.8 mm** before legibility/stitch-quality risk rises sharply [Source: search-aggregated small-lettering digitizing guides, e.g. Digitizing Buddy — How to Digitize Small Letters](https://digitizingbuddy.com/digitize-small-letters-for-embroidery/).
- Monogramming specifically is most reliable in the **~1.3–5 cm (½ inch – 2 inch)** height range, where satin-stitch letterforms remain durable and well-formed [Source: search-aggregated monogram digitizing guides](https://digitizingbuddy.com/digitize-small-letters-for-embroidery/).
- Minimum satin column width (the width of a single letter stroke) is commonly cited around **0.8 mm** — below that, the column becomes unstable and pull compensation trade-offs get severe [Source: search-aggregated small-lettering digitizing guides](https://digitizingbuddy.com/digitize-small-letters-for-embroidery/).
- Letter internal counters (the enclosed space inside a closed loop, e.g. inside an "o" or "e") should stay at least **~0.9 mm** in diameter to avoid filling in [Source: search-aggregated small-lettering digitizing guides](https://digitizingbuddy.com/digitize-small-letters-for-embroidery/).
- Recommended inter-letter spacing ("walking distance") is roughly **0.5–1.0 mm** — tighter reads as cluttered, looser reads as disconnected [Source: search-aggregated small-lettering digitizing guides](https://digitizingbuddy.com/digitize-small-letters-for-embroidery/).
- For small text, underlay is deliberately kept light (a light centerline-style underlay works best under ~1.5 cm letters) so the underlay itself doesn't visually bulk up thin strokes [Source: search-aggregated small-lettering digitizing guides](https://digitizingbuddy.com/digitize-small-letters-for-embroidery/).
- Reducing satin density slightly for small letters (row spacing around **0.30–0.40 mm**) is a common technique to avoid excess bulk on thin strokes [Source: search-aggregated small-lettering digitizing guides](https://digitizingbuddy.com/digitize-small-letters-for-embroidery/).

All of the numeric ranges in this section come from digitizing-education/tutorial sources rather than a single manufacturer spec, and should be treated as **starting points to expose as tunable settings**, not fixed validation thresholds.
