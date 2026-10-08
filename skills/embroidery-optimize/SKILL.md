---
name: embroidery-optimize
model: inherit
effort: medium
agents: []
description: >
  Use to change the sequence of an existing stitch plan to decrease jump-stitch travel, forced
  trims and color/thread changes. It does not change what is stitched. Triggers on "optimize the
  stitch order for {design}", "reduce jumps in {design}", "minimize trims for {design}",
  "/sdd-emb:embroidery-optimize {design}", "оптимізуй порядок стібків для {design}". Reads
  docs/embroidery/{design}/stitch-plan.json. Puts the color blocks in an order with the fewest
  thread changes, and the regions in each block in order of spatial proximity to decrease jump
  travel. Calculates the trim points again against the jump-threshold guidance in
  docs/domain/embroidery/machine-constraints.md. Writes an optimized stitch-plan.json and an
  optimization-report.md with before/after metrics. Standalone utility in the
  embroidery-digitize → embroidery-optimize → embroidery-export → embroidery-qa chain. Hard-refuses
  if stitch-plan.json is missing.
---

# Skill: embroidery-optimize

This skill changes the order of a digitized stitch plan. It decreases the **jump-stitch travel
distance, the number of forced trims and the number of color/thread changes**. `machine-constraints.md`
and `production.md` both name these values as the production-cost drivers. The skill changes
*only the sequence*. It never changes the set of stitches or the region decisions from
`embroidery-digitize` (stitch type, underlay, density, pull compensation).

This skill keeps only its own machinery. The question phrasing is **shared** →
[`../_shared/ask-style.md`](../_shared/ask-style.md). The prose follows `artifact_language` →
[`../_shared/artifact-language.md`](../_shared/artifact-language.md). The chat with the user is in
Ukrainian → [`../_shared/chat-language.md`](../_shared/chat-language.md).
English prose (artifacts and reports) follows ASD-STE100 Simplified Technical English → [`../_shared/ste100.md`](../_shared/ste100.md).

## Owner

The digitizer, or the implementer of the calling feature.

## Inputs

- `<design>` — the same slug that `embroidery-digitize` used.
- **Gate (hard-refuse if missing):** `docs/embroidery/<design>/stitch-plan.json`. If it is absent,
  STOP and show: «спочатку запусти `embroidery-digitize <design>` — немає плану стібків для оптимізації».
- (Optional) `docs/domain/embroidery/machine-constraints.md` — the trim-after-N-consecutive-jumps
  convention and the jump-length thresholds. If it is absent, use the generic convention "trim after
  3–5 consecutive jumps" and say so in the report.
- Each hard order constraint from the user that the sequencing notes of `stitch-plan.json` already
  record (for example, appliqué phases). **Never change this order**, also when the optimizer would
  select a different order.

## Protocol

1. **Gate.** Run `test -f docs/embroidery/<design>/stitch-plan.json`. If the file is missing, refuse
   with the message above.
2. **Baseline metrics.** From the current order of `regions`, calculate these values:
   - the total number of color/thread changes;
   - the total number of jumps;
   - the total jump travel distance (a straight line from region centroid to region centroid,
     because exact stitch coordinates do not exist yet at this stage);
   - the number of trims that the jump-threshold convention gives.
3. **Group by color block, and keep the hard order.** Divide the regions into their existing
   `color_block` groups. **Each region pair with a recorded hard order constraint stays in that
   relative order** in its group. The optimizer changes only the order of regions that are free to
   move.
4. **Put the color blocks in the order with the fewest thread changes.** If the design permits it
   (no hard order constraint between blocks), put the blocks that have the same color next to each
   other. This removes unnecessary re-threads.
5. **Put the regions in each block in order of proximity.** Use a nearest-neighbor heuristic on the
   region centroids to decrease the total jump travel in the block.
6. **Calculate the trim points again.** Go through the new order. Add a trim where the number of
   consecutive jumps reaches the configured (or default) threshold. Also add a trim where the travel
   of a jump is more than the format-agnostic jump-length guidance in `machine-constraints.md`.
7. **After metrics and delta report.** Calculate the same four metrics from step 2 again. Use
   [`./templates/optimization-report.md`](./templates/optimization-report.md) to write
   `docs/embroidery/<design>/optimization-report.md` — before and after. If a metric did not
   improve, write an honest note (for example, a hard order constraint prevented a shorter path).
   Do not silently omit it.
8. **Write the optimized plan.** Write `docs/embroidery/<design>/stitch-plan.json`. The region
   content (stitch type / underlay / density / pull compensation / `id`) does not change. Only
   `sequence_order` and the `color_block` groups change. Make `stitch-plan.md` again, so that it
   agrees with the JSON.
9. **Self-check.** Compare the region *content* (all fields except `sequence_order`) of the input
   JSON and the output JSON. It must be byte-identical. Only the order can be different. Make sure
   that each hard order constraint from the input is still correct in the output.
10. **Handoff.** Propose the commit `embroidery-optimize: <design> (Δjumps, Δtrims, Δcolor-changes)`.
    Then **emit the stage-handoff block** per [`../_shared/handoff.md`](../_shared/handoff.md) —
    *Що я зробив* (the before/after metrics) + *Перевір перед тим як продовжити* (`optimization-report.md`, `stitch-plan.json`)
    + *Що далі*: `/sdd-emb:embroidery-export <design>` or `/sdd-emb:embroidery-qa <design>`.

## Definition of Done

- `optimization-report.md` exists, with before/after counts for color changes, jumps, trims and
  travel distance. It has an honest note for each metric that did not improve.
- The optimized `stitch-plan.json` has **the same region content** as the input. This is verified,
  not assumed. Only the sequence changed.
- The output keeps each hard order constraint from the input.
- The content-diff self-check of step 9 is the **structural self-check** of this skill
  ([`../_shared/self-check.md`](../_shared/self-check.md)). The handoff reports its result.

## Anti-patterns

- **Reordering across a stated hard constraint** to decrease the travel distance by a small
  quantity (for example, the tack-down of an appliqué before its placement line). This is never
  permitted, whatever the metric gain.
- **Changing region content during "optimization".** This skill changes only the sequence. A change
  of density or stitch type belongs to `embroidery-digitize`. Run that skill again deliberately. Do
  not hide the change here.
- **Reporting only the metrics that improved.** Report a trade-off in the two directions (for
  example, fewer color changes and more jump travel). Do not select only the good values.
- **Guessing a jump-length threshold without a source** when `machine-constraints.md` is absent.
  Write clearly in the report which generic default you used.

## References & template

- [`../_shared/ask-style.md`](../_shared/ask-style.md) · [`../_shared/handoff.md`](../_shared/handoff.md) · [`../_shared/artifact-language.md`](../_shared/artifact-language.md) · [`../_shared/ste100.md`](../_shared/ste100.md).
- `docs/domain/embroidery/machine-constraints.md` — the jump/trim threshold guidance that this skill reads.
- [`./templates/optimization-report.md`](./templates/optimization-report.md) — the before/after report scaffold.
