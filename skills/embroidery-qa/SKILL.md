---
name: embroidery-qa
model: inherit
effort: medium
agents: []
description: >
  Use to validate a stitch plan or an exported machine file against domain quality rules before
  production — density in range, underlay present where needed, pull compensation recorded, hoop
  fit, jump/trim budget, color-stop count vs. needle count, and lettering-specific minimums when a
  region is marked small text. Triggers on "check {design} for problems", "QA the stitch plan for
  {design}", "validate {design} before production", "/sdd-emb:embroidery-qa {design}",
  "перевір {design} перед виробництвом". Reads docs/embroidery/{design}/stitch-plan.json (or an
  exported file's export-report.md) plus docs/domain/embroidery/{machine-constraints,
  stitch-vocabulary}.md, runs a fixed rule set, and writes a cited qa-report.md with a
  PASS / ISSUES-FOUND verdict — read-only, it never edits the design. Hard-refuses if neither a
  stitch plan nor an export report exists for the design.
---

# Skill: embroidery-qa

The read-only quality gate for an embroidery design — the last of the four embroidery capability
skills. It judges a stitch plan (or an already-exported file, via its `embroidery-export` report)
against domain physics rules, the same "cite or drop" discipline the `reviewer` agent applies to
code: every finding names the region, the rule, and the concrete numbers involved, or it's dropped.
It never edits the design — findings route back to `embroidery-digitize` or `embroidery-optimize` to
fix.

## Owner

The digitizer, or whoever signs off before a design goes to production.

## Inputs

- `<design>` — the same slug used across the embroidery-* chain.
- **Gate (hard-refuse if missing both):** `docs/embroidery/<design>/stitch-plan.json` and any
  `docs/embroidery/<design>/_export/export-report-*.md`. Neither exists → STOP and point: «run
  `embroidery-digitize <design>` first — there is nothing to check yet».
- (Expected) `docs/domain/embroidery/machine-constraints.md` — density/jump/hoop/needle-count
  bounds. Absent → run the checks that don't need it (underlay presence, pull-compensation presence,
  hard-ordering respected) and say plainly which checks were skipped and why.
- (Expected) `docs/domain/embroidery/stitch-vocabulary.md` — lettering/monogram minimums, when a
  region is marked as small text.
- (Optional) a target machine profile (needle count, hoop inventory) — if the user names one, check
  against it; otherwise check against the generic ranges the domain docs give and say so.

## Rule set

Each rule below produces a finding only when it fires; a region that passes every applicable rule is
not mentioned individually — the report's coverage count (§ Definition of Done) proves it was
checked.

1. **Density in range.** A region's density falls within the domain doc's cited range for its stitch
   type; outside the range → finding, citing the region, the value, and the doc's range.
2. **Underlay present where needed.** Every satin or fill region has an underlay decision (or an
   explicit `<!-- N/A: reason -->` from `embroidery-digitize`) — a silent gap is a finding.
3. **Pull compensation recorded.** Any region dense enough to imply real distortion risk (per the
   domain doc's guidance) has a non-zero pull-compensation value recorded.
4. **Hoop fit.** The design's extents (from the stitch plan, or the export report's measured
   extents) fit within the declared hoop's usable field, not just its nominal size.
5. **Jump/trim budget.** Total jump count and forced-trim count (from `optimization-report.md` if it
   exists, else recomputed from the plan) are reported against the domain doc's guidance — flagged
   only if they're materially higher than what an optimized plan for a design of this size would
   suggest (a design that skipped `embroidery-optimize` gets a note, not an automatic fail).
6. **Color-stop vs. needle count.** If a target machine's needle count is known, flag when the
   design's color-block count exceeds it (a manual re-thread will be required mid-run) — informational,
   not a hard fail, unless the user stated no manual intervention is acceptable for this run.
7. **Lettering minimums.** For any region tagged as small text/monogram, check satin-column width,
   letter height, internal-counter size, and inter-letter spacing against `stitch-vocabulary.md`'s
   cited ranges.
8. **Format round-trip (if an export report exists).** Surface any `embroidery-export` round-trip
   mismatch as a QA finding too — a design isn't production-ready with a failed round-trip, even if
   every other rule passes.

## Protocol

1. **Gate.** Confirm at least one of `stitch-plan.json` / an `_export/export-report-*.md` exists;
   refuse with the pointer above if neither does. Read whichever domain docs are present; note which
   rules above will run in **skipped** mode due to a missing doc.
2. **Run every applicable rule** from the Rule set against the plan (and the export report, if
   present). For each finding: cite the region id, the rule number, the actual value, and the
   doc-cited threshold/range it's being checked against.
3. **Classify severity.** `blocks-production` (a hard format/machine violation, e.g. a stitch over
   the format's length limit slipped through, or a failed round-trip), `warning` (a domain-guidance
   range violation — density, lettering minimums, hoop fit), `informational` (needle-count re-thread
   note, an un-optimized jump count).
4. **Write the report.** [`./templates/qa-report.md`](./templates/qa-report.md) →
   `docs/embroidery/<design>/_qa/qa-report-<date>.md`: every finding cited per the rule set, the
   rules that ran in skipped mode and why, and a closing verdict.
5. **Verdict.** `PASS` — zero `blocks-production` findings (warnings/informational may still be
   listed, with the user's acknowledgement noted). `ISSUES-FOUND` — at least one `blocks-production`
   finding; name which upstream skill (`embroidery-digitize` for a content fix,
   `embroidery-optimize` for a sequencing fix, `embroidery-export` for a re-export) should address
   each one.
6. **Self-check.** Every finding traces to a region id + a rule + a cited number — an uncited
   finding is dropped before the report is written, the same discipline `reviewer` applies to code.
7. **Handoff.** Then **emit the stage-handoff block** per [`../_shared/handoff.md`](../_shared/handoff.md)
   — *What I did* (verdict + finding count) + *Review* (`qa-report.md`) + *Run next*: `PASS` →
   production-ready, resume whatever you were doing; `ISSUES-FOUND` → the named upstream skill for
   each finding, then re-run `embroidery-qa <design>`.

## Definition of Done

- `qa-report.md` exists with a `PASS` / `ISSUES-FOUND` verdict.
- Every finding cites a region id, a rule, an actual value, and the doc-cited threshold — an
  uncited finding does not appear in the report.
- The report states which rules ran in skipped mode (missing domain doc) rather than silently
  omitting them.
- A `blocks-production` finding names the upstream skill that should fix it.
- Step 6's citation discipline is this skill's **structural self-check**
  ([`../_shared/self-check.md`](../_shared/self-check.md)); its result is reported in the handoff.

## Anti-patterns

- **An uncited "this looks off" finding.** Cite the region + rule + numbers, or drop it — the same
  bar `reviewer` holds code findings to.
- **Treating a `<!-- TBD -->`-sourced domain number as an authoritative pass/fail line** without
  flagging that the threshold itself is unverified.
- **Silently skipping a rule** because its domain doc is missing, with no note in the report — say
  which rules didn't run and why.
- **Editing the design to fix a finding.** This skill is read-only, like `reviewer`; it reports, the
  named upstream skill fixes.
- **A blanket PASS with unresolved warnings the user never saw.** Warnings/informational findings
  are listed even on a PASS verdict — a clean report proves coverage, not silence.

## References & template

- `docs/domain/embroidery/machine-constraints.md` · `stitch-vocabulary.md` — the rule thresholds
  this skill cites.
- [`../embroidery-export/templates/export-report.md`](../embroidery-export/templates/export-report.md)
  — the round-trip result this skill's rule 8 reads when present.
- [`../_shared/ask-style.md`](../_shared/ask-style.md) · [`../_shared/handoff.md`](../_shared/handoff.md) · [`../_shared/artifact-language.md`](../_shared/artifact-language.md).
- [`./templates/qa-report.md`](./templates/qa-report.md) — output scaffold.
