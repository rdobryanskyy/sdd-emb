---
name: embroidery-export
model: inherit
effort: medium
agents: []
description: >
  Use to serialize a stitch plan into one or more machine embroidery file formats (DST, PES, EXP,
  JEF, VP3, HUS, XXX, ART, EMB), validating every stitch against machine/format limits before
  writing and round-trip-verifying the result after. Triggers on "export {design} to DST",
  "generate the stitch file for {design}", "produce a PES for {design}",
  "/sdd-emb:embroidery-export {design} {format}", "експортуй {design} у DST". Reads
  docs/embroidery/{design}/stitch-plan.json plus docs/domain/embroidery/{file-formats,
  machine-constraints}.md, actually serializes via a real open-source embroidery library
  (pyembroidery is the documented default — never hand-computed byte fabrication), re-parses the
  output to confirm stitch count/color-stop count/extents round-trip, and writes an
  export-report.md. If the target feature also declared an "Embroidery file export" api contract
  (docs/features/{slug}/contracts/embroidery-export.md), validates against it too. Hard-refuses if
  stitch-plan.json is missing; degrades honestly (no fabricated file) if no serialization library is
  available in the environment.
---

# Skill: embroidery-export

Serializes a stitch plan into real machine-format bytes and proves the result is correct — the
third of the four embroidery capability skills. This is the one skill in the family that must touch
real tooling: a stitch file is a precise binary/structured artifact, not prose, so this skill runs
actual serialization code rather than reasoning its way to bytes. **If it cannot verify the export,
it says so — it never hands over a file it has not round-trip-checked.**

## Owner

The digitizer or the calling feature's implementer.

## Inputs

- `<design>` — the same slug used by `embroidery-digitize` / `embroidery-optimize`.
- **Gate (hard-refuse if missing):** `docs/embroidery/<design>/stitch-plan.json`. Absent → STOP and
  point: «run `embroidery-digitize <design>` first».
- `<format(s)>` — one or more of DST / PES / EXP / JEF / VP3 / HUS / XXX / ART / EMB. Ask if not
  given.
- **Strongly expected:** `docs/domain/embroidery/file-formats.md` (per-format encoding model, hard
  stitch-length limits, which formats have an independent open-source parser) and
  `docs/domain/embroidery/machine-constraints.md` (generic stitch/jump-length bounds). Absent →
  proceed using only the fixed hard limits documented inline below (§ Fallback bounds) and say so
  loudly in the report — narrower coverage, not a refusal.
- (Optional) `docs/features/<slug>/contracts/embroidery-export.md` — if the calling feature declared
  this as its `api` contract kind, validate the export against its per-field bounds table too.
- Recommended but not required: `docs/embroidery/<design>/optimization-report.md` — exporting an
  un-optimized plan is legal (warn that jump/trim counts weren't minimized), never blocked.

## Fallback bounds (used only when `machine-constraints.md` is absent)

- Max single-stitch/jump delta: **12.1 mm** (ternary coding, e.g. DST/Tajima) or **12.7 mm** (binary
  coding, e.g. EXP/Barudan) — split any longer stitch into multiple jump records.
- Forced trim after **3–5 consecutive jumps** (generic convention; a real machine profile overrides
  this).

## Protocol

1. **Gate.** `test -f docs/embroidery/<design>/stitch-plan.json` → missing = refuse with the pointer
   above. Read the target format(s); read the domain docs if present (note any
   `<!-- TBD: verify -->` markers on numbers this export would rely on, and flag them in the report
   rather than silently trusting them as hard truths).
2. **Pre-export validation.** Before writing anything: for every stitch implied by the plan's
   regions, check it against the target format's max stitch/jump delta (split oversized moves into
   multiple jumps, per that format's documented convention in `file-formats.md`); check the design's
   overall extents against the declared hoop; check the color-block count against the target
   machine's needle count if one is specified (flag — don't block — if a manual re-thread will be
   needed mid-run).
3. **Determine the serialization approach.** Check whether a suitable open-source embroidery library
   is available (or installable) in this environment — **`pyembroidery`** is the documented default
   (it normalizes ~40 formats to a common command set: `STITCH`/`JUMP`/`TRIM`/`STOP`/`END`/
   `COLOR_CHANGE`/`SEQUIN_MODE`/`SEQUIN_EJECT`, per `file-formats.md`). If available: write a small,
   throwaway script that builds the stitch/command stream from `stitch-plan.json` and calls the
   library's writer for the target format(s). **If no such library is available or installable
   here**, do **not** fabricate file bytes by reasoning alone — write the export as a precise,
   human-readable "export specification" (the exact command stream the format should contain) and
   state plainly in the report that no binary file was produced and why.
4. **Round-trip validate.** For every format actually written: re-parse the produced file (same
   library) and compare against the source plan — stitch count matches, color-stop/color-block count
   matches, and design extents match within a small tolerance. A mismatch is a real bug — fix the
   serialization and re-validate, never ship a file that fails its own round-trip.
5. **Contract check (if applicable).** If `docs/features/<slug>/contracts/embroidery-export.md`
   exists, check the produced file's fields/bounds against its table; report any drift the same way
   `api`'s own drift check does — surfaced, not silently resolved.
6. **Write the report.** `docs/embroidery/<design>/_export/export-report-<format>-<date>.md` from
   [`./templates/export-report.md`](./templates/export-report.md): target format, pre-export
   validation results (splits performed, hoop-fit check, needle-count flag), whether a real
   serialization library was used or the fallback spec-only path was taken, the round-trip result,
   and every domain-doc `<!-- TBD -->` this export leaned on.
7. **Self-check.** The round-trip validation in step 4 **is** this skill's structural self-check — a
   file that hasn't round-tripped is not considered exported, it's considered a draft.
8. **Handoff.** Propose commit `embroidery-export: <design> (<format list>)`. Then **emit the
   stage-handoff block** per [`../_shared/handoff.md`](../_shared/handoff.md) — *What I did* (formats
   produced, round-trip result) + *Review* (the exported file(s), `export-report.md`) + *Run next*:
   `/sdd-emb:embroidery-qa <design>` before treating the design as production-ready.

## Definition of Done

- For every requested format, either a real exported file exists **and has round-tripped clean**, or
  the report explicitly states no binary was produced and why (no library available) — never a
  silently fabricated file.
- No stitch in the output exceeds the target format's hard length limit — oversized moves were split
  into jumps, not truncated or silently allowed through.
- Every domain-doc `<!-- TBD -->` this export relied on is named in the report, not silently trusted.
- `export-report.md` exists per format/date and documents the validation + round-trip results.
- Step 4's round-trip check is this skill's **structural self-check**
  ([`../_shared/self-check.md`](../_shared/self-check.md)); its result is reported in the handoff.

## Anti-patterns

- **Fabricating file bytes by reasoning about the format's structure** instead of running real
  serialization code. A stitch file is a precise artifact; "this should be roughly right" is not
  export, it's a guess with a `.dst` extension.
- **Skipping the round-trip check** because the serialization "should be fine" — the exact failure
  mode this step exists to catch, mirroring `api`'s own "a clean 4/4 ✓ is cheap; a silent ✗ in prod
  is not" discipline.
- **Treating a `<!-- TBD -->`-marked machine-constraints number as a verified hard limit** without
  flagging it — an unverified 12.1 mm could be wrong for the actual target machine.
- **Truncating or silently dropping an oversized stitch** instead of splitting it into the
  format-correct multi-jump sequence.
- **Claiming success on the fallback spec-only path.** If no library was available, the handoff must
  say so plainly — never imply a binary file exists when only a specification does.

## References & template

- `docs/domain/embroidery/file-formats.md` — per-format encoding, hard limits, and which open-source
  libraries (pyembroidery, libembroidery, Ink/Stitch) can read/write each one.
- `docs/domain/embroidery/machine-constraints.md` — generic stitch/jump-length and trim-threshold
  guidance used when a specific machine profile isn't given.
- [`../api/templates/embroidery-export.md`](../api/templates/embroidery-export.md) — the **contract**
  (spec-time document) this skill's actual output is checked against, when the calling feature
  declared one.
- [`../_shared/ask-style.md`](../_shared/ask-style.md) · [`../_shared/handoff.md`](../_shared/handoff.md) · [`../_shared/artifact-language.md`](../_shared/artifact-language.md).
- [`./templates/export-report.md`](./templates/export-report.md) — output scaffold.
