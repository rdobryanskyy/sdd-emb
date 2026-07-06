# Embroidery Production Knowledge

> Compiled via web research on 2026-07 — verify against authoritative manufacturer/format specs before treating any number here as a hard constraint in code (e.g. a database CHECK constraint).

This document covers the production-floor side of embroidery: how multi-head machines run a job, how multi-color designs are sequenced, thread numbering conventions, stabilizer selection, and how production time is estimated.

## Multi-head / multi-needle production workflow

Industrial embroidery scales output by adding **heads**, not by making one head faster:

- On a multi-head machine, **every head stitches the same design at the same time**, synchronized by the controller so all items come out identical — you cannot run different designs on different heads of the same machine in a single run [Source: search-aggregated multi-head machine guides, e.g. digitizingusa — Single vs Multi-Head Embroidery Machines](https://www.digitizingusa.com/showblog/single-head-vs-multi-head-embroidery-machines).
- Each head has its own needle bank and thread supply, so color changes happen in lockstep across every head simultaneously [Source: search-aggregated multi-head machine guides](https://www.fujamachine.com/a-exploring-the-advantages-of-a-multi-head-computerized-embroidery-machine.html).
- This is why multi-head machines are the production answer to volume: identical items are loaded one per head, and the marginal cost per item drops as head count goes up for a large enough run, since setup (digitizing, hooping the first frame, color-change programming) is amortized across all heads [Source: search-aggregated multi-head machine guides](https://www.fujamachine.com/a-exploring-the-advantages-of-a-multi-head-computerized-embroidery-machine.html).
- Common commercial head counts (Tajima product history): 1, 2, 4, 6, 8, 10, 12, 15, 20, and 24 heads [Source: search-aggregated Tajima product listings](https://www.theembroiderywarehouse.com/xcart/tajima-tme-dc912-12-head-9-needle-commercial-embroidery-machine.html); Barudan offers comparable 12–15-head lines [Source: Barudan America — 12-15-Head](https://www.barudanamerica.com/12-15-head/).

## Multi-color workflow

A multi-color design is sequenced as an ordered list of "color blocks," each block being a run of stitches in one thread before the next `COLOR_CHANGE`. Two strategies exist for getting the right thread under the needle at the right time:

- **Thread-change (single/few-needle machines)**: the operator manually re-threads a needle each time the design reaches a new color block. This is standard on single-needle home machines and slows production because the machine stops and waits for the operator.
- **Needle pre-assignment (multi-needle/multi-head machines)**: all colors the design needs are threaded onto separate needles *before* the run starts (up to the machine's needle count), and the machine auto-advances between needles at each `COLOR_CHANGE` without stopping. If a design needs more distinct colors than the machine has needles, the run still requires at least one manual re-thread stop partway through.
- Digitizing software (and file formats like DST) represent color changes as ordered markers, not resolved thread assignments — resolving marker-to-needle or marker-to-physical-thread is a separate production-planning step, distinct from the digitizing step (see `file-formats.md` on how little color metadata some formats actually store).
- A `STOP` command (as opposed to `COLOR_CHANGE`) is the mechanism used to force a deliberate pause mid-color-block, most commonly for appliqué fabric placement/trimming — see `stitch-vocabulary.md` and `machine-constraints.md`.

## Thread manufacturer / palette conventions

Thread color numbers are **manufacturer-specific and not portable** across brands — the same numeric code means a different color (or doesn't exist) in a different manufacturer's catalog:

- Major embroidery thread brands with their own numbering systems include **Madeira**, **Isacord**, and **Robison-Anton** [Source: search-aggregated thread conversion guides, e.g. sewingmachinefun — Thread Color Conversion Charts](https://www.sewingmachinefun.com/embroidery-thread-color-conversion-charts/).
- Cross-brand "conversion charts" exist precisely because there is no shared numbering standard; even so, a converted match is typically the *closest available* color, not an exact dye match — sheen, fiber type (rayon vs. polyester), and dye absorption all cause visible differences even at a supposedly equivalent number [Source: search-aggregated thread conversion guides](https://www.hooptalent.com/blogs/news/ultimate-embroidery-hoop-size-chart-master-conversions-perfect-fit); [Source: The Thread Exchange — Madeira to Robison-Anton conversion chart](https://www.thethreadexchange.com/miva/merchant.mvc?Screen=CTGY&Category_Code=rastore-madeira-to-robison-anton-rayon).
- Some conversions are indirect: e.g. no direct Madeira-to-Isacord chart is commonly published, so a practical workflow goes Madeira → Robison-Anton (as an intermediary) → Isacord [Source: search-aggregated thread conversion guides](https://colmanandcompany.com/blog/wp-content/uploads/2017/11/All-Thread-Conversions-1.pdf).
- Implication for a data model: a design's stored "color" per color-block should carry **both** a manufacturer identity and a manufacturer-specific code (e.g. `{brand: "Isacord", code: "0123"}`), never a bare numeric code alone — and any brand-to-brand substitution logic should be modeled as a lookup against a conversion table, explicitly allowing for "closest match" rather than exact equivalence.

## Stabilizers

Stabilizer choice is driven primarily by fabric type and by whether the backing needs to be removable/invisible on the finished piece. The three main removal-method categories:

| Stabilizer type | Removal | Typical use | Source |
|---|---|---|---|
| **Cutaway** | Trimmed close to the stitching after completion; the rest stays in the garment permanently | Best general-purpose stabilizer for stretch/knit fabrics (t-shirts, sweaters) and any fabric needing maximum registration stability under dense designs; recommended default for most embroidery [Source: OESD — Machine Embroidery Stabilizer Basics](https://support.oesd.com/article/30-machine-embroidery-stabilizer-basics); [Source: Sulky — Cut Away and Tear Away Stabilizer Basics](https://blog.sulky.com/stabilizer-basics-cut-away-tear-away-stabilizers/) |
| **Tearaway** | Torn away by hand after stitching, though residual fibers soften further with washing | Preferred for stable wovens where the stitch back will be visible (towels, linens) and for lighter designs — not recommended for very dense designs since tearaway gives less support [Source: OESD — Machine Embroidery Stabilizer Basics](https://support.oesd.com/article/30-machine-embroidery-stabilizer-basics) |
| **Water-soluble (wash-away)** | Fully dissolves in water (sometimes needing a steam/soak step) | Used where no stabilizer residue can remain: freestanding lace, freestanding appliqué/emblems, and any design meant to show no backing at all [Source: OESD — Machine Embroidery Stabilizer Basics](https://support.oesd.com/article/30-machine-embroidery-stabilizer-basics) |
| **Adhesive / sticky-back** | Pressure-sensitive or heat/water-activated adhesive backing that the fabric is pressed onto instead of being hooped directly | Used for hard-to-hoop items, items too small to hoop conventionally, or blanks (like pre-made caps or bags) that would shift inside a normal hoop [Source: search-aggregated stabilizer guides](https://americanemb.com/pages/all-about-stabilizers-faq) |

General rule of thumb: **denser designs need heavier/stronger stabilizer** — a lightweight tearaway will not hold up under a heavily filled design, and large designs generally do better on cutaway regardless of fabric [Source: search-aggregated stabilizer guides](https://support.oesd.com/article/30-machine-embroidery-stabilizer-basics).

## Production-time / stitch-count estimation

The basic time model is: **run time ≈ total stitch count ÷ effective stitches-per-minute (SPM)**, where effective SPM is *lower* than the machine's rated maximum because of real-world slowdowns:

- Baseline formula example: a 25,600-stitch design at 600 SPM takes roughly 42 minutes [Source: search-aggregated production-time calculators, e.g. Creshy — Stitch Count Calculator](https://creshy.com/tools/stitch-count-calculator/).
- If stitch count isn't known yet (e.g. at the estimation stage before digitizing is final), a rough density-based heuristic is **4,000–6,000 stitches per square inch** for dense/filled designs and **2,000–3,000 stitches per square inch** for light/open designs [Source: search-aggregated production-time guides, e.g. jpgtodst — Embroidery Time Calculator](https://jpgtodst.com/embroidery-time-calculator/).
- Rated machine SPM is not the number to plug directly into a time estimate — real average throughput is commonly modeled as the rated SPM reduced by an **efficiency factor** (a commonly cited example uses ~80% of a 1,000 SPM rated machine, i.e. ~800 effective SPM) to account for thread breaks, color changes, and manual interventions [Source: search-aggregated production-time guides](https://tex-inc.com/blogs/digitizing-embroidery/maximizing-embroidery-production-calculating-pieces-per-hour-for-different-machine-types).
- Color changes and trims have a real, non-trivial time cost beyond the stitching itself — one aggregated example cites roughly **8 manual color swaps burning up to ~30 minutes** before a design finishes, purely from thread-change overhead [Source: search-aggregated production-time guides](https://tex-inc.com/blogs/digitizing-embroidery/maximizing-embroidery-production-calculating-pieces-per-hour-for-different-machine-types).
- A commonly cited rule of thumb adds **25–40%** on top of the naive stitch-count ÷ rated-SPM calculation to account for real-world slowdowns across a full job [Source: search-aggregated production-time guides](https://tex-inc.com/blogs/digitizing-embroidery/maximizing-embroidery-production-calculating-pieces-per-hour-for-different-machine-types).
- **Hooping/setup time** is a separate additive term, not part of the stitching-speed calculation: a commonly cited planning figure is **2–5 minutes per garment** for hooping and trimming, on top of a fixed per-order setup time [Source: search-aggregated production-time guides](https://tex-inc.com/blogs/digitizing-embroidery/maximizing-embroidery-production-calculating-pieces-per-hour-for-different-machine-types).

### Estimation model summary (for a data-model / API to encode)

A defensible production-time estimate needs, at minimum, these inputs — each independently sourced above:

1. **Stitch count** (from the actual digitized file if available; otherwise density × area heuristic)
2. **Effective SPM** = rated machine SPM × an efficiency factor (commonly ~0.7–0.8, not 1.0)
3. **Number of color changes / trims**, each carrying its own fixed time cost (manual re-thread on low-needle-count machines is much more expensive than an automatic needle-bank advance on a multi-needle machine)
4. **Hooping/setup time**, a fixed per-item overhead independent of stitch count
5. **Head count**, if running a multi-head job — total run time for a batch is roughly the per-item time above (since all heads run in parallel), not per-item time × item count, once a batch is loaded across all heads

None of the specific coefficients above (efficiency %, per-swap minutes, per-garment hooping minutes) come from a single authoritative benchmark — they are aggregated from industry blogs and calculators, and should be exposed as configurable parameters per machine/shop profile rather than hard-coded constants.
