---
name: embroidery-digitize
model: inherit
effort: medium
agents: []
description: >
  Use to turn artwork or a design brief into a machine-readable stitch plan — a stitch type
  (satin/fill/running/motif), underlay, density, and pull compensation decided per design region —
  before any optimization or file export happens. Triggers on "digitize {design}", "turn this logo
  into a stitch plan", "digitize the artwork for {design}", "/sdd-emb:embroidery-digitize {design}",
  "оцифруй дизайн {design}", "перетвори лого в стібки". Reads
  docs/domain/embroidery/stitch-vocabulary.md and machine-constraints.md (if present) for
  vocabulary/defaults/limits, confirms target fabric/stabilizer/hoop with the user, decides a
  stitch type + underlay + density + pull compensation per region, and writes
  docs/embroidery/{design}/stitch-plan.md + stitch-plan.json. Standalone utility, not gated into
  the specify→...→ship backbone — a product feature that exposes "auto-digitize" as a capability
  still goes through that backbone, and its `implement` step calls into this skill.
---

# Skill: embroidery-digitize

Turns artwork or a design brief into a **stitch plan**: a per-region decision of stitch type,
underlay, density, and pull compensation, sequenced into color blocks — the first of the four
embroidery capability skills (`embroidery-digitize` → `embroidery-optimize` → `embroidery-export` →
`embroidery-qa`). This is domain computation, not process orchestration — it does not gate on or
replace any stage of the specify→design→...→ship backbone; a feature that ships "auto-digitize" as
a product capability is still built through that backbone, and its `implement` step calls into this
skill the way it would call into any other library.

This skill keeps only its own machinery. Question phrasing is **shared** →
[`../_shared/ask-style.md`](../_shared/ask-style.md). Document prose follows the project's
`artifact_language` setting — headings, frontmatter, and machine tokens stay English →
[`../_shared/artifact-language.md`](../_shared/artifact-language.md).

## Owner

The digitizer (human or the calling feature's implementer).

## Inputs

- `<design>` — kebab-case slug for this design.
- The artwork or design brief — an image reference, a written description, or an existing
  outline/vector the user supplies.
- Target fabric type + stabilizer choice, and target hoop — ask if not given; confirm against
  `docs/domain/embroidery/production.md` (stabilizer-by-fabric guidance) and
  `docs/domain/embroidery/machine-constraints.md` (hoop families) when present.
- (Optional) `docs/domain/embroidery/stitch-vocabulary.md` — stitch-type/underlay/density/pull-
  compensation vocabulary and starting-point ranges. Absent → proceed with the generic vocabulary
  below and say so in the handoff; **never invent a numeric default with no source** — ask instead.
- (Optional) `docs/domain/embroidery/machine-constraints.md` — stitch-length/jump limits to respect
  from the start (cheaper than discovering a violation at `embroidery-export`).
- (Optional) `CONTEXT.md` — read for canonical domain-term names already fixed for this project.

## Protocol

1. **Resolve the slug + read domain references.** If `docs/domain/embroidery/stitch-vocabulary.md`
   or `machine-constraints.md` exist, read them; note any `<!-- TBD: verify -->` markers so later
   steps don't silently trust an unverified number. Absent → proceed with the built-in generic
   vocabulary (satin / fill / running / motif; edge-walk / zigzag / lattice underlay) and flag the
   gap in the handoff.
2. **Capture intent.** One `AskUserQuestion` (phrasing per `../_shared/ask-style.md`) confirming the
   design's rough regions (e.g. "a text part, a border, a fill background — is that the shape of
   it?"), then confirm target fabric, stabilizer, and hoop. Offer the stabilizer-by-fabric mapping
   from `production.md` as the recommended option when available.
3. **Decide per region.** For each region, pick: **stitch type** (satin for text/borders/thin
   shapes; fill/tatami for broad areas; running for fine outline detail; motif for ornamental
   texture — never satin a region wide enough that coverage/snagging risk applies, per
   `stitch-vocabulary.md`'s width guidance), **underlay** (edge-walk for small objects/letters,
   zigzag under satin columns, lattice under fill — `<!-- N/A -->` only for very light running-
   stitch-only regions), **density** and **pull compensation** (from the domain doc's starting-point
   ranges if present; otherwise ask — never fabricate a number). Record the source of every number
   (`doc: <file>` or `user-confirmed`).
4. **Sequence into color blocks.** Order regions to minimize color/thread switching (group same-
   color regions), respecting any user-stated hard ordering (e.g. appliqué's placement → tack-down
   → trim → cover sequence from `stitch-vocabulary.md` must stay in that order). Underlay always
   precedes its region's top stitch.
5. **Write the stitch plan.** From [`./templates/stitch-plan.md`](./templates/stitch-plan.md):
   `docs/embroidery/<design>/stitch-plan.md` (human-readable, one section per region) +
   `docs/embroidery/<design>/stitch-plan.json` (machine-readable — contract below), the same
   human+machine pairing `tasks` uses for `tasks.json`.
6. **Self-check.** Re-read the written files: every region has a stitch type, an underlay decision
   (or an explicit `<!-- N/A: reason -->`), a density value, and a pull-compensation value, each
   with a recorded source; fabric/stabilizer/hoop are recorded; the color-block sequence respects
   any stated hard ordering.
7. **Handoff.** Propose commit `embroidery-digitize: <design> stitch plan`. Then **emit the
   stage-handoff block** per [`../_shared/handoff.md`](../_shared/handoff.md) — *Що я зробив* + *Перевір перед тим як продовжити*
   (`stitch-plan.md`, `stitch-plan.json`) + *Що далі*: `/sdd-emb:embroidery-optimize <design>` (or
   `/sdd-emb:embroidery-export <design>` directly for a design with only one or two regions).

## `stitch-plan.json` contract (read by `embroidery-optimize` / `embroidery-export` / `embroidery-qa`)

```json
{
  "design": "<design>",
  "fabric": "<fabric type>",
  "stabilizer": "<cutaway|tearaway|water-soluble|adhesive>",
  "hoop": "<nominal hoop size>",
  "regions": [
    {
      "id": "R1",
      "name": "descriptive name",
      "stitch_type": "satin|fill|running|motif",
      "underlay": "edge-walk|zigzag|lattice|none",
      "density": { "value": "<number>", "unit": "<unit>", "source": "doc:<file>|user-confirmed" },
      "pull_compensation": { "value": "<number>", "unit": "mm", "source": "doc:<file>|user-confirmed" },
      "color_block": "<color/thread id>",
      "sequence_order": 1
    }
  ]
}
```

- `regions` order in the file reflects the sequenced color-block order from step 4.
- Every numeric field carries a `source` — `embroidery-optimize`/`embroidery-export`/`embroidery-qa`
  treat a `doc:<file>` source as more trustworthy than `user-confirmed`-with-no-domain-doc, and both
  as more trustworthy than a value with no source (which should not exist after this skill's
  self-check).

## Definition of Done

- `docs/embroidery/<design>/stitch-plan.md` and `stitch-plan.json` exist and agree (same regions,
  same values — no translation-layer drift, the same discipline `tasks` uses for its two outputs).
- Every region has a stitch type, an underlay decision (or explicit N/A + reason), density, and pull
  compensation, each with a recorded source — no fabricated numbers.
- Fabric, stabilizer, and hoop are recorded.
- Any stated hard ordering (e.g. appliqué's phase sequence) is respected in `sequence_order`.
- Step 6's self-check is this skill's **structural self-check**
  ([`../_shared/self-check.md`](../_shared/self-check.md)); its result is reported in the handoff.

## Anti-patterns

- **Inventing a density/compensation number with no source** when the domain doc is absent or marks
  the figure `<!-- TBD -->` — ask the user instead of fabricating precision.
- **Satin-stitching an oversized region** instead of switching to fill once width exceeds the
  coverage/snagging threshold `stitch-vocabulary.md` describes.
- **Skipping underlay "to save time"** on a region that needs it (any satin column or fill area).
- **Reordering a stated hard sequence** (e.g. running appliqué's cover stitch before its tack-down).
- **Treating this skill as a backbone stage.** It never hard-refuses on a missing `spec.md`/`sad.md`
  — it's a standalone utility a feature's `implement` step or a user calls directly.

## References & template

- [`../_shared/ask-style.md`](../_shared/ask-style.md) — phrasing for the intent/region questions.
- [`../_shared/handoff.md`](../_shared/handoff.md) — the stage-handoff block this skill emits.
- [`../_shared/artifact-language.md`](../_shared/artifact-language.md) — prose language for
  `stitch-plan.md`; `stitch-plan.json` keys stay English.
- `docs/domain/embroidery/stitch-vocabulary.md` · `machine-constraints.md` · `production.md` — the
  domain reference packs this skill reads when present.
- [`./templates/stitch-plan.md`](./templates/stitch-plan.md) — output scaffold.
