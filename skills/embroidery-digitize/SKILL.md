---
name: embroidery-digitize
model: inherit
effort: medium
agents: []
description: >
  Use to change artwork or a design brief into a machine-readable stitch plan. The stitch plan sets
  a stitch type (satin/fill/running/motif), underlay, density and pull compensation for each design
  region, before optimization or file export. Triggers on "digitize {design}", "turn this logo into
  a stitch plan", "digitize the artwork for {design}", "/sdd-emb:embroidery-digitize {design}",
  "оцифруй дизайн {design}", "перетвори лого в стібки". Reads
  docs/domain/embroidery/stitch-vocabulary.md and machine-constraints.md (if present) for
  vocabulary, defaults and limits. Confirms the target fabric, stabilizer and hoop with the user.
  Sets a stitch type, underlay, density and pull compensation for each region, and writes
  docs/embroidery/{design}/stitch-plan.md + stitch-plan.json. Standalone utility, not gated into
  the specify→...→ship backbone. A product feature that gives "auto-digitize" as a capability
  still goes through that backbone, and its `implement` step calls this skill.
---

# Skill: embroidery-digitize

This skill changes artwork or a design brief into a **stitch plan**. A stitch plan sets the stitch
type, underlay, density and pull compensation for each region, in a sequence of color blocks. This
is the first of the four embroidery capability skills (`embroidery-digitize` →
`embroidery-optimize` → `embroidery-export` → `embroidery-qa`). It is domain computation, not
process orchestration. It does not gate or replace a stage of the specify→design→...→ship backbone.
A feature that ships "auto-digitize" as a product capability still goes through that backbone. Its
`implement` step calls this skill, the same as it calls a library.

This skill keeps only its own machinery. The question phrasing is **shared** →
[`../_shared/ask-style.md`](../_shared/ask-style.md). The document prose follows the
`artifact_language` setting of the project. Headings, frontmatter and machine tokens stay in
English → [`../_shared/artifact-language.md`](../_shared/artifact-language.md). The chat with the
user is in Ukrainian → [`../_shared/chat-language.md`](../_shared/chat-language.md).
English prose (artifacts and reports) follows ASD-STE100 Simplified Technical English → [`../_shared/ste100.md`](../_shared/ste100.md).

## Owner

The digitizer (a human, or the implementer of the calling feature).

## Inputs

- `<design>` — kebab-case slug for this design.
- The artwork or design brief — an image reference, a written description, or an existing
  outline or vector that the user supplies.
- The target fabric type, the stabilizer and the target hoop. If the user did not give them, ask.
  If these files exist, compare the answers with `docs/domain/embroidery/production.md`
  (stabilizer-by-fabric guidance) and `docs/domain/embroidery/machine-constraints.md` (hoop families).
- (Optional) `docs/domain/embroidery/stitch-vocabulary.md` — the vocabulary for stitch type,
  underlay, density and pull compensation, and the starting-point ranges. If it is absent, continue
  with the generic vocabulary below and say so in the handoff. **Never invent a numeric default
  without a source.** Ask the user.
- (Optional) `docs/domain/embroidery/machine-constraints.md` — the stitch-length and jump limits to
  obey from the start. This costs less than a violation that `embroidery-export` finds later.
- (Optional) `CONTEXT.md` — read it for the canonical domain-term names of this project.

## Protocol

1. **Resolve the slug and read the domain references.** If
   `docs/domain/embroidery/stitch-vocabulary.md` or `machine-constraints.md` exist, read them. Note
   each `<!-- TBD: verify -->` marker, so that later steps do not trust an unverified number. If
   they are absent, continue with the built-in generic vocabulary (satin / fill / running / motif;
   edge-walk / zigzag / lattice underlay). Flag the gap in the handoff.
2. **Capture intent.** Ask one `AskUserQuestion` (phrasing per `../_shared/ask-style.md`) to confirm
   the approximate regions of the design. Example: "a text part, a border, a fill background — is
   that the shape of it?". Then confirm the target fabric, stabilizer and hoop. If `production.md`
   is available, give its stabilizer-by-fabric mapping as the recommended option.
3. **Decide for each region.** For each region, select these values:
   - **Stitch type:** satin for text, borders and thin shapes; fill/tatami for broad areas; running
     for fine outline detail; motif for ornamental texture. Never use satin for a region that is
     wide enough to cause a coverage or snagging risk (see the width guidance in
     `stitch-vocabulary.md`).
   - **Underlay:** edge-walk for small objects and letters, zigzag under satin columns, lattice
     under fill. Use `<!-- N/A -->` only for very light regions with only running stitch.
   - **Density** and **pull compensation:** use the starting-point ranges of the domain document if
     it is present. If not, ask the user. Never invent a number.

   Record the source of each number (`doc: <file>` or `user-confirmed`).
4. **Put the regions in a sequence of color blocks.** Put the regions in an order that gives the
   smallest number of color and thread changes (group the regions that have the same color). Obey
   each hard order that the user gave. For example, the appliqué sequence placement → tack-down →
   trim → cover from `stitch-vocabulary.md` must stay in that order. The underlay of a region is
   always before the top stitch of that region.
5. **Write the stitch plan.** Use [`./templates/stitch-plan.md`](./templates/stitch-plan.md) to
   write `docs/embroidery/<design>/stitch-plan.md` (human-readable, one section for each region) and
   `docs/embroidery/<design>/stitch-plan.json` (machine-readable, see the contract below). This is
   the same human and machine pair that `tasks` uses for `tasks.json`.
6. **Self-check.** Read the written files again. Make sure that:
   - each region has a stitch type, an underlay decision (or an explicit `<!-- N/A: reason -->`), a
     density value and a pull-compensation value, each with a recorded source;
   - the fabric, stabilizer and hoop are recorded;
   - the color-block sequence obeys each stated hard order.
7. **Handoff.** Propose the commit `embroidery-digitize: <design> stitch plan`. Then **emit the
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

- The order of `regions` in the file is the color-block order from step 4.
- Each numeric field has a `source`. `embroidery-optimize`/`embroidery-export`/`embroidery-qa` trust
  a `doc:<file>` source more than a `user-confirmed` source without a domain document. They trust
  both more than a value without a source. After the self-check of this skill, a value without a
  source must not exist.

## Definition of Done

- `docs/embroidery/<design>/stitch-plan.md` and `stitch-plan.json` exist and agree: the same regions
  and the same values. There is no drift between the two files. `tasks` uses the same discipline for
  its two outputs.
- Each region has a stitch type, an underlay decision (or an explicit N/A + reason), density and
  pull compensation, each with a recorded source. There are no invented numbers.
- The fabric, stabilizer and hoop are recorded.
- `sequence_order` obeys each stated hard order (for example, the phase sequence of appliqué).
- The self-check of step 6 is the **structural self-check** of this skill
  ([`../_shared/self-check.md`](../_shared/self-check.md)). The handoff reports its result.

## Anti-patterns

- **Inventing a density or compensation number without a source** when the domain document is
  absent or marks the value `<!-- TBD -->`. Ask the user. Do not invent precision.
- **Satin-stitching an oversized region.** When the width is more than the coverage/snagging
  threshold in `stitch-vocabulary.md`, use fill.
- **Skipping underlay "to save time"** on a region that needs it (each satin column or fill area).
- **Reordering a stated hard sequence** (for example, the cover stitch of an appliqué before its
  tack-down).
- **Treating this skill as a backbone stage.** It never hard-refuses because `spec.md`/`sad.md` is
  missing. It is a standalone utility. The `implement` step of a feature or a user calls it directly.

## References & template

- [`../_shared/ask-style.md`](../_shared/ask-style.md) — the phrasing for the intent and region questions.
- [`../_shared/handoff.md`](../_shared/handoff.md) — the stage-handoff block that this skill emits.
- [`../_shared/artifact-language.md`](../_shared/artifact-language.md) — the prose language of
  `stitch-plan.md`. The `stitch-plan.json` keys stay in English.
- [`../_shared/ste100.md`](../_shared/ste100.md) — the writing standard for English prose.
- `docs/domain/embroidery/stitch-vocabulary.md` · `machine-constraints.md` · `production.md` — the
  domain reference packs that this skill reads when they are present.
- [`./templates/stitch-plan.md`](./templates/stitch-plan.md) — the output scaffold.
