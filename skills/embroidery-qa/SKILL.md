---
name: embroidery-qa
model: inherit
effort: medium
agents: []
description: >
  Use to validate a stitch plan or an exported machine file against domain quality rules before
  production: density in range, underlay where necessary, recorded pull compensation, hoop fit,
  jump/trim budget, color-stop count against needle count, and lettering minimums when a region is
  marked as small text. Triggers on "check {design} for problems", "QA the stitch plan for
  {design}", "validate {design} before production", "/sdd-emb:embroidery-qa {design}",
  "перевір {design} перед виробництвом". Reads docs/embroidery/{design}/stitch-plan.json (or the
  export-report.md of an exported file) and docs/domain/embroidery/{machine-constraints,
  stitch-vocabulary}.md. Runs a fixed rule set and writes a cited qa-report.md with a
  PASS / ISSUES-FOUND verdict. Read-only: it never edits the design. Hard-refuses if the design has
  no stitch plan and no export report.
---

# Skill: embroidery-qa

This skill is the read-only quality gate for an embroidery design. It is the last of the four
embroidery capability skills. It judges a stitch plan, or an exported file through its
`embroidery-export` report, against domain physics rules. It uses the same "cite or drop"
discipline as the `reviewer` agent for code: each finding names the region, the rule and the
concrete numbers, or the skill drops it. It never edits the design. The findings go back to
`embroidery-digitize` or `embroidery-optimize` for the repair.

The chat with the user is in Ukrainian → [`../_shared/chat-language.md`](../_shared/chat-language.md).
English prose (artifacts and reports) follows ASD-STE100 Simplified Technical English → [`../_shared/ste100.md`](../_shared/ste100.md).

## Owner

The digitizer, or the person who approves a design before it goes to production.

## Inputs

- `<design>` — the same slug that the embroidery-* chain uses.
- **Gate (hard-refuse if missing both):** `docs/embroidery/<design>/stitch-plan.json` and each
  `docs/embroidery/<design>/_export/export-report-*.md`. If neither exists, STOP and show: «спочатку запусти
  `embroidery-digitize <design>` — поки що нічого перевіряти».
- (Expected) `docs/domain/embroidery/machine-constraints.md` — the density, jump, hoop and
  needle-count limits. If it is absent, run the checks that do not need it (underlay present, pull
  compensation present, hard order obeyed). Say clearly which checks you skipped and why.
- (Expected) `docs/domain/embroidery/stitch-vocabulary.md` — the lettering and monogram minimums,
  for a region that is marked as small text.
- (Optional) a target machine profile (needle count, hoop inventory). If the user names one, check
  against it. If not, check against the generic ranges of the domain documents and say so.

## Rule set

Each rule below gives a finding only when it fires. The report does not mention each region that
passes all applicable rules. The coverage count of the report (§ Definition of Done) proves that
the skill checked it.

1. **Density in range.** The density of a region is in the cited range of the domain document for
   its stitch type. If it is outside the range, give a finding. Cite the region, the value and the
   range of the document.
2. **Underlay present where needed.** Each satin or fill region has an underlay decision, or an
   explicit `<!-- N/A: reason -->` from `embroidery-digitize`. A silent gap is a finding.
3. **Pull compensation recorded.** Each region with a density that causes a real distortion risk
   (per the guidance of the domain document) has a recorded pull-compensation value that is not zero.
4. **Hoop fit.** The extents of the design (from the stitch plan, or the measured extents of the
   export report) fit in the usable field of the declared hoop, not only in its nominal size.
5. **Jump/trim budget.** Report the total jump count and the forced-trim count against the guidance
   of the domain document. Use `optimization-report.md` if it exists. If not, calculate them again
   from the plan. Flag them only if they are much higher than an optimized plan for a design of this
   size. A design that did not go through `embroidery-optimize` gets a note, not an automatic fail.
6. **Color-stop vs. needle count.** If the needle count of the target machine is known, flag a
   design with more color blocks than needles. A manual re-thread will be necessary during the run.
   This is informational, not a hard fail. It is a hard fail only if the user stated that no manual
   work is acceptable for this run.
7. **Lettering minimums.** For each region tagged as small text or monogram, check the satin-column
   width, the letter height, the internal-counter size and the space between letters. Compare them
   with the cited ranges in `stitch-vocabulary.md`.
8. **Format round-trip (if an export report exists).** Also show each round-trip mismatch from
   `embroidery-export` as a QA finding. A design with a failed round-trip is not production-ready,
   also if it passes all other rules.

## Protocol

1. **Gate.** Make sure that `stitch-plan.json` or an `_export/export-report-*.md` exists. If
   neither exists, refuse with the message above. Read the domain documents that are present. Note
   which rules above will run in **skipped** mode because a document is missing.
2. **Run each applicable rule** from the Rule set against the plan (and the export report, if it is
   present). For each finding, cite the region id, the rule number, the actual value, and the
   threshold or range from the document.
3. **Classify the severity.**
   - `blocks-production`: a hard format or machine violation. Examples: a stitch longer than the
     length limit of the format that was not divided, or a failed round-trip.
   - `warning`: a violation of a domain-guidance range (density, lettering minimums, hoop fit).
   - `informational`: a needle-count re-thread note, or a jump count that is not optimized.
4. **Write the report.** Use [`./templates/qa-report.md`](./templates/qa-report.md) to write
   `docs/embroidery/<design>/_qa/qa-report-<date>.md`. Include each finding, cited per the rule set,
   the rules that ran in skipped mode and the reason, and a closing verdict.
5. **Verdict.**
   - `PASS` — zero `blocks-production` findings. The report can still list warnings and
     informational findings, with a note that the user acknowledged them.
   - `ISSUES-FOUND` — one or more `blocks-production` findings. For each finding, name the upstream
     skill that must repair it: `embroidery-digitize` for a content repair, `embroidery-optimize`
     for a sequence repair, `embroidery-export` for a new export.
6. **Self-check.** Make sure that each finding has a region id, a rule and a cited number. Before
   you write the report, drop each finding without a citation. `reviewer` uses the same discipline
   for code.
7. **Handoff.** Then **emit the stage-handoff block** per [`../_shared/handoff.md`](../_shared/handoff.md)
   — *Що я зробив* (verdict + finding count) + *Перевір перед тим як продовжити* (`qa-report.md`) + *Що далі*:
   - `PASS` → production-ready. Continue the work that you did before.
   - `ISSUES-FOUND` → for each finding, the named upstream skill. Then run `embroidery-qa <design>`
     again.

## Definition of Done

- `qa-report.md` exists with a `PASS` / `ISSUES-FOUND` verdict.
- Each finding cites a region id, a rule, an actual value and the threshold from the document. A
  finding without a citation is not in the report.
- The report states which rules ran in skipped mode (missing domain document). It does not silently
  omit them.
- Each `blocks-production` finding names the upstream skill that must repair it.
- The citation discipline of step 6 is the **structural self-check** of this skill
  ([`../_shared/self-check.md`](../_shared/self-check.md)). The handoff reports its result.

## Anti-patterns

- **A "this looks off" finding without a citation.** Cite the region, the rule and the numbers, or
  drop the finding. `reviewer` uses the same standard for code findings.
- **Using a domain number with a `<!-- TBD -->` source as an authoritative pass/fail line** without
  a flag that the threshold is not verified.
- **Silently skipping a rule** because its domain document is missing, without a note in the report.
  Say which rules did not run and why.
- **Editing the design to repair a finding.** This skill is read-only, the same as `reviewer`. It
  reports. The named upstream skill repairs.
- **A general PASS with open warnings that the user did not see.** List warnings and informational
  findings also on a PASS verdict. A clean report proves coverage, not silence.

## References & template

- `docs/domain/embroidery/machine-constraints.md` · `stitch-vocabulary.md` — the rule thresholds
  that this skill cites.
- [`../embroidery-export/templates/export-report.md`](../embroidery-export/templates/export-report.md)
  — the round-trip result that rule 8 of this skill reads when it is present.
- [`../_shared/ask-style.md`](../_shared/ask-style.md) · [`../_shared/handoff.md`](../_shared/handoff.md) · [`../_shared/artifact-language.md`](../_shared/artifact-language.md) · [`../_shared/ste100.md`](../_shared/ste100.md).
- [`./templates/qa-report.md`](./templates/qa-report.md) — the output scaffold.
