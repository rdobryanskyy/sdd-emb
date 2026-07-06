---
name: embroidery-optimize
model: inherit
effort: medium
agents: []
description: >
  Use to reorder an existing stitch plan's sequence to reduce jump-stitch travel, forced trims, and
  color/thread changes — without altering what gets stitched. Triggers on "optimize the stitch order
  for {design}", "reduce jumps in {design}", "minimize trims for {design}",
  "/sdd-emb:embroidery-optimize {design}", "оптимізуй порядок стібків для {design}". Reads
  docs/embroidery/{design}/stitch-plan.json, reorders color blocks to minimize thread changes and
  regions within each block by spatial proximity to minimize jump travel, re-derives trim points
  against docs/domain/embroidery/machine-constraints.md's jump-threshold guidance, and writes an
  optimized stitch-plan.json plus an optimization-report.md with before/after metrics. Standalone
  utility in the embroidery-digitize → embroidery-optimize → embroidery-export → embroidery-qa
  chain; hard-refuses if stitch-plan.json is missing.
---

# Skill: embroidery-optimize

Reorders an already-digitized stitch plan to reduce **jump-stitch travel distance, forced-trim
count, and color/thread-change count** — the production-cost drivers `machine-constraints.md` and
`production.md` both name. It changes *sequence only*: the set of stitches and their per-region
decisions (stitch type, underlay, density, pull compensation) from `embroidery-digitize` are never
altered here.

This skill keeps only its own machinery. Question phrasing is **shared** →
[`../_shared/ask-style.md`](../_shared/ask-style.md). Prose follows `artifact_language` →
[`../_shared/artifact-language.md`](../_shared/artifact-language.md).

## Owner

The digitizer or the calling feature's implementer.

## Inputs

- `<design>` — the same slug `embroidery-digitize` used.
- **Gate (hard-refuse if missing):** `docs/embroidery/<design>/stitch-plan.json`. Absent → STOP and
  point: «run `embroidery-digitize <design>` first — there is no stitch plan to optimize».
- (Optional) `docs/domain/embroidery/machine-constraints.md` — the trim-after-N-consecutive-jumps
  convention and jump-length thresholds. Absent → use the generic "trim after 3–5 consecutive jumps"
  convention and say so in the report.
- Any user-stated hard ordering constraint already recorded in `stitch-plan.json`'s sequencing notes
  (e.g. appliqué phases) — **never reordered**, regardless of what the optimizer would otherwise pick.

## Protocol

1. **Gate.** `test -f docs/embroidery/<design>/stitch-plan.json` → missing = refuse with the pointer
   above.
2. **Baseline metrics.** From the current `regions` order, compute: total color/thread changes,
   total jump count, total jump travel distance (straight-line, region-centroid to region-centroid,
   since exact stitch coordinates don't exist yet at this stage), and the trim count implied by the
   jump-threshold convention.
3. **Group by color block, preserving hard order.** Partition regions into their existing
   `color_block` groups; **any region pair with a recorded hard-ordering constraint stays in that
   relative order** inside its group — the optimizer only reorders what's free to move.
4. **Order color blocks to minimize thread changes.** Where the design allows (no hard cross-block
   ordering constraint), group blocks so identical colors that appear more than once end up
   adjacent, cutting redundant re-threads.
5. **Order regions within each block by proximity.** Apply a nearest-neighbor heuristic over region
   centroids to minimize total jump travel within the block.
6. **Re-derive trim points.** Walk the new order; insert a trim wherever the consecutive-jump count
   reaches the configured (or default) threshold, or wherever a jump's implied travel exceeds the
   format-agnostic jump-length guidance in `machine-constraints.md`.
7. **After metrics + delta report.** Recompute the same four metrics from step 2; write
   `docs/embroidery/<design>/optimization-report.md` from
   [`./templates/optimization-report.md`](./templates/optimization-report.md) — before vs. after,
   and an honest note if a metric didn't improve (e.g. a hard ordering constraint prevented a
   shorter path) rather than silently omitting it.
8. **Write the optimized plan.** `docs/embroidery/<design>/stitch-plan.json` — same region content
   (stitch type / underlay / density / pull compensation / `id` unchanged), only `sequence_order`
   and `color_block` grouping updated. `stitch-plan.md` is regenerated to match.
9. **Self-check.** Diff the region *content* (everything except `sequence_order`) between the input
   and output JSON — it must be byte-identical; only ordering may differ. Verify every hard-ordering
   constraint from the input still holds in the output.
10. **Handoff.** Propose commit `embroidery-optimize: <design> (Δjumps, Δtrims, Δcolor-changes)`.
    Then **emit the stage-handoff block** per [`../_shared/handoff.md`](../_shared/handoff.md) —
    *What I did* (the before/after metrics) + *Review* (`optimization-report.md`, `stitch-plan.json`)
    + *Run next*: `/sdd-emb:embroidery-export <design>` or `/sdd-emb:embroidery-qa <design>`.

## Definition of Done

- `optimization-report.md` exists with before/after counts for color changes, jumps, trims, and
  travel distance — including an honest note wherever a metric didn't improve.
- The optimized `stitch-plan.json` has **identical region content** to the input (verified, not
  assumed) — only sequence changed.
- Every hard-ordering constraint from the input is preserved in the output.
- Step 9's content-diff self-check is this skill's **structural self-check**
  ([`../_shared/self-check.md`](../_shared/self-check.md)); its result is reported in the handoff.

## Anti-patterns

- **Reordering across a stated hard constraint** (e.g. an appliqué's tack-down before its placement
  line) to shave a fraction of travel distance — never legal, regardless of the metric gain.
- **Changing region content while "optimizing."** This skill touches sequence only; a density or
  stitch-type change belongs to `embroidery-digitize`, re-run deliberately, not smuggled in here.
- **Reporting only the metrics that improved.** A trade-off (fewer color changes, more jump travel)
  is reported both ways, not cherry-picked.
- **Guessing a jump-length threshold with no source** when `machine-constraints.md` is absent —
  state the generic default used, plainly, in the report.

## References & template

- [`../_shared/ask-style.md`](../_shared/ask-style.md) · [`../_shared/handoff.md`](../_shared/handoff.md) · [`../_shared/artifact-language.md`](../_shared/artifact-language.md).
- `docs/domain/embroidery/machine-constraints.md` — jump/trim threshold guidance this skill reads.
- [`./templates/optimization-report.md`](./templates/optimization-report.md) — before/after report scaffold.
