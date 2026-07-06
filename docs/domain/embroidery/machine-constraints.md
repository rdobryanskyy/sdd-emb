# Embroidery Machine Physical Constraints

> Compiled via web research on 2026-07 — verify against authoritative manufacturer/format specs before treating any number here as a hard constraint in code (e.g. a database CHECK constraint).

Exact numbers vary by machine brand, model, and generation. This file gives **defensible ranges with citations**, not a single invented number. Where sources genuinely disagree or a number could not be confirmed, that is marked explicitly rather than silently resolved.

## Stitch length limits

Embroidery file formats historically inherited their maximum single-stitch (needle-to-needle) delta from the punch-tape coding system a machine's controller used:

| Coding system | Max single stitch/jump delta | Representative machines/formats | Source |
|---|---|---|---|
| Ternary coding | **12.1 mm** (121 × 0.1 mm units) | Tajima, and the classic DST format | [Wilcom EmbroideryStudio — Stitch & jump length settings](https://docs.wilcom.com/embroiderystudio/26/en/OnlineHelp/Setup/machines/Stitch_jump_length_settings.htm); [pyembroidery README](https://github.com/EmbroidePy/pyembroidery) |
| Binary coding | **12.7 mm** (127 × 0.1 mm units, signed 8-bit byte range) | Barudan, and formats like EXP that store one signed byte per axis | [Wilcom EmbroideryStudio — Stitch & jump length settings](https://docs.wilcom.com/embroiderystudio/26/en/OnlineHelp/Setup/machines/Stitch_jump_length_settings.htm) |

Any stitch or jump longer than the active format's limit must be split into multiple consecutive jump/stitch records by the digitizing/export software — this is not optional, it's a hard format ceiling.

For the **minimum** safe stitch length, sources converge on a risk band rather than a single number:

- A commonly cited "red zone" of **0.1–0.9 mm** carries a high risk of thread breakage because the needle doesn't have time to clear the fabric between punches, while a "green zone" of **1.5–3.0 mm** is considered safe for most threads/fabrics [Source: EmbroideryHoopStore — Digitizing that actually stitches out](https://embroideryhoopstore.com/blogs/articles/digitizing-that-actually-stitches-out-the-6-rules-stitch-length-density-underlay-push-pull-pathing-beginners-miss).
- A separate rule of thumb: stitches shorter than roughly **2 mm** can push the machine into a rapid up-down cycle that doesn't let thread recover, leading to snapping [Source: Melco Service — Stitches Too Small](https://www.melco-service.com/source1/Stitches_Too_Small.htm).
- General digitizing guidance is to keep stitch length longer than the needle's diameter and to increase minimum stitch length for denser materials/thicker thread [Source: Wilcom EmbroideryStudio — Compensation/stitch settings docs, aggregated](https://softwarehelp.mysewnet.com/MSM/120/Digitizing/Topics/Compensation.htm).

<!-- TBD: verify — no single authoritative numeric "minimum stitch length" constant exists across formats; the format-level byte encoding (e.g. JEF/EXP's 8-bit signed delta) has no enforced minimum other than 0 (a stitch could in principle land on the same point), so the real floor is a *quality* recommendation, not a *format* limit. -->

## Jump length and trim behavior

- A jump (needle-up travel) longer than the format's per-stitch delta limit (above) must be split into a chain of jumps.
- Machines commonly infer a `TRIM` command after a run of consecutive jumps once some threshold is reached — historically often **3 jumps**, though this can be configured up to **5** on some controllers [Source: search-aggregated pyembroidery/DST community documentation].
- On Brother home/semi-commercial machines, the "minimum jump length to trim" is a discrete, user-selectable setting **from 5 mm to 50 mm in 5 mm increments** — jumps shorter than the selected threshold are not auto-trimmed [Source: Brother — Selecting the length of jump stitch not to trim](https://help.brother-usa.com/app/answers/detail/a_id/186603).
- Very short connector jumps — often cited around **0.5 mm** — may fall below what auto-trim logic recognizes at all, requiring a manual trim [Source: search-aggregated, e.g. Embroideres forum — Trimming jump stitches](https://forum.embroideres.com/articles.html/articles/trimming-jump-stitches-r47/).
- Reducing the configured max-jump value generally improves stitch-out quality and reduces machine wear, at the cost of more trims and longer run time [Source: Wilcom EmbroideryStudio — Stitch & jump length settings](https://docs.wilcom.com/embroiderystudio/26/en/OnlineHelp/Setup/machines/Stitch_jump_length_settings.htm).

## `TRIM` / `COLOR_CHANGE` / `STOP` semantics

These three commands are frequently conflated at the byte level but have distinct production meaning:

- **`COLOR_CHANGE`**: the machine (on multi-needle/multi-head hardware) automatically advances to the next needle/thread color and continues without operator intervention.
- **`STOP`**: on many stitch-file formats this is encoded as *the same underlying command* as a color change, distinguished by whether the target needle/color slot differs from the current one — a color-change-to-the-same-needle is read by the machine as a deliberate pause rather than a real color swap. On Barudan controllers this deliberate pause is explicitly exposed as its own instruction (e.g. `C00`) assignable via the machine UI [Source: search-aggregated digitizing-software documentation, e.g. Sierra Software — Stitches & Machine Commands](https://www.sierra-software.com/downloads/manuals/se20/emb-stitch-machine-commands.html).
- In practice, digitizers insert a `STOP` where a human needs to intervene — most commonly to place and trim appliqué fabric, or to apply an add-on like puff foam — and the machine frame typically advances/ejects toward the operator at that point to make the manual step easier [Source: search-aggregated appliqué/digitizing documentation, e.g. Brother — How to Applique on a Multi-Needle Embroidery Machine](https://www.brother-usa.com/blogs/stitching-sewcial/appliqu-on-a-multi-needle-embroidery-machine).
- `TRIM` cuts the thread at the current position, typically emitted by the digitizing software before a jump long enough to otherwise leave a visible thread carry, or before/after a color change.
- Digitizing software commonly lets an operator freely convert `STOP` commands into `COLOR_CHANGE` commands and vice versa post-hoc, confirming these are treated as the same command class with different intent flags rather than fundamentally different machine operations [Source: search-aggregated digitizing-software documentation].

## Hoop size families

Hoop/frame sizes are named by nominal outer dimension, but the **usable stitching field is always somewhat smaller** than the nominal hoop size:

| Nominal hoop | Approx. usable field (varies by hoop/machine) | Typical use | Source |
|---|---|---|---|
| 4×4 in | ~3.9 in usable | Small logos, monograms, cuffs, baby items | [B-Sew Inn — Embroidery Hoop Size Chart](https://www.bsewinn.com/blogs/inspiration/embroidery-hoop-size-chart) |
| 5×7 in | ~4.9 × 6.9 in usable | General-purpose home-machine "workhorse" size | [B-Sew Inn — Embroidery Hoop Size Chart](https://www.bsewinn.com/blogs/inspiration/embroidery-hoop-size-chart) |
| 6×10 in | ~6.3 × 10.2 in usable | Totes, pillows, jacket backs | [B-Sew Inn — Embroidery Hoop Size Chart](https://www.bsewinn.com/blogs/inspiration/embroidery-hoop-size-chart) |
| 8×8 in, 8×12 in, 14×14 in | Commercial-only territory; standard home machines cannot reach these | Quilt blocks, large panels | [B-Sew Inn — Embroidery Hoop Size Chart](https://www.bsewinn.com/blogs/inspiration/embroidery-hoop-size-chart) |
| 9×9, 10×10 in and larger | Machine- and brand-specific; not a universal standard size | Large-format commercial designs | <!-- TBD: verify — treated as commercial-tier sizes but no single manufacturer spec sheet was checked for these two specific nominal sizes --> |

Cap/hat embroidery uses a fundamentally different fixture rather than a flat hoop:

- **Cap frames** clamp a curved cap crown and commonly allow **up to ~270° of rotation** ("ear to ear") so a multi-position design can be stitched around the curve [Source: search-aggregated cap-frame vendor documentation, e.g. Durkee cap frame listings](https://allstitch.com/products/durkee-tajima-compatible-embroidery-cap-frame-360-sewing-field).
- **Tubular hooping** is the traditional method for cap-back and similar hard-to-reach areas; specialized "hoopless" multi-frame kits exist for sleeves, pockets, socks, and other small/irregular items [Source: search-aggregated, e.g. Ricoma 8-in-1 device](https://ricoma.com/products/8-in-1-device).

**Bottom line**: hoop family names (4×4, 5×7, 6×10, etc.) are industry-conventional but not standardized to a single millimeter figure — exact usable dimensions vary by hoop manufacturer and machine model, and any hoop-size validation logic should treat the nominal name as a rough bucket, not an exact bounding box, unless the target machine/hoop model is known.

## Needle count conventions

Two very different machine architectures use "needle count" differently:

- **Home / semi-commercial multi-needle machines**: a single embroidery head carries multiple pre-threaded needles (commonly **6, 7, 8, or 10** on current consumer/prosumer models), letting the machine auto-advance between thread colors without manual rethreading. Examples cited in vendor material: a 6-needle Brother PR680W, a 10-needle Brother PR1060W, a 7-needle Janome MB-7 [Source: search-aggregated vendor listings, e.g. Brother multi-needle machines](https://www.brother-usa.com/home/sewing-embroidery/multi-needle-machines). Higher-end home-adjacent multi-needle machines up to **12, 15, or 20 needles** exist in the broader market <!-- TBD: verify — 12/15/20-needle figures are commonly quoted in embroidery-business blogs but a specific current manufacturer spec sheet for a 15- or 20-needle single-head consumer/prosumer machine was not independently checked in this pass -->.
- **Industrial multi-head machines**: production floors instead scale by adding **heads**, where each head is itself a small multi-needle unit (all heads stitch the *same* design in parallel, one per garment/item). Tajima has historically offered 1, 2, 4, 6, 8, 10, 12, 15, 20, and 24-head machines, and needle count *per head* varies by model — some listings show 12-head/12-needle (one needle per head) configurations, others 12-head/9-needle [Source: search-aggregated commercial embroidery machine listings, e.g. Tajima TEHX-C1212 and TME-DC912 listings on The Embroidery Warehouse](https://www.theembroiderywarehouse.com/xcart/tajima-tme-dc912-12-head-9-needle-commercial-embroidery-machine.html); Barudan offers comparable 12–15-head product lines [Source: Barudan America — 12-15-Head](https://www.barudanamerica.com/12-15-head/).
- Needle count is **not** a proxy for simultaneous colors across heads in the same way it is within one head — on a multi-head machine, needle *N* is threaded with the same color on every head, so the whole bank of heads changes to color *N* together.

## Machine speed (stitches per minute)

Speed is quoted in stitches per minute (SPM) and varies by machine tier and by whether the figure is a rated maximum or a sustained real-world average:

| Tier | Representative rated max SPM | Source |
|---|---|---|
| Home / entry-level (e.g. Tajima SAI) | ~800 SPM | [search-aggregated Tajima product summaries](https://www.truedigitizing.com/blog/tajima-embroidery-machines) |
| Multi-head commercial (e.g. Barudan 4-head) | ~1,100 SPM | [search-aggregated Barudan product summaries](https://www.maggieframes.com/blogs/embroidery-blogs/barudan-vs-tajima-2025-expert-comparison-for-machine-embroidery-professionals) |
| High-end industrial (e.g. Tajima TWMX-C1501, TMEZ-SC) | up to ~1,200 SPM | [search-aggregated Tajima product summaries](https://www.truedigitizing.com/blog/tajima-embroidery-machines) |
| High-end industrial (e.g. Barudan BEKT-S1501CBIII) | up to ~1,300 SPM | [search-aggregated Barudan product summaries](https://www.maggieframes.com/blogs/embroidery-blogs/barudan-vs-tajima-2025-expert-comparison-for-machine-embroidery-professionals) |

Rated maximum SPM is a ceiling, not a sustained production rate — designs with dense fill, small satin columns, or frequent color changes force the machine to run slower, and real-world *effective* throughput is commonly modeled as the rated speed reduced by an efficiency factor (commonly cited around 70–80%) to account for thread breaks, color changes, and hooping downtime — see `production.md` for the production-time estimation model this feeds into [Source: search-aggregated production-time calculators, e.g. TEX Inc — Calculating pieces per hour](https://tex-inc.com/blogs/digitizing-embroidery/maximizing-embroidery-production-calculating-pieces-per-hour-for-different-machine-types).

## Summary caveat

None of the numeric ranges above should be hard-coded as a single universal constant. A validating pipeline should model these as **per-machine-profile** configuration (max stitch length, max jump length, trim-jump-count threshold, hoop inventory, needle/head count, rated SPM) rather than baking one brand's number in as a global constraint.
