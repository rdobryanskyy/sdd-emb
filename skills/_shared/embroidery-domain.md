# Embroidery domain overlay — conditional routing for SDD roles

> **Reference-only.** This file is not a pipeline stage and does not make an artifact. It gives the generic SDD agents one shared, evidence-first embroidery profile. Use it when the feature controls or makes machine-embroidery work. The normal SDD flow and all documentation templates do not change.

## When to activate it

Activate the overlay when the task, the feature artifacts, the changed files or the code are about one or more of these items:

- a stitch plan, digitizing or stitch geometry;
- hoop or needle profiles;
- colour changes, jumps or trims;
- machine-file import, export or validation (`DST`, `PES`, `PEC`, `EXP`, `JEF`, `VP3`, `HUS`, `XXX`, `ART`, `EMB`);
- production scheduling for embroidery machines;
- an `embedded-firmware` surface that controls such a machine.

Do **not** activate it only because this plugin is installed.

The dispatcher writes `embroidery domain overlay: active` in the agent prompt. It also names the applicable machine profile, format or design path. If the scope is not clear, the agent asks the dispatcher to route it. The agent must not think that ordinary application code is embroidery code.

## Canonical evidence

Before you judge or research, read only the documents that apply to the stated scope:

| Scope | Source of truth |
|---|---|
| Stitch type, underlay, density, pull compensation, appliqué, lettering | `docs/domain/embroidery/stitch-vocabulary.md` |
| Per-stitch/jump limits, trims, hoop fit, needle count, speed | `docs/domain/embroidery/machine-constraints.md` |
| Binary/structured file semantics and format limits | `docs/domain/embroidery/file-formats.md` |
| Multi-head production, thread changes, stabilizers, time estimation | `docs/domain/embroidery/production.md` |
| Project vocabulary | `CONTEXT.md` |

An `<!-- TBD: verify -->` value is **not** a hard production limit. An agent can report that code used it as authoritative. The agent must not invent a replacement threshold. A real machine profile that the user supplied, or an accepted feature contract, has priority over generic guidance.

## Role-specific duties

- **explorer / survey:** find the actual machine boundaries, file readers and writers, geometry and unit conversions, profile and configuration sources, safety interlocks, simulators, fixtures, and the nearest tested precedent. Give citations. Do not design a new machine protocol.
- **researcher:** first read the applicable local domain document. Then research only the question that has no answer. Keep vendor capability claims separate from verified format or machine facts. In each claim, give the model, format or version and the source date.
- **critic:** make sure that a spec or SAD does not change an unverified generic number into a hard requirement. Make sure that it names the machine profile, file format, units and failure/abort path when they have an effect on the feature. This is a coherence check. It does not replace digitizing or QA.
- **reviewer:** make sure that the implementation keeps coordinates and units, validates limits before machine or file output, and handles malformed or unsupported input safely. Make sure that it does not invent binary formats, and that it has tests for the applicable boundary and the round-trip/error path. For each finding, cite the source code and the governing contract or domain rule.
- **test-author / implementer:** change the selected domain constraints into executable tests and explicit configuration. Never hard-code a generic or `TBD` value as a machine-specific fact. Keep profile-dependent values configurable, and validate them before side effects.

## Escalation boundary

This overlay makes the engineering process better. It never claims that a physical design is ready for production. When a change makes or changes a stitch plan or an exported machine file, `review` must also ask for the applicable `embroidery-qa` / `embroidery-export` evidence. If that evidence is not available, `review` marks the feature as not ready for production. No agent can approve a physical run only from code review.
