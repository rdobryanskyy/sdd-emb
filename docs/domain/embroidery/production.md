# Embroidery Production Knowledge

> Compiled through web research on 2026-07. Before you use a number from this file as a hard constraint in code (for example, a database CHECK constraint), compare it with the authoritative manufacturer or format specifications.

This document is about the production-floor side of embroidery. It tells how multi-head machines run a job, how multi-color designs are put in sequence, the thread numbering conventions, the stabilizer selection, and how to estimate the production time.

## Multi-head / multi-needle production workflow

Industrial embroidery increases the output with more **heads**, not with a faster head:

- On a multi-head machine, **each head stitches the same design at the same time**. The controller synchronizes the heads, so that all items are identical. You cannot run different designs on different heads of the same machine in one run [Source: search-aggregated multi-head machine guides, e.g. digitizingusa — Single vs Multi-Head Embroidery Machines](https://www.digitizingusa.com/showblog/single-head-vs-multi-head-embroidery-machines).
- Each head has its own needle bank and thread supply. Thus the color changes occur at the same time on all heads [Source: search-aggregated multi-head machine guides](https://www.fujamachine.com/a-exploring-the-advantages-of-a-multi-head-computerized-embroidery-machine.html).
- For this reason, multi-head machines are the production solution for volume. The operator loads one identical item on each head. For a sufficiently large run, the cost of each item decreases when the head count increases. The setup (digitizing, hooping of the first frame, color-change programming) is divided across all heads [Source: search-aggregated multi-head machine guides](https://www.fujamachine.com/a-exploring-the-advantages-of-a-multi-head-computerized-embroidery-machine.html).
- Common commercial head counts (Tajima product history): 1, 2, 4, 6, 8, 10, 12, 15, 20 and 24 heads [Source: search-aggregated Tajima product listings](https://www.theembroiderywarehouse.com/xcart/tajima-tme-dc912-12-head-9-needle-commercial-embroidery-machine.html); Barudan has similar 12–15-head lines [Source: Barudan America — 12-15-Head](https://www.barudanamerica.com/12-15-head/).

## Multi-color workflow

A multi-color design is an ordered list of "color blocks". Each block is a sequence of stitches in one thread before the next `COLOR_CHANGE`. There are two strategies to get the correct thread under the needle at the correct time:

- **Thread-change (machines with one or few needles)**: each time the design gets to a new color block, the operator threads a needle again manually. This is standard on single-needle home machines. It makes production slower, because the machine stops and waits for the operator.
- **Needle pre-assignment (multi-needle/multi-head machines)**: *before* the run starts, the operator threads all colors of the design on separate needles (up to the needle count of the machine). At each `COLOR_CHANGE`, the machine goes to the next needle automatically, without a stop. If a design needs more different colors than the machine has needles, the run still needs one or more manual re-thread stops.
- Digitizing software (and file formats such as DST) show color changes as ordered markers, not as resolved thread assignments. The assignment of a marker to a needle or to a physical thread is a separate production-planning step. It is not part of the digitizing step (see `file-formats.md` about how little color metadata some formats keep).
- A `STOP` command (not a `COLOR_CHANGE`) is the mechanism that causes a deliberate pause in a color block. It is most frequently used to put and trim appliqué fabric. See `stitch-vocabulary.md` and `machine-constraints.md`.

## Thread manufacturer / palette conventions

Thread color numbers are **specific to the manufacturer and not portable** across brands. The same numeric code is a different color (or does not exist) in the catalog of a different manufacturer:

- Major embroidery thread brands with their own numbering systems are, for example, **Madeira**, **Isacord** and **Robison-Anton** [Source: search-aggregated thread conversion guides, e.g. sewingmachinefun — Thread Color Conversion Charts](https://www.sewingmachinefun.com/embroidery-thread-color-conversion-charts/).
- Cross-brand "conversion charts" exist because there is no shared numbering standard. But a converted match is usually the *nearest available* color, not an exact dye match. Sheen, fiber type (rayon or polyester) and dye absorption all cause visible differences, also at a number that is said to be equivalent [Source: search-aggregated thread conversion guides](https://www.hooptalent.com/blogs/news/ultimate-embroidery-hoop-size-chart-master-conversions-perfect-fit); [Source: The Thread Exchange — Madeira to Robison-Anton conversion chart](https://www.thethreadexchange.com/miva/merchant.mvc?Screen=CTGY&Category_Code=rastore-madeira-to-robison-anton-rayon).
- Some conversions are indirect. For example, a direct Madeira-to-Isacord chart is not usually published. Thus a practical workflow goes Madeira → Robison-Anton (as an intermediary) → Isacord [Source: search-aggregated thread conversion guides](https://colmanandcompany.com/blog/wp-content/uploads/2017/11/All-Thread-Conversions-1.pdf).
- Effect on a data model: the stored "color" of each color block must have **both** a manufacturer identity and a manufacturer-specific code (for example, `{brand: "Isacord", code: "0123"}`), never only a numeric code. Model each brand-to-brand substitution as a lookup in a conversion table. Explicitly permit a "nearest match", not only an exact equivalent.

## Stabilizers

The fabric type is the primary factor for the stabilizer selection. The second factor is whether the backing must be removable or invisible on the finished item. There are three main categories of removal method:

| Stabilizer type | Removal | Typical use | Source |
|---|---|---|---|
| **Cutaway** | After stitching, cut it near the stitches; the remaining part stays permanently in the garment | The best general-purpose stabilizer for stretch/knit fabrics (t-shirts, sweaters), and for each fabric that needs maximum registration stability under dense designs; the recommended default for most embroidery [Source: OESD — Machine Embroidery Stabilizer Basics](https://support.oesd.com/article/30-machine-embroidery-stabilizer-basics); [Source: Sulky — Cut Away and Tear Away Stabilizer Basics](https://blog.sulky.com/stabilizer-basics-cut-away-tear-away-stabilizers/) |
| **Tearaway** | After stitching, tear it away by hand; the remaining fibers become softer with each wash | The preferred stabilizer for stable woven fabrics where the back of the stitching is visible (towels, linens), and for lighter designs. Not recommended for very dense designs, because tearaway gives less support [Source: OESD — Machine Embroidery Stabilizer Basics](https://support.oesd.com/article/30-machine-embroidery-stabilizer-basics) |
| **Water-soluble (wash-away)** | Fully dissolves in water (sometimes it needs a steam or soak step) | For items where no stabilizer residue can stay: freestanding lace, freestanding appliqué/emblems, and each design that must show no backing [Source: OESD — Machine Embroidery Stabilizer Basics](https://support.oesd.com/article/30-machine-embroidery-stabilizer-basics) |
| **Adhesive / sticky-back** | A backing with pressure-sensitive or heat/water-activated adhesive. The operator presses the fabric on it, and does not put the fabric directly in the hoop | For items that are difficult to hoop, items that are too small for a conventional hoop, or blanks (such as finished caps or bags) that would move in a normal hoop [Source: search-aggregated stabilizer guides](https://americanemb.com/pages/all-about-stabilizers-faq) |

General rule of thumb: **denser designs need a heavier, stronger stabilizer**. A lightweight tearaway does not hold a design with much fill. Large designs usually get a better result on cutaway, whatever the fabric [Source: search-aggregated stabilizer guides](https://support.oesd.com/article/30-machine-embroidery-stabilizer-basics).

## Production-time / stitch-count estimation

The basic time model is: **run time ≈ total stitch count ÷ effective stitches per minute (SPM)**. The effective SPM is *lower* than the rated maximum of the machine, because of real-world slowdowns:

- Example of the baseline formula: a design with 25,600 stitches at 600 SPM takes about 42 minutes [Source: search-aggregated production-time calculators, e.g. Creshy — Stitch Count Calculator](https://creshy.com/tools/stitch-count-calculator/).
- If the stitch count is not known yet (for example, at the estimation stage before digitizing is final), use an approximate density heuristic. It is **4,000–6,000 stitches per square inch** for dense or filled designs, and **2,000–3,000 stitches per square inch** for light or open designs [Source: search-aggregated production-time guides, e.g. jpgtodst — Embroidery Time Calculator](https://jpgtodst.com/embroidery-time-calculator/).
- Do not use the rated SPM of the machine directly in a time estimate. Models usually calculate the real average throughput as the rated SPM multiplied by an **efficiency factor**. A frequently cited example uses ~80% of a machine with a rated 1,000 SPM, thus ~800 effective SPM. The factor includes thread breaks, color changes and manual work [Source: search-aggregated production-time guides](https://tex-inc.com/blogs/digitizing-embroidery/maximizing-embroidery-production-calculating-pieces-per-hour-for-different-machine-types).
- Color changes and trims have a real time cost that is not small, in addition to the stitching. One aggregated example says that about **8 manual color changes can use up to ~30 minutes** before a design is finished, only for the thread changes [Source: search-aggregated production-time guides](https://tex-inc.com/blogs/digitizing-embroidery/maximizing-embroidery-production-calculating-pieces-per-hour-for-different-machine-types).
- A frequently cited rule of thumb adds **25–40%** to the simple calculation (stitch count ÷ rated SPM). This includes the real-world slowdowns across a full job [Source: search-aggregated production-time guides](https://tex-inc.com/blogs/digitizing-embroidery/maximizing-embroidery-production-calculating-pieces-per-hour-for-different-machine-types).
- **Hooping and setup time** is a separate term that you add. It is not part of the stitching-speed calculation. A frequently cited planning value is **2–5 minutes per garment** for hooping and trimming, in addition to a fixed setup time for each order [Source: search-aggregated production-time guides](https://tex-inc.com/blogs/digitizing-embroidery/maximizing-embroidery-production-calculating-pieces-per-hour-for-different-machine-types).

### Estimation model summary (for a data-model / API to encode)

A defensible production-time estimate needs at least these inputs. The sections above give an independent source for each input:

1. **Stitch count** (from the actual digitized file, if it is available; if not, the density × area heuristic)
2. **Effective SPM** = rated SPM of the machine × an efficiency factor (usually ~0.7–0.8, not 1.0)
3. **Number of color changes and trims**, each with its own fixed time cost. A manual re-thread on a machine with a low needle count costs much more than an automatic needle-bank change on a multi-needle machine.
4. **Hooping and setup time**, a fixed overhead for each item that does not depend on the stitch count
5. **Head count**, for a multi-head job. When a batch is loaded on all heads, the total run time of the batch is about the time for one item above, because all heads run at the same time. It is not the time for one item × the item count.

No specific coefficient above (efficiency %, minutes for each color change, minutes of hooping for each garment) comes from one authoritative benchmark. They come from industry blogs and calculators. Show them as configurable parameters for each machine or shop profile. Do not hard-code them as constants.
