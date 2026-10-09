# Embroidery Machine Physical Constraints

> Compiled through web research on 2026-07. Before you use a number from this file as a hard constraint in code (for example, a database CHECK constraint), compare it with the authoritative manufacturer or format specifications.

The exact numbers change with the machine brand, the model and the generation. This file gives **defensible ranges with citations**, not one invented number. Where sources really disagree, or where a number could not be confirmed, the file marks this explicitly. It does not silently resolve it.

## Stitch length limits

Embroidery file formats got their maximum delta for one stitch (needle to needle) from the punch-tape coding system of the machine controller:

| Coding system | Max single stitch/jump delta | Representative machines/formats | Source |
|---|---|---|---|
| Ternary coding | **12.1 mm** (121 × 0.1 mm units) | Tajima, and the classic DST format | [Wilcom EmbroideryStudio — Stitch & jump length settings](https://docs.wilcom.com/embroiderystudio/26/en/OnlineHelp/Setup/machines/Stitch_jump_length_settings.htm); [pyembroidery README](https://github.com/EmbroidePy/pyembroidery) |
| Binary coding | **12.7 mm** (127 × 0.1 mm units, signed 8-bit byte range) | Barudan, and formats such as EXP that keep one signed byte for each axis | [Wilcom EmbroideryStudio — Stitch & jump length settings](https://docs.wilcom.com/embroiderystudio/26/en/OnlineHelp/Setup/machines/Stitch_jump_length_settings.htm) |

The digitizing or export software must divide each stitch or jump that is longer than the limit of the active format into more than one consecutive jump/stitch record. This is not optional. It is a hard format limit.

For the **minimum** safe stitch length, the sources agree on a risk band, not on one number:

- Sources frequently give a "red zone" of **0.1–0.9 mm**. In this zone, the risk of thread breakage is high, because the needle has no time to come out of the fabric between punches. A "green zone" of **1.5–3.0 mm** is safe for most threads and fabrics [Source: EmbroideryHoopStore — Digitizing that actually stitches out](https://embroideryhoopstore.com/blogs/articles/digitizing-that-actually-stitches-out-the-6-rules-stitch-length-density-underlay-push-pull-pathing-beginners-miss).
- A different rule of thumb: stitches shorter than about **2 mm** can cause a fast up-down cycle in the machine. The thread cannot recover, and it breaks [Source: Melco Service — Stitches Too Small](https://www.melco-service.com/source1/Stitches_Too_Small.htm).
- General digitizing guidance: keep the stitch length longer than the needle diameter. Increase the minimum stitch length for denser materials and thicker thread [Source: Wilcom EmbroideryStudio — Compensation/stitch settings docs, aggregated](https://softwarehelp.mysewnet.com/MSM/120/Digitizing/Topics/Compensation.htm).

<!-- TBD: verify — no single authoritative numeric "minimum stitch length" constant exists across formats; the format-level byte encoding (e.g. JEF/EXP's 8-bit signed delta) has no enforced minimum other than 0 (a stitch could in principle land on the same point), so the real floor is a *quality* recommendation, not a *format* limit. -->

## Jump length and trim behavior

- A jump (needle-up travel) that is longer than the per-stitch delta limit of the format (above) must be divided into a chain of jumps.
- After a number of consecutive jumps, machines usually add a `TRIM` command automatically. Historically the threshold is frequently **3 jumps**. Some controllers can be configured up to **5** [Source: search-aggregated pyembroidery/DST community documentation].
- On Brother home and semi-commercial machines, the "minimum jump length to trim" is a discrete setting. The user selects it **from 5 mm to 50 mm in 5 mm increments**. The machine does not trim jumps that are shorter than the selected threshold automatically [Source: Brother — Selecting the length of jump stitch not to trim](https://help.brother-usa.com/app/answers/detail/a_id/186603).
- The auto-trim logic possibly does not recognize very short connector jumps (frequently about **0.5 mm**). These jumps need a manual trim [Source: search-aggregated, e.g. Embroideres forum — Trimming jump stitches](https://forum.embroideres.com/articles.html/articles/trimming-jump-stitches-r47/).
- A lower configured max-jump value usually gives a better stitch-out and less machine wear. The cost is more trims and a longer run time [Source: Wilcom EmbroideryStudio — Stitch & jump length settings](https://docs.wilcom.com/embroiderystudio/26/en/OnlineHelp/Setup/machines/Stitch_jump_length_settings.htm).

## `TRIM` / `COLOR_CHANGE` / `STOP` semantics

At the byte level, these three commands are frequently mixed up. But they have different production meanings:

- **`COLOR_CHANGE`**: on multi-needle or multi-head hardware, the machine automatically goes to the next needle or thread color. It continues without operator work.
- **`STOP`**: in many stitch-file formats, this is *the same command* as a color change. The difference is whether the target needle or color slot is different from the current one. The machine reads a color change to the same needle as a deliberate pause, not as a real color change. Barudan controllers show this deliberate pause as a separate instruction (for example, `C00`) that the operator can assign in the machine UI [Source: search-aggregated digitizing-software documentation, e.g. Sierra Software — Stitches & Machine Commands](https://www.sierra-software.com/downloads/manuals/se20/emb-stitch-machine-commands.html).
- In practice, digitizers add a `STOP` where a person must do manual work. The most frequent cases are to put and trim appliqué fabric, or to apply an add-on such as puff foam. At that point, the machine frame usually moves toward the operator to make the manual step easier [Source: search-aggregated appliqué/digitizing documentation, e.g. Brother — How to Applique on a Multi-Needle Embroidery Machine](https://www.brother-usa.com/blogs/stitching-sewcial/appliqu-on-a-multi-needle-embroidery-machine).
- `TRIM` cuts the thread at the current position. Usually the digitizing software adds it before a jump that is long enough to leave a visible thread carry, or before or after a color change.
- Digitizing software usually lets an operator change `STOP` commands into `COLOR_CHANGE` commands, and the opposite, at a later time. This confirms that they are the same command class with different intent flags, not different machine operations [Source: search-aggregated digitizing-software documentation].

## Hoop size families

Hoop and frame sizes have names from their nominal outer dimension. But the **usable stitching field is always a little smaller** than the nominal hoop size:

| Nominal hoop | Approx. usable field (varies by hoop/machine) | Typical use | Source |
|---|---|---|---|
| 4×4 in | ~3.9 in usable | Small logos, monograms, cuffs, baby items | [B-Sew Inn — Embroidery Hoop Size Chart](https://www.bsewinn.com/blogs/inspiration/embroidery-hoop-size-chart) |
| 5×7 in | ~4.9 × 6.9 in usable | General-purpose "workhorse" size for home machines | [B-Sew Inn — Embroidery Hoop Size Chart](https://www.bsewinn.com/blogs/inspiration/embroidery-hoop-size-chart) |
| 6×10 in | ~6.3 × 10.2 in usable | Totes, pillows, jacket backs | [B-Sew Inn — Embroidery Hoop Size Chart](https://www.bsewinn.com/blogs/inspiration/embroidery-hoop-size-chart) |
| 8×8 in, 8×12 in, 14×14 in | Only for commercial machines; standard home machines cannot use these sizes | Quilt blocks, large panels | [B-Sew Inn — Embroidery Hoop Size Chart](https://www.bsewinn.com/blogs/inspiration/embroidery-hoop-size-chart) |
| 9×9, 10×10 in and larger | Specific to the machine and the brand; not a universal standard size | Large-format commercial designs | <!-- TBD: verify — treated as commercial-tier sizes but no single manufacturer spec sheet was checked for these two specific nominal sizes --> |

Cap and hat embroidery uses a different fixture, not a flat hoop:

- **Cap frames** hold a curved cap crown. They usually permit **up to ~270° of rotation** ("ear to ear"), so that the machine can stitch a multi-position design around the curve [Source: search-aggregated cap-frame vendor documentation, e.g. Durkee cap frame listings](https://allstitch.com/products/durkee-tajima-compatible-embroidery-cap-frame-360-sewing-field).
- **Tubular hooping** is the traditional method for the back of a cap and for areas that are similarly difficult to reach. Special "hoopless" multi-frame kits exist for sleeves, pockets, socks and other small or irregular items [Source: search-aggregated, e.g. Ricoma 8-in-1 device](https://ricoma.com/products/8-in-1-device).

**Conclusion**: hoop family names (4×4, 5×7, 6×10, etc.) are industry conventions. They are not standardized to one millimeter value. The exact usable dimensions change with the hoop manufacturer and the machine model. Hoop-size validation logic must use the nominal name as an approximate bucket, not an exact bounding box, unless the target machine and hoop model are known.

## Needle count conventions

Two very different machine architectures use "needle count" in different ways:

- **Home and semi-commercial multi-needle machines**: one embroidery head has a number of needles that are threaded in advance. Current consumer and prosumer models usually have **6, 7, 8 or 10**. The machine changes between thread colors automatically, without manual rethreading. Examples in vendor material: a 6-needle Brother PR680W, a 10-needle Brother PR1060W, a 7-needle Janome MB-7 [Source: search-aggregated vendor listings, e.g. Brother multi-needle machines](https://www.brother-usa.com/home/sewing-embroidery/multi-needle-machines). Higher-end multi-needle machines near the home segment, with up to **12, 15 or 20 needles**, exist in the broader market <!-- TBD: verify — 12/15/20-needle figures are commonly quoted in embroidery-business blogs but a specific current manufacturer spec sheet for a 15- or 20-needle single-head consumer/prosumer machine was not independently checked in this pass -->.
- **Industrial multi-head machines**: production floors increase capacity with more **heads**. Each head is a small multi-needle unit. All heads stitch the *same* design at the same time, one head for each garment or item. Tajima has offered 1, 2, 4, 6, 8, 10, 12, 15, 20 and 24-head machines. The needle count *for each head* changes with the model. Some listings show 12-head/12-needle configurations (one needle for each head), others 12-head/9-needle [Source: search-aggregated commercial embroidery machine listings, e.g. Tajima TEHX-C1212 and TME-DC912 listings on The Embroidery Warehouse](https://www.theembroiderywarehouse.com/xcart/tajima-tme-dc912-12-head-9-needle-commercial-embroidery-machine.html); Barudan has similar 12–15-head product lines [Source: Barudan America — 12-15-Head](https://www.barudanamerica.com/12-15-head/).
- On a multi-head machine, the needle count does **not** show the number of simultaneous colors across heads, as it does in one head. Needle *N* has the same color on each head. Thus all heads change to color *N* at the same time.

## Machine speed (stitches per minute)

The speed unit is stitches per minute (SPM). The speed changes with the machine tier, and with whether the value is a rated maximum or a sustained real-world average:

| Tier | Representative rated max SPM | Source |
|---|---|---|
| Home / entry-level (e.g. Tajima SAI) | ~800 SPM | [search-aggregated Tajima product summaries](https://www.truedigitizing.com/blog/tajima-embroidery-machines) |
| Multi-head commercial (e.g. Barudan 4-head) | ~1,100 SPM | [search-aggregated Barudan product summaries](https://www.maggieframes.com/blogs/embroidery-blogs/barudan-vs-tajima-2025-expert-comparison-for-machine-embroidery-professionals) |
| High-end industrial (e.g. Tajima TWMX-C1501, TMEZ-SC) | up to ~1,200 SPM | [search-aggregated Tajima product summaries](https://www.truedigitizing.com/blog/tajima-embroidery-machines) |
| High-end industrial (e.g. Barudan BEKT-S1501CBIII) | up to ~1,300 SPM | [search-aggregated Barudan product summaries](https://www.maggieframes.com/blogs/embroidery-blogs/barudan-vs-tajima-2025-expert-comparison-for-machine-embroidery-professionals) |

The rated maximum SPM is a ceiling, not a sustained production rate. Designs with dense fill, small satin columns or frequent color changes make the machine run slower. Models usually calculate the real-world *effective* throughput as the rated speed multiplied by an efficiency factor (frequently about 70–80%). This factor includes thread breaks, color changes and hooping downtime. For the production-time estimation model that uses this value, see `production.md` [Source: search-aggregated production-time calculators, e.g. TEX Inc — Calculating pieces per hour](https://tex-inc.com/blogs/digitizing-embroidery/maximizing-embroidery-production-calculating-pieces-per-hour-for-different-machine-types).

## Summary caveat

Do not hard-code a numeric range above as one universal constant. A validating pipeline must keep these values as **per-machine-profile** configuration: max stitch length, max jump length, trim-jump-count threshold, hoop inventory, needle/head count, rated SPM. It must not use the number of one brand as a global constraint.
