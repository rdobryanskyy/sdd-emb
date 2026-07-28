# Embroidery domain overlay — conditional routing for SDD roles

> **Reference-only.** This is not a pipeline stage and does not create an artifact. It gives the generic SDD agents a shared, evidence-first embroidery profile when the feature actually controls or produces machine-embroidery work. The normal SDD flow and every documentation template remain unchanged.

## When to activate it

Activate the overlay when the task, feature artifacts, changed files, or code concern one or more of: a stitch plan; digitizing; stitch geometry; hoop or needle profiles; colour changes, jumps or trims; machine-file import/export/validation (`DST`, `PES`, `PEC`, `EXP`, `JEF`, `VP3`, `HUS`, `XXX`, `ART`, `EMB`); production scheduling for embroidery machines; or an `embedded-firmware` surface that drives such a machine. Do **not** activate merely because this plugin is installed.

The dispatcher states `embroidery domain overlay: active` in the agent prompt and names the relevant machine profile / format / design path. If the scope is unclear, the agent asks the dispatcher to route it rather than assuming that ordinary application code is embroidery code.

## Canonical evidence

Read only the documents relevant to the stated scope, before judging or researching:

| Scope | Source of truth |
|---|---|
| Stitch type, underlay, density, pull compensation, appliqué, lettering | `docs/domain/embroidery/stitch-vocabulary.md` |
| Per-stitch/jump limits, trims, hoop fit, needle count, speed | `docs/domain/embroidery/machine-constraints.md` |
| Binary/structured file semantics and format limits | `docs/domain/embroidery/file-formats.md` |
| Multi-head production, thread changes, stabilizers, time estimation | `docs/domain/embroidery/production.md` |
| Project vocabulary | `CONTEXT.md` |

An `<!-- TBD: verify -->` value is **not** a hard production limit. Agents may report that code treated it as authoritative, but must not invent a replacement threshold. A real, supplied machine profile or accepted feature contract takes precedence over generic guidance.

## Role-specific duties

- **explorer / survey:** locate the actual machine boundaries, file readers/writers, geometry and unit conversions, profile/config sources, safety interlocks, simulators, fixtures, and the nearest tested precedent. Return citations; do not design a new machine protocol.
- **researcher:** read the relevant local domain document first, then research only the unanswered question. Separate vendor capability claims from verified format or machine facts; include the model/format/version and source date in every claim.
- **critic:** check that a spec/SAD has not turned an unverified generic number into a hard requirement, and that it names the machine profile, file format, units, and failure/abort path whenever they materially affect the feature. This is a coherence check, not a substitute for digitizing or QA.
- **reviewer:** verify the implementation preserves coordinates/units, validates limits before machine/file output, handles malformed or unsupported inputs safely, does not fabricate binary formats, and has tests for the relevant boundary and round-trip/error path. Cite source code and the governing contract/domain rule for every finding.
- **test-author / implementer:** turn the selected domain constraints into executable tests and explicit configuration. Never hard-code a generic or `TBD` value as a machine-specific fact; keep profile-dependent values configurable and validate before side effects.

## Escalation boundary

This overlay improves the engineering process; it never claims a physical design is production-ready. When a change creates or mutates a stitch plan or exported machine file, `review` must also require the relevant `embroidery-qa` / `embroidery-export` evidence (or mark the feature not ready for production). No agent may approve a physical run solely from code review.
