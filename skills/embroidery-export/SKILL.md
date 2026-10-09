---
name: embroidery-export
model: inherit
effort: medium
agents: []
description: >
  Use to serialize a stitch plan into one or more machine embroidery file formats (DST, PES, EXP,
  JEF, VP3, HUS, XXX, ART, EMB). It validates each stitch against the machine and format limits
  before it writes, and it does a round-trip check of the result after. Triggers on "export
  {design} to DST", "generate the stitch file for {design}", "produce a PES for {design}",
  "/sdd-emb:embroidery-export {design} {format}", "експортуй {design} у DST". Reads
  docs/embroidery/{design}/stitch-plan.json and docs/domain/embroidery/{file-formats,
  machine-constraints}.md. Serializes with a real open-source embroidery library (pyembroidery is
  the documented default; never bytes calculated by hand). Parses the output again to confirm that
  the stitch count, the color-stop count and the extents agree, and writes an export-report.md. If
  the target feature also declared an "Embroidery file export" api contract
  (docs/features/{slug}/contracts/embroidery-export.md), it also validates against that contract.
  Hard-refuses if stitch-plan.json is missing. If no serialization library is available in the
  environment, it says so honestly and makes no invented file.
---

# Skill: embroidery-export

This skill serializes a stitch plan into real machine-format bytes and proves that the result is
correct. It is the third of the four embroidery capability skills. It is the only skill in the
family that must use real tooling. A stitch file is a precise binary or structured artifact, not
prose. Thus this skill runs actual serialization code. It does not get the bytes from reasoning.
**If it cannot verify the export, it says so. It never gives a file that did not pass the
round-trip check.**

The chat with the user is in Ukrainian → [`../_shared/chat-language.md`](../_shared/chat-language.md).
English prose (artifacts and reports) follows ASD-STE100 Simplified Technical English → [`../_shared/ste100.md`](../_shared/ste100.md).

## Owner

The digitizer, or the implementer of the calling feature.

## Inputs

- `<design>` — the same slug that `embroidery-digitize` / `embroidery-optimize` used.
- **Gate (hard-refuse if missing):** `docs/embroidery/<design>/stitch-plan.json`. If it is absent,
  STOP and show: «спочатку запусти `embroidery-digitize <design>`».
- `<format(s)>` — one or more of DST / PES / EXP / JEF / VP3 / HUS / XXX / ART / EMB. If the user
  did not give them, ask.
- **Strongly expected:**
  - `docs/domain/embroidery/file-formats.md` — the encoding model of each format, the hard
    stitch-length limits, and the formats that have an independent open-source parser.
  - `docs/domain/embroidery/machine-constraints.md` — the generic stitch-length and jump-length
    limits.

  If they are absent, continue with only the fixed hard limits in § Fallback bounds below. Say this
  clearly in the report. The coverage is narrower, but this is not a refusal.
- (Optional) `docs/features/<slug>/contracts/embroidery-export.md` — if the calling feature declared
  this as its `api` contract kind, also validate the export against its table of field limits.
- Recommended, not necessary: `docs/embroidery/<design>/optimization-report.md`. The export of a
  plan that is not optimized is permitted. Warn that the jump and trim counts are not at their
  minimum. Never block it.

## Fallback bounds (used only when `machine-constraints.md` is absent)

- Maximum delta of one stitch or jump: **12.1 mm** (ternary coding, for example DST/Tajima) or
  **12.7 mm** (binary coding, for example EXP/Barudan). Divide each longer stitch into more than
  one jump record.
- Forced trim after **3–5 consecutive jumps** (a generic convention; a real machine profile has
  priority over it).

## Protocol

1. **Gate.** Run `test -f docs/embroidery/<design>/stitch-plan.json`. If the file is missing,
   refuse with the message above. Read the target formats. If the domain documents are present,
   read them. Note each `<!-- TBD: verify -->` marker on a number that this export uses. Flag it in
   the report. Do not silently trust it as a hard fact.
2. **Pre-export validation.** Before you write a file, do these checks:
   - Compare each stitch that the plan regions imply with the maximum stitch/jump delta of the
     target format. Divide each move that is too long into more than one jump, as `file-formats.md`
     documents for that format.
   - Compare the overall extents of the design with the declared hoop.
   - If the target machine has a specified needle count, compare the number of color blocks with
     it. If a manual re-thread will be necessary during the run, flag it. Do not block.
3. **Select the serialization approach.** Find out if a suitable open-source embroidery library is
   available (or can be installed) in this environment. **`pyembroidery`** is the documented
   default. It changes about 40 formats into one command set: `STITCH`/`JUMP`/`TRIM`/`STOP`/`END`/
   `COLOR_CHANGE`/`SEQUIN_MODE`/`SEQUIN_EJECT`, per `file-formats.md`.
   - If a library is available, write a small temporary script. The script makes the stitch and
     command stream from `stitch-plan.json` and calls the writer of the library for each target
     format.
   - **If no library is available or can be installed here**, do **not** invent file bytes from
     reasoning. Write the export as a precise, human-readable "export specification": the exact
     command stream that the format must contain. In the report, state clearly that no binary file
     was made, and give the reason.
4. **Round-trip validate.** For each format that you wrote, parse the file again with the same
   library and compare it with the source plan:
   - the stitch count agrees;
   - the color-stop/color-block count agrees;
   - the design extents agree within a small tolerance.

   A mismatch is a real bug. Repair the serialization and validate again. Never ship a file that
   fails its own round-trip check.
5. **Contract check (if applicable).** If `docs/features/<slug>/contracts/embroidery-export.md`
   exists, compare the fields and limits of the produced file with its table. Report each drift the
   same way as the drift check of `api`: show it, do not silently resolve it.
6. **Write the report.** Use [`./templates/export-report.md`](./templates/export-report.md) to write
   `docs/embroidery/<design>/_export/export-report-<format>-<date>.md`. Include these items:
   - the target format;
   - the pre-export validation results (divided stitches, hoop-fit check, needle-count flag);
   - if a real serialization library was used, or if the fallback spec-only path was used;
   - the round-trip result;
   - each `<!-- TBD -->` in a domain document that this export used.
7. **Self-check.** The round-trip validation of step 4 **is** the structural self-check of this
   skill. A file that did not pass the round-trip check is not an export. It is a draft.
8. **Handoff.** Propose the commit `embroidery-export: <design> (<format list>)`. Then **emit the
   stage-handoff block** per [`../_shared/handoff.md`](../_shared/handoff.md) — *Що я зробив* (formats
   produced, round-trip result) + *Перевір перед тим як продовжити* (the exported file(s), `export-report.md`) + *Що далі*:
   `/sdd-emb:embroidery-qa <design>` before the design is used as production-ready.

## Definition of Done

- For each requested format, one of these is true:
  - a real exported file exists **and passed the round-trip check**;
  - the report states clearly that no binary was made, and gives the reason (no library available).

  Never a silently invented file.
- No stitch in the output is longer than the hard length limit of the target format. Moves that
  were too long were divided into jumps. They were not truncated and not silently accepted.
- The report names each `<!-- TBD -->` in a domain document that this export used. It does not
  silently trust it.
- `export-report.md` exists for each format and date, and records the validation and round-trip
  results.
- The round-trip check of step 4 is the **structural self-check** of this skill
  ([`../_shared/self-check.md`](../_shared/self-check.md)). The handoff reports its result.

## Anti-patterns

- **Fabricating file bytes from reasoning about the format structure** in place of real
  serialization code. A stitch file is a precise artifact. "This should be roughly right" is not an
  export. It is a guess with a `.dst` extension.
- **Skipping the round-trip check** because the serialization "should be fine". This step is there
  to find exactly this failure. It copies the discipline of `api`: "a clean 4/4 ✓ is cheap; a
  silent ✗ in prod is not".
- **Using a machine-constraints number with a `<!-- TBD -->` marker as a verified hard limit**
  without a flag. An unverified 12.1 mm can be wrong for the actual target machine.
- **Truncating or silently dropping a stitch that is too long.** Divide it into the correct
  multi-jump sequence for the format.
- **Claiming success on the fallback spec-only path.** If no library was available, the handoff
  must say so clearly. Never imply that a binary file exists when only a specification exists.

## References & template

- `docs/domain/embroidery/file-formats.md` — the encoding and hard limits of each format, and the
  open-source libraries (pyembroidery, libembroidery, Ink/Stitch) that can read and write each format.
- `docs/domain/embroidery/machine-constraints.md` — the generic stitch/jump-length and
  trim-threshold guidance for when no specific machine profile is given.
- [`../api/templates/embroidery-export.md`](../api/templates/embroidery-export.md) — the **contract**
  (a spec-time document). If the calling feature declared one, this skill compares its actual
  output with it.
- [`../_shared/ask-style.md`](../_shared/ask-style.md) · [`../_shared/handoff.md`](../_shared/handoff.md) · [`../_shared/artifact-language.md`](../_shared/artifact-language.md) · [`../_shared/ste100.md`](../_shared/ste100.md).
- [`./templates/export-report.md`](./templates/export-report.md) — the output scaffold.
