# SDD → Embroidery Engineering Platform — Assessment & Transformation Plan

> Status: **proposal, not yet executed.** Nothing in this repo's `agents/`, `skills/`, or config has
> been changed to produce this document. It is the analysis + plan the user asked for before any
> code is touched.

## 0. What this repo actually is (read before anything else)

`sdd` is **not** a domain application. It is a **stack-agnostic, domain-agnostic software-engineering
methodology**: 9 subagents + 19 skills that carry *any* software feature from idea → spec → design →
data model → API → tasks → TDD implementation → review → ship, plus a portfolio roadmap layer. Every
skill's own text asserts this repeatedly — "stack-agnostic", "imposes no philosophy", "detects and
follows", "no product or library names", "product-level only". The agents are **process roles**
(critic, explorer, reviewer, test-author…), not domain simulators.

This matters because it inverts the naive reading of the goal. **"Turn this into an embroidery
platform" does not mean rewriting the 9 agents / 19 skills to know about stitches.** It means:

1. Keep the generic engine exactly as it is — it will build *any* embroidery software (a digitizing
   app, a production-planning service, a machine-file exporter) exactly as well as it builds anything
   else, the moment the *content* flowing through it (the spec, the glossary, the architecture) is
   about embroidery.
2. Add the **domain knowledge** the engine is missing — file formats, machine constraints, stitch
   vocabulary — as reference content the existing "optional context" hooks already know how to
   consume (`CONTEXT.md`, "read a reference module", `docs/domain/`).
3. Add a **small, clearly-separated new capability** only where the engine is genuinely missing a
   *kind of work* it has no role for today: computing stitch geometry and emitting machine binary
   formats. That is not process orchestration — nothing in the current 9+19 does geometry or binary
   I/O — so this is the one place net-new skills are justified.

Everything below is organized around that split.

---

## 1. Repository Assessment

### 1.1 Architecture (as built)

```
plugin.json / marketplace.json         — Claude Code / Codex / Cursor plugin manifests
agents/*.md            (9)             — process-role subagents (judgment / execution / scan)
skills/<name>/SKILL.md (19)            — one atomic pipeline stage or utility each
skills/<name>/references/*.md          — per-skill protocol detail (kept out of the spine file)
skills/<name>/templates/*.md           — output scaffolds with inline generation contracts
skills/_shared/*.md                    — cross-cutting contracts (12 files): Socratic loop, critic
                                          dispatch, ask-style, size/route matrix, surfaces taxonomy,
                                          agent roster + model policy, artifact-language, handoff
                                          block, diagram presentation, mermaid validation, self-check,
                                          tool-adapters (Codex/Cursor mapping)
server/ + dashboard/                   — optional local read-only visual dashboard (Bun MCP server)
evals/                                 — scenario-based skill regression tests
install.sh                             — installs the same files unchanged into Claude/Codex/Cursor
```

The backbone is a strict pipeline where **each stage reads the previous stage's artifact and
hard-refuses if it's missing**: `survey → specify → clarify → design → sequences → data-model → api
→ tasks → plan-tests → implement → review → ship`, with `glossary`, `roadmap`, `fix`, `decide-adr`,
`classify-size`, `interview`, `start` as always-available utilities. A `.size`/`.route` pair picked
once at intake governs how much of every downstream artifact gets produced and which optional stages
auto-skip.

### 1.2 Strengths (why this survives the domain switch almost untouched)

- **No domain or stack is hard-coded anywhere.** Every skill explicitly forbids naming a concrete
  technology in the artifacts it's not responsible for (spec has no HTTP, SAD has no vendor lock,
  sequences have no product names). This is exactly the property that lets a *new* domain (embroidery)
  slot in without touching the engine — the same property that already lets it build a fintech app or
  a game equally well.
- **A named, fixed-but-extensible extension point already exists for "what kind of thing is this
  feature": the target-surface taxonomy** (`skills/_shared/surfaces.md`). It's explicitly designed to
  gain a new row ("a genuinely new surface — e.g. a voice/IVR or an embedded firmware target — extends
  the table here, in one place"). An embroidery-machine controller target is *exactly* the case this
  sentence anticipates.
- **A second, already-extensible interface-kind list** in `api` (HTTP/REST, gRPC, CLI, Library/SDK,
  Event-only) is the natural home for an "embroidery file export" contract kind.
- **A working, unused-so-far domain-knowledge channel**: `CONTEXT.md` (two-level glossary), an
  optional "reference module" input every Q&A skill already asks about, and `architecture-map.md`'s
  precedent/convention sections. None of this requires code changes to carry embroidery vocabulary.
- **Judgment agents are genuinely domain-blind by design** (`analyst`, `strategist`, `researcher`,
  `devils-advocate`, `critic`, `reviewer` all explicitly forbid naming concrete products/tech and stay
  "product-level" / "abstract") — they will reason about an embroidery feature's trade-offs exactly as
  well as any other, with zero prompt changes.
- **Stack-agnostic data/API skills already detect-and-follow** rather than impose — so embroidery
  entities (Design, StitchBlock, ColorStop, Hoop, Machine) flow through `data-model`/`api` the same way
  any other domain's entities do.

### 1.3 Weaknesses / gaps relative to the stated goal

- **Nothing in the engine computes anything domain-specific.** No skill or agent does geometry,
  optimization, or binary serialization — because that was never in scope. If the goal is "the AI can
  generate/optimize/export an actual embroidery design," that capability does not exist anywhere in
  the current architecture and cannot be obtained by editing existing files — it's additive.
- **No embroidery knowledge exists yet** — no glossary terms, no format facts, no machine-constraint
  numbers. This is pure content debt, not architecture debt.
- **The surfaces/interface-kind lists are currently silent on embedded/firmware and on non-HTTP binary
  export** — small, explicitly-anticipated gaps, not flaws.
- **The plugin's public identity (name, description, keywords, README) is 100% generic SDLC branding.**
  **Decided: rename the plugin identity to `sdd-emb`** (see §9/§13 Phase 0) — this is bigger than a
  packaging tweak: a grep across the repo found **177 occurrences of the `sdd:` dispatch/slash-command
  namespace across 40 files**, **155 `/sdd:<skill>` slash-command references**, **15 files using the
  `sdd-` prefix convention** (Codex/Cursor agent installs), **5 `plugin.json`-family config files**
  declaring `"name": "sdd"`, and **37 occurrences inside `install.sh`**. It's fully mechanical and
  scriptable, but it touches nearly every file in the repo — sized and sequenced accordingly below.

### 1.4 Reusable components (the honest inventory)

| Component | Reuse verdict |
|---|---|
| All 9 agents | **100% reusable, unchanged.** |
| 17 of 19 skills (`specify`, `clarify`, `sequences`, `tasks`, `implement`, `review`, `ship`, `survey`, `fix`, `roadmap`, `glossary`, `interview`, `start`, `classify-size`, `decide-adr`, `data-model`, `plan-tests`) | **Reusable unchanged**, or with a one-line optional-input addition. |
| 2 of 19 skills (`design`, `api`) | Reusable, need a small, additive extension (surface + contract-kind). |
| `skills/_shared/*` (12 files) | Reusable; `surfaces.md` gets one new taxonomy row. |
| `server/` + `dashboard/` | Fully reusable — renders whatever `docs/features/` contains, domain-blind by construction. |
| `evals/` harness | Reusable mechanism; needs new embroidery-flavored scenarios, not new machinery. |
| Branding (`plugin.json`, `marketplace.json`, README framing) | Needs updating **only if** the product identity itself is meant to change, independent of the engine. |

---

## 2. Per-Agent Inspection

All nine agents are **process roles** — judgment, execution, or scan — defined entirely in terms of
SDLC artifacts (specs, diffs, ADRs, tests), never in terms of a business domain. None reference a
domain anywhere in their system prompt.

| Agent | Current Role | Proposed Role | Required Changes | Mod. Size |
|---|---|---|---|---|
| `analyst` | 3-lens (Engineer/Executive/UX) synthesis of candidate approaches | Unchanged | None | **None** |
| `critic` | Cross-section coherence critic (F1–F6) for spec/SAD | Unchanged | None | **None** |
| `devils-advocate` | Ambiguity hunt (spec) / failure-mode hunt (idea) | Unchanged | None | **None** |
| `explorer` | Fast read-only brownfield scout / bug localizer | Unchanged | None (optionally told, per-dispatch, to recognize `.dst/.pes/.exp/…` as domain assets when scanning an embroidery-software repo — a prompt detail owned by the *dispatching skill*, not the agent file) | **None** |
| `implementer` | GREEN/REFACTOR/GATE for one task | Unchanged | None | **None** |
| `researcher` | Competitive/adjacent-solution web research | Unchanged | None — will correctly research Wilcom/Hatch/Embird/SewArt etc. once dispatched on an embroidery idea, because it already stays product-level | **None** |
| `reviewer` | Read-only AC + quality review of a diff | Unchanged | None — embroidery-specific quality bars (stitch bounds enforced, format round-trips) enter as ordinary spec ACs/NFRs, which `reviewer` already checks | **None** |
| `strategist` | 3 strategic approaches (Simplicity/Differentiation/Balanced) | Unchanged | None | **None** |
| `test-author` | RED test writer | Unchanged | None | **None** |

**Verdict: zero of the nine agents require modification.** None become obsolete. None need new
tools, new system prompts, or new domain knowledge baked in — by design, they receive whatever domain
context the dispatching skill inlines or points them at (`CONTEXT.md`, spec, upstream artifacts).

---

## 3. Per-Skill Inspection

| Skill | Current Function | Required Changes | Reason | Impact on Other Skills | Compatibility Risk |
|---|---|---|---|---|---|
| `survey` | Map an existing repo / bootstrap a greenfield one | None | Architecture-mapping logic is domain-blind | None | None |
| `interview` | Pressure-test a raw idea pre-spec | None | Explicitly scoped to "any idea" | None | None |
| `classify-size` | XS–XL sizing + route | None | Size signals (PR count, migrations, breaking changes) are domain-blind | None | None |
| `specify` | Idea → reviewed `spec.md` | None (embroidery vocabulary enters via §5 in the domain-population step, see §4 below) | Spec drafting is content-driven by the interview + glossary, not by skill code | None | None |
| `clarify` | Ambiguity sweep on `spec.md` | None | Ambiguity classes are structural, not domain | None | None |
| `glossary` | Capture/update `CONTEXT.md` terms | None (this **is** the domain-knowledge injection point — usage, not code) | Already generic; embroidery terms are just terms | Feeds `specify`/`design`/`api` automatically once populated | None |
| `design` | Arc42 SAD + C4 + ADR-on-blast-radius | **Small**: add one row to the *shared* surface taxonomy (`skills/_shared/surfaces.md`) for an embedded/machine-controller target; the skill file itself is unchanged (it already just reads the taxonomy) | Firmware/embedded-controller software is a genuinely different C4 container the current 7-surface list doesn't name | `api`, `sequences`, `tasks`, `plan-tests` all read the taxonomy and gain the new row automatically | Low — purely additive, taxonomy is explicitly designed for this |
| `sequences` | Runtime-flow Mermaid diagrams | None | Generic participant vocabulary already covers any domain | None | None |
| `data-model` | Schema + staged migrations | **Very small**: document one new optional input — a domain-constraints reference doc (e.g. `docs/domain/machine-constraints.md`) the Socratic column/constraint dialogue can cite when the user confirms bounds (e.g. `stitch_length_mm CHECK <= 12.1`) | Today the skill only detects constraints from the *repo's own* schema conventions; a physical-world numeric bound (needle throw limit) isn't a schema convention, it's domain physics — needs a place to come from | None | None — additive input, no behavior change if the doc is absent |
| `api` | Derives one interface contract per declared surface | **Small**: add one more interface-kind row — "Embroidery file export" → `contracts/embroidery-export.md` (format, header/record layout reference, per-field bounds) — alongside the existing HTTP/gRPC/CLI/Library/Event-only list | The skill's own list of contract *kinds* is closed; a stitch-file exporter is a real interface kind the current list doesn't name | None | Low — same derive→drift-check→reconcile loop, one more kind |
| `tasks` | Breaks a designed feature into ≤1-day tasks | None | The 9 generic layers (migration/domain/infra/app/ports/ui/tests/wiring/docs) already cover stitch-generation logic as `domain`, file emission as `ports`/`infra`, etc. | None | None |
| `plan-tests` | AC → test-level mapping | None required now; if the new `api` export kind is adopted, its Definition-of-Done naturally gains a "format round-trip / byte-compliance" test row under the existing generic vocabulary — no schema change needed | The existing level vocabulary (unit/integration/e2e/contract) already fits "assert the emitted DST parses back to the same stitch count" | None | None |
| `implement` | TDD engine (RED→GREEN→REFACTOR→GATE→COMMIT) | None | Command-detection and gating are stack-agnostic by construction | None | None |
| `review` | Independent AC + quality review of the whole diff | None | Domain-specific correctness bars are just ACs; the review protocol doesn't change | None | None |
| `ship` | Verify-it-works + changelog + PR | None | "Run the feature for real" already accommodates "generate a stitch file and open it in a viewer" as the verification step, with zero wording change needed | None | None |
| `fix` | Spec-first bugfix entry point | None | Triage logic (regression / spec-bug / gap) is domain-blind | None | None |
| `roadmap` | Now/Next/Later portfolio view | None | Outcome-altitude items are domain-blind | None | None |
| `decide-adr` | Post-hoc/async ADR recording | None | MADR format + blast-radius gate are domain-blind | None | None |
| `start` | Opens the local dashboard | None | Renders whatever `docs/features/` holds | None | None |

**Verdict: 17 of 19 skills need zero changes. Two (`design`, `api`) need a small, additive extension
that lives almost entirely in the already-designed extension points (`_shared/surfaces.md`, `api`'s
interface-kind list) — not in the skills' own protocol logic.** No skill becomes obsolete. No skill
needs replacing.

---

## 4. Domain Conversion — where embroidery knowledge actually lives

The list of embroidery sub-domains in the brief (digitizing, underlay, pull compensation, density,
satin/fill/running stitches, motifs, appliqué, sequins, multi-head/multi-color, lettering, monograms,
production, QC) is **content**, not architecture. It becomes real through four concrete artifacts,
all using mechanisms the pipeline already has:

1. **`CONTEXT.md` (repo-root)** — seeded once, via the existing `glossary` skill, with the canonical
   one-line definitions for every domain term above (e.g. `- underlay — a foundation stitch layer laid
   before the top stitching to stabilize fabric and improve coverage. NOT the top stitch itself.`).
   `specify`, `clarify`, `design`, `api` already treat this as canonical — no code change.
2. **`docs/domain/` (new, plain reference docs, not skills)** — the deeper structured facts a glossary
   line can't hold:
   - `docs/domain/embroidery/file-formats.md` — DST, PES, EXP, JEF, VP3, HUS, XXX, ART, EMB: encoding
     model (relative vs. absolute coordinates), stitch record layout, color-change/trim/jump command
     semantics, per-format capability matrix.
   - `docs/domain/embroidery/machine-constraints.md` — max/min stitch length, max jump length before a
     forced trim, needle count/sequencing conventions, hoop size table, typical machine speed ranges,
     trim/color-stop command semantics.
   - `docs/domain/embroidery/stitch-vocabulary.md` — satin, fill, running, motif, appliqué, sequin
     definitions; underlay types; density defaults; pull-compensation heuristics.
   - `docs/domain/embroidery/production.md` — multi-head/multi-color workflow, thread manufacturer
     palette conventions, production-time estimation factors.
   These are read exactly like `architecture-map.md`'s precedent sections are read today — cited,
   never re-derived, never imposed as a "philosophy" (they're physical/domain facts, not
   implementation-stack opinions, so they don't conflict with the "detect, don't impose" principle
   `data-model`/`survey` already follow for *stack* choices).
3. **`spec.md` §5 acceptance criteria** — every domain capability in the brief ("reduce jump stitches",
   "select densities", "detect embroidery problems") becomes an ordinary, business-observable AC on
   whichever feature owns it (e.g. the digitizing engine's spec), exactly like any other product
   requirement. This is `specify`'s existing job, not a new mechanism.
4. **`evals/scenarios/`** — new embroidery-flavored eval scenarios (e.g. `specify` on a "reduce jump
   stitches" idea; `data-model` on a stitch-plan schema) exercise the *existing* skills against
   embroidery content, catching any place a skill's wording accidentally assumes a non-embroidery
   domain (none currently found in the audit above).

---

## 5. Required AI Capabilities — mapped, not invented

| Capability from the brief | Where it lives today / after this plan |
|---|---|
| Generating embroidery designs, planning stitch paths, selecting stitch types/densities | **New**: `embroidery-digitize` skill (§7) — no existing skill computes geometry |
| Optimizing embroidery order, reducing jump stitches, optimizing trims, reducing thread/color changes | **New**: `embroidery-optimize` skill (§7) |
| Compensating for fabric distortion, suggesting stabilizers, selecting needles/threads | Digitizing/optimization skills' internal logic, informed by `docs/domain/embroidery/*` |
| Calculating production time, calculating stitch counts | Deterministic derivation inside `embroidery-optimize` / `embroidery-export`; surfaced as ordinary spec KPIs/NFRs for whatever product feature reports them |
| Detecting embroidery problems, suggesting improvements | **New**: `embroidery-qa` skill (§7) |
| Preparing files for production, generating machine-ready projects | **New**: `embroidery-export` skill (§7) + `api`'s new "Embroidery file export" contract kind (§3) |
| Everything about *building the software that does the above* (planning, architecture, data model, API, tasks, tests, implementation, review, shipping) | **Already fully covered** by the unmodified 19-skill/9-agent backbone — this is exactly what SDD already does for any product |

This table is the crux of the plan: the brief conflates two different products — *"software that can
digitize/optimize/export embroidery"* (a new, small, additive skill family) and *"a methodology for
building that software correctly"* (already 100% present and untouched).

---

## 6. New Skills — justified, scoped, and why existing skills can't do this

None of the 19 existing skills compute geometry, run stitch-optimization algorithms, or serialize a
binary machine format — that kind of work has no analog in the current skill set (the closest is
`data-model`, which designs *schemas*, and `api`, which designs *contracts* — neither *computes*
anything). Four new skills are proposed, organized as a **separate, optional module**
(`skills/embroidery/`) that sits *beside* the SDLC backbone, not inside its gated chain — invoked by
name, the same way `glossary`/`roadmap`/`fix` are utilities today, not new stages that other stages
hard-refuse without.

| New skill | Why no existing skill fits | Integrates via | Dependencies | Inputs | Outputs |
|---|---|---|---|---|---|
| `embroidery-digitize` | Turning artwork/outline + user intent into a stitch plan (stitch types, underlay, density) is a geometry-generation task; no skill in the pipeline generates geometry | Standalone utility skill, callable directly or from a feature's `implement` task as a library call | `docs/domain/embroidery/stitch-vocabulary.md`, `CONTEXT.md` | Artwork/outline reference, target fabric/stabilizer, hoop size | A stitch-plan document (entities: Design, StitchBlock, ColorStop) — the natural next input to `data-model` if this becomes a persisted feature |
| `embroidery-optimize` | Reordering stitch/color sequence to minimize jump/trim/thread-change count is a combinatorial optimization, not a review or a design decision | Standalone utility; consumes a stitch plan (from `embroidery-digitize` or an imported file) | `docs/domain/embroidery/machine-constraints.md` | A stitch plan | An optimized stitch plan + a delta report (jumps/trims/changes before vs. after) |
| `embroidery-export` | Serializing a stitch plan into DST/PES/EXP/JEF/VP3/HUS/XXX/ART/EMB byte layouts, and validating it against machine limits, is binary I/O + constraint-checking — no skill does either | Standalone utility; the *contract shape* it targets is authored once by `api`'s new "Embroidery file export" kind (§3) when this is a persisted product feature | `docs/domain/embroidery/file-formats.md`, `docs/domain/embroidery/machine-constraints.md` | A stitch plan, target format | The exported file + a validation report |
| `embroidery-qa` | Judging a stitch plan/file against density/pull-compensation/hoop-fit/jump-budget/color-stop-count rules is domain-physics judgment, distinct from `review` (which judges *code* against a *spec*) | Standalone utility, analogous in spirit to `reviewer` but over embroidery content, not a diff | `docs/domain/embroidery/machine-constraints.md` | A stitch plan or exported file | A findings report (cited, same discipline as `reviewer`'s output) |

All four are scoped as **skills**, not pipeline stages: they don't gate on or hard-refuse for missing
upstream SDD artifacts, and the 19-skill backbone doesn't gate on them either. A feature that *builds*
one of these capabilities as a product (e.g., "our app has an auto-digitize button") still goes
through the unmodified `specify → design → … → ship` chain like any other feature; these new skills
are what that feature's implementation *calls*, and/or a standalone tool a user invokes directly for a
one-off design task.

---

## 7. New Agents — none justified yet

Applying the same bar the existing roster uses (`agents/*.md` + `_shared/agent-roster.md`: an agent
exists when a task needs **isolated fresh context** for judgment, or a **narrow, fast, cheap scan**,
or a **role split** that must not see its counterpart's work — e.g. `test-author` must not see
`implementer`'s code before writing RED):

- The four new skills in §7 are **deterministic/algorithmic** (geometry, optimization, serialization,
  constraint-checking) — they don't need a fresh, blind perspective the way `critic`/`reviewer`/
  `devils-advocate` do. They're closer in shape to `data-model`/`api`, which run their derivation
  **inline in the skill**, with no dedicated agent.
- No role-isolation requirement exists yet analogous to RED-must-not-see-GREEN.

**Recommendation: implement all four as agent-free skills first** (matching `data-model`/`api`'s
existing pattern). Revisit only if a concrete need appears later — e.g., trying several digitizing
strategies **in parallel** for the user to compare (which would look like `strategist`'s pattern:
`sdd:embroidery-digitize-candidate`, dispatched N times with different strategies, then synthesized).
That is a plausible Phase-4 enhancement, not a Phase-1 requirement — do not build it speculatively.

The brief's suggested agent names (Embroidery Digitizer, Stitch Optimizer, Fabric Analysis Agent,
Embroidery QA Agent, Thread Optimization Agent, Production Planning Agent, Export Agent, Machine
Compatibility Agent) map onto the four skills above at roughly 2:1 — several of the suggested names
are facets of the same skill (fabric analysis + thread optimization + machine compatibility are all
inputs `embroidery-digitize`/`embroidery-optimize` read from `docs/domain/embroidery/*`, not separate
computation engines). Collapsing them avoids the "an ADR/agent for every decision" anti-pattern the
existing skills explicitly warn against (`design`'s own anti-patterns list: "kills the genre").

---

## 8. Embroidery Machine Knowledge — consumers, not new mechanisms

| Knowledge area | Lives in | Read by |
|---|---|---|
| File formats (DST/PES/EXP/JEF/VP3/HUS/XXX/ART/EMB) | `docs/domain/embroidery/file-formats.md` | `embroidery-export`, `api`'s export contract kind |
| Machine limits (stitch length, jump length, hoop sizes, speed, needle sequencing, trim/color-stop commands) | `docs/domain/embroidery/machine-constraints.md` | `embroidery-optimize`, `embroidery-qa`, `data-model` (column bounds), `plan-tests` (format-compliance test rows) |
| Color palettes / thread manufacturers | `docs/domain/embroidery/production.md` | `embroidery-digitize`, `embroidery-optimize` |
| Digitizing vocabulary (satin/fill/running/motif/appliqué/sequin, underlay, density, pull compensation) | `docs/domain/embroidery/stitch-vocabulary.md` + `CONTEXT.md` | `specify`, `design`, `embroidery-digitize` |

No new mechanism is required to make the pipeline *aware* of this content — `specify` step 5 already
asks "which extra channels to read" (reference module / project docs / KB / none) and treats a named
path as authoritative once pointed at it. The only actual work is **writing these four documents**
and, for `data-model`/`api`/`plan-tests`, adding one line to their existing "Inputs" sections naming
`docs/domain/` as a place to look (§3's "very small" changes).

---

## 9. Modification Plan (priority order)

**Decisions locked in from review:** rebrand to **`sdd-emb`**, done first (Phase 0) rather than
deferred — content authored in later phases (new skills, docs, evals) should be written under the
final name so nothing needs a second pass. New-capability scope confirmed at **4 skills / 0 agents**.
Domain-knowledge docs (row 3) are drafted with explicit `<!-- TBD: verify -->` markers on every
unverified numeric/technical fact, not asserted as ground truth — a required follow-up verification
pass (by the user or a domain expert) gates before `data-model`/`embroidery-export`/`embroidery-qa`
treat any of those numbers as authoritative.

| # | Change | Files touched | Size | Priority |
|---|---|---|---|---|
| 0 | **Rename plugin identity `sdd` → `sdd-emb`**: all 5 `plugin.json`-family configs (Claude/Codex/Cursor + both marketplace.json), every `sdd:<skill>` dispatch/slash-command reference (177 occurrences / 40 files), the `sdd-` install-prefix convention (15 files), `install.sh` (37 occurrences), `agent-roster.md`, `tool-adapters.md`, README | ~45 files, scripted find/replace + manual verification pass | **Medium** (mechanical but wide) | **P0 — before any new content is authored**, so it's written under the final name once |
| 1 | Write the four `docs/domain/embroidery/*.md` reference docs, **with `<!-- TBD: verify -->` on every unverified numeric/format fact** (stitch-length/jump limits, hoop sizes, byte layouts) | new files | Small (content, not code) | P0 — everything else reads these |
| 2 | Seed `CONTEXT.md` with the core embroidery glossary via `glossary` | `CONTEXT.md` (generated, not hand-edited) | Trivial (usage) | P0 |
| 3 | Add the embedded/machine-controller row to `skills/_shared/surfaces.md` | 1 shared file | Small | P1 |
| 4 | Add the "Embroidery file export" interface kind to `skills/api/SKILL.md` (+ a `contracts/embroidery-export.md` template) | `skills/api/SKILL.md`, new template | Small | P1 |
| 5 | Add the optional `docs/domain/` input line to `data-model`, `plan-tests` Inputs sections | 2 skill files | Very small | P1 |
| 6 | Build `skills/embroidery/embroidery-digitize` | new skill dir | Medium | P2 |
| 7 | Build `skills/embroidery/embroidery-optimize` | new skill dir | Medium | P2 |
| 8 | Build `skills/embroidery/embroidery-export` | new skill dir | Medium | P2 |
| 9 | Build `skills/embroidery/embroidery-qa` | new skill dir | Medium | P2 |
| 10 | New eval scenarios exercising `specify`/`data-model`/`api` on embroidery content | `evals/scenarios/*` | Small | P2 (parallel with 6–9) |
| 11 | **Verification pass on every `<!-- TBD: verify -->` marker** from row 1, once real format/manufacturer sources are available | `docs/domain/embroidery/*.md` | Small, but gates trusting the content | P3 — required before shipping anything that relies on the numbers (e.g. `embroidery-export`, `data-model` CHECK constraints) |
| 12 | (Optional, later) parallel-candidate digitizing agent, if/when needed | new agent | Medium | P4 — not now (§7) |

Rows 1–11 touch only 4 of 19 backbone `skills/*/SKILL.md` files (`api`, `data-model`, `plan-tests` get
one line each; `design` gets zero — it already just reads the shared taxonomy file row 3 edits), and
zero `agents/*.md` files. Row 0 is the one wide-reaching change, and it's pure renaming — no protocol
logic changes.

---

## 10. Migration Plan (how to evolve without breaking anything)

1. **Phase 1 is purely additive and non-breaking by construction.** `docs/domain/` is a new directory;
   nothing reads it until a skill's optional-input check finds it, so its absence today doesn't change
   any existing behavior. Existing non-embroidery uses of this plugin (if any) are completely
   unaffected — no skill loses generality.
2. **`surfaces.md` and `api`'s interface-kind list are extended, never edited-in-place.** Both are
   explicitly designed as open lists ("fixed but extensible" / a closed enum today). Adding a row
   cannot regress an existing surface/kind's behavior — every existing gating-table row is untouched.
3. **The four new skills ship as an independent module** (`skills/embroidery/`), versioned and
   installable the same way the rest of the plugin is (picked up automatically by `install.sh`'s
   verbatim-copy mechanism — no installer change needed, confirmed by reading `install.sh`'s copy
   logic operating on the whole `skills/` tree).
4. **Rollback is trivial at every step**: delete `docs/domain/`, revert the two shared-file additions,
   or remove `skills/embroidery/` — none of the 19 backbone skills reference these paths as hard
   requirements (`test -f` gates), so their absence degrades to "the optional input wasn't found,"
   never a refusal.
5. **Sequencing**: do P0 (knowledge docs + glossary) before anything else — it's the only work every
   later phase depends on, and it's zero-risk (content, no code path changes).
6. **No "big bang."** Because nothing is edited in place, this can ship as N small, independently
   reviewable PRs, one per row in §9's table, in the priority order given.

---

## 11. Dependency Graph (after this plan)

```
docs/domain/embroidery/*.md  ─┬──────────────────────────────────────────────────┐
                               │                                                    │
CONTEXT.md (glossary) ────────┼──> specify ──> clarify ──> design ──> sequences ──> data-model ──> api ──> tasks ──> plan-tests ──> implement ──> review ──> ship
                               │                    │                     │                          │
                               │                    │           (surfaces.md: +embedded row)          │  (+ "Embroidery file export" kind)
                               │                    │                                                  │
                               └────────────────────┴──────────────────────────────────────────────────┘
                                              (all backbone stages: UNCHANGED protocol logic)

skills/embroidery/  (new, standalone module — not gated into the chain above)
  embroidery-digitize ──> embroidery-optimize ──> embroidery-export ──> embroidery-qa
       ^                        ^                        ^                   ^
       └── docs/domain/embroidery/stitch-vocabulary.md ───┘                   │
       └── docs/domain/embroidery/machine-constraints.md ────────────────────┘
       └── docs/domain/embroidery/file-formats.md (export only)

  A product feature that surfaces these (e.g. "auto-digitize button") is built through the
  UNCHANGED backbone above; its `implement` tasks call into skills/embroidery/* as the actual
  computation, the same way a normal feature's tasks call into a payment SDK or a PDF library.

agents/*.md (9)  — dispatched by the backbone exactly as today; zero new edges.
```

The graph's key property: the new embroidery module is a **leaf dependency of feature implementations**,
never an input the backbone itself depends on. The backbone works with zero embroidery-specific
content present; the embroidery module works standalone without ever invoking the backbone.

---

## 12. Risk Analysis

| Risk | Description | Mitigation |
|---|---|---|
| **Scope conflation** (highest risk) | Treating "build embroidery software" (the backbone's job) and "compute embroidery geometry" (the new module's job) as the same thing leads to stuffing domain algorithms into `design`/`data-model`/`api`'s prompts, violating their stack-agnostic contract and creating exactly the kind of one-off special-casing the whole plugin is built to avoid. | Keep the module boundary in §6/§7 hard: no domain computation logic enters `agents/*.md` or the 19 backbone `SKILL.md` files. |
| **Domain knowledge accuracy** | File-format byte layouts and machine limits (stitch length, jump thresholds) are precise engineering facts; a wrong number in `docs/domain/embroidery/machine-constraints.md` silently propagates into `data-model` CHECK constraints and `embroidery-qa` verdicts. | **Decided:** Phase 1 drafts every unverified numeric/format fact behind an explicit `<!-- TBD: verify -->` marker rather than asserting it — the docs are structurally complete but not trusted until §9 row 11's verification pass (real format specs / manufacturer docs / an existing library like `libembroidery`/`pyembroidery`) clears each marker. `data-model`/`embroidery-export`/`embroidery-qa` should refuse to treat a still-marked TBD value as a hard constraint. |
| **Rename regressions** | A missed `sdd:` occurrence among the 177 found leaves a dangling dispatch reference or a slash command that silently 404s (e.g. a skill's handoff still printing `/sdd:design <slug>` instead of `/sdd-emb:design <slug>`). | Do the rename as one scripted, reviewable pass (§9 row 0): sed/grep-replace, then `grep -rn "sdd:"` and `grep -rn "/sdd:"` across the repo must return **zero** hits outside deliberate historical references (e.g. a CHANGELOG entry); re-run the `evals/` scenarios afterward as a regression check. |
| **Taxonomy sprawl** | Adding surfaces/interface-kinds is explicitly designed to be safe, but doing it for every conceivable embroidery variant (one row per machine brand, say) would recreate the "ADR for every decision" anti-pattern the plugin's own docs warn against. | Add exactly the two rows justified in §3 now; require a real blast-radius justification (irreversible / multi-module / real alternatives, the same 3-criteria gate `design` already uses) before adding more. |
| **New-skill scope creep** | The brief's 8 suggested agent names could balloon into 8 separate skills/agents if taken literally. | §6/§7 already collapse them to 4 skills, 0 new agents — hold this line during implementation; anything that looks like a 5th skill should first be checked against "is this actually a facet of digitize/optimize/export/qa." |
| **Rebranding ambiguity — resolved** | ~~The plugin's identity is separate from whether the product built with it is embroidery software~~ — **decided: rename to `sdd-emb`.** Residual risk: existing installs of `sdd` (if any are in active use) break on update since the plugin id changes; the marketplace listing needs republishing under the new name. | Do the rename in one clean pass (§9 row 0) with a clear CHANGELOG/README note for anyone with an existing `sdd` install; confirm whether the **GitHub repo name** / marketplace source URL also needs to change or only the in-plugin `name` fields (this repo is still `rdobryanskyy/sdd` at the git-remote level per `plugin.json`'s `repository` field — flag this as a separate, smaller decision before executing row 0, not assumed). |
| **Performance / no runtime concerns identified** | The backbone does no computation itself (it's a documentation/orchestration pipeline); the new module's algorithms (stitch optimization, format export) are the only place real performance work exists, and that's ordinary software-engineering scope covered by the backbone's own NFR/plan-tests machinery once specified. | No special mitigation needed beyond normal NFR capture in `specify`. |
| **Missing knowledge at plan time** | This assessment does not itself contain the actual numeric machine limits or byte-level format specs (that's proprietary/manufacturer-specific detail, out of scope for an architecture assessment). | P0 in §9 explicitly calls for sourcing and writing these docs as the first real content task — flagged here so it isn't silently assumed already done. |

---

## 13. Implementation Roadmap

### Phase 0 — Identity rename (`sdd` → `sdd-emb`), done first
- One scripted pass across all 5 `plugin.json`-family configs, the 177 `sdd:` dispatch/slash-command
  references (40 files), the 15 `sdd-`-prefix install references, `install.sh` (37 occurrences),
  `agent-roster.md`, `tool-adapters.md`, README.
- Verify with `grep -rn "sdd:"` / `grep -rn "/sdd:"` returning zero unintended hits; re-run `evals/`.
- Confirm separately whether the GitHub repo/marketplace source URL renames too, or only the in-plugin
  `name` fields (flagged, not assumed, in §12's risk table).
- Done before Phase 1 so every new file authored from here on uses the final name once.

### Phase 1 — Domain foundation (content only, zero code risk)
- Write `docs/domain/embroidery/{file-formats,machine-constraints,stitch-vocabulary,production}.md`
  with `<!-- TBD: verify -->` on every unverified numeric/format fact — structurally complete, not
  yet trusted (see §9 row 11).
- Run `glossary` to seed `CONTEXT.md` with the core term set (stitch, underlay, pull compensation,
  density, satin/fill/running/motif/appliqué/sequin, jump, trim, color stop, hoop, digitizing).
- No agent/skill files touched beyond Phase 0's rename. Fully reversible (delete the new files).

### Phase 2 — Backbone extension points (small, additive)
- Add the embedded/machine-controller row to `skills/_shared/surfaces.md`.
- Add the "Embroidery file export" interface kind + `contracts/embroidery-export.md` template to
  `skills/api/`.
- Add the one-line optional `docs/domain/` pointer to `data-model` and `plan-tests` Inputs sections.
- Verify via a `design`/`api` dry run on a throwaway embroidery feature idea that the new rows engage
  correctly and every other surface/kind is untouched.

### Phase 3 — Embroidery capability module
- Build `skills/embroidery/{embroidery-digitize,embroidery-optimize,embroidery-export,embroidery-qa}`
  as agent-free skills (own `SKILL.md`, `references/`, `templates/` per the existing skill shape).
- Add matching `evals/scenarios/` for each.
- Exercise the full backbone end-to-end on a real feature idea ("auto-digitize a logo") to confirm the
  backbone (unchanged) correctly specs/designs/tasks/implements a feature whose `implement` step calls
  into the new module.

### Phase 4 — Hardening + optional enhancements
- Revisit whether any new skill's algorithmic core needs agent-level isolation (e.g., parallel
  digitizing-candidate comparison) once real usage shows a concrete need — build only then (§7).
- Decide and execute the packaging/rebrand question (row 11) if the product identity itself is meant
  to diverge from "SDD."
- Expand `docs/domain/embroidery/` with anything Phase 3 usage reveals is missing (thread-manufacturer
  palettes, more machine-family constraint tables) — content growth, not architecture growth.

---

## 14. Summary

- **9/9 agents unchanged.**
- **17/19 skills unchanged**; 2 (`design` via a shared file, `api`) get small additive extensions.
- **All embroidery domain knowledge is content** (glossary + 4 reference docs, drafted with `<!-- TBD:
  verify -->` markers on anything unverified), consumed through mechanisms the pipeline already has.
- **4 new skills, 0 new agents** deliver the genuinely new capability (computing/optimizing/exporting/
  validating embroidery geometry) that nothing in the existing architecture could do — scoped as a
  standalone module the backbone can call into, never the other way around.
- **Plugin identity renamed to `sdd-emb`**, executed as its own mechanical Phase 0 (177 dispatch/
  slash-command references, 5 config files, `install.sh`) before any embroidery content is authored.
- The repository remains recognizably itself: the same 19+9 engine, the same gated pipeline, the same
  stack-agnostic discipline — now named, and lightly extended, for one domain.

## 15. Open items — resolved

1. **Repo/marketplace-URL scope — resolved.** You're moving this branch to a separate repo later, so
   `repository`/`homepage`/install URLs were renamed now too (`rdobryanskyy/sdd-emb`, placeholder until
   the real repo exists — update once it's created).
2. **Verification owner for Phase 1 — resolved.** I run the web research myself (WebSearch/WebFetch,
   citing sources); you correct/supplement with real data afterward. See §16.

## 16. Phase 0 — executed and verified

Done, in this session:

- Grepped the exact footprint before touching anything: **80 files** contained the `sdd` token; **79**
  were mechanically renamed (`EMBROIDERY-TRANSFORMATION-PLAN.md` itself excluded — its prose describes
  the rename, not the identifier).
- Ran a word-boundary rename (`\bsdd\b` → `sdd-emb`, case-sensitive so the `SDD` methodology acronym
  and `SDD-Task`/`SDD-AC`/`SDD-Fix` git trailers were deliberately left untouched — those name
  "Spec-Driven Development" the methodology, not the plugin package id) across all 5
  `plugin.json`-family configs, `install.sh`, `scripts/validate_plugin.py` (including its own
  self-check assertions), `.mcp.json` + `.claude/settings.local.json` (the `sdd-dashboard` MCP server
  → `sdd-emb-dashboard`, consistently across `.mcp.json`, `server/package.json`, `server/bun.lock`, the
  `x-sdd-token`/`x-sdd-mtime` headers in `dashboard/app.js`), every `skills/**/*.md` dispatch/
  slash-command reference, `agent-roster.md`, `tool-adapters.md`, README, CONTRIBUTING, `.gitignore`,
  the CI workflow, and every `evals/scenarios/*` fixture.
- Left the dashboard's visible `SDD` logo/title/prose untouched (uppercase, methodology branding —
  distinct from the lowercase plugin-id rename that was actually asked for; revisit only if a full
  visual rebrand is wanted later).
- **Verified, not just assumed clean:** a corrected leftover-check (`\bsdd\b(?!-emb)`) found **zero**
  stray lone `sdd` tokens; the repo's own `scripts/validate_plugin.py` — which independently asserts
  the plugin name, the invocation-form convention, and the dashboard config — ran clean at
  **359/359 checks, exit 0**; `git diff --stat` showed a symmetric **428 insertions / 428 deletions
  across 78 files**, consistent with a pure line-level rename and no structural corruration.
- **Not yet done** (flagging so it isn't silently assumed): `server/bun.lock`'s package name was
  text-substituted rather than regenerated via `bun install` — safe here because it's a private,
  unpublished root package name with no dependency-hash implications, but worth a `bun install`
  sanity run before shipping. The actual GitHub repo has not been renamed/created — `rdobryanskyy/
  sdd-emb` is a placeholder in the config until the real separate repo exists.

## 17. Phase 2 — executed and verified

- Added `embedded-firmware` to the surface taxonomy in `skills/_shared/surfaces.md` (both the
  taxonomy table and the gating table), explicitly reusing existing generic participant/layer/test
  vocabulary rather than inventing new ones — a hardware-in-the-loop test tier is deliberately
  deferred until a real firmware feature needs it (no speculative build).
- Added **"Embroidery file export"** as an additive `api` contract kind (`skills/api/SKILL.md`),
  triggered by AC content (the spec describes producing a machine file), not by `target_surfaces` —
  it rides alongside whichever primary contract the surface already produces. Added the matching
  `skills/api/templates/embroidery-export.md` scaffold (target format, per-field bounds sourced from
  `docs/domain/embroidery/machine-constraints.md`, round-trip validation) and linked it from the
  skill's References section.
- Added one optional-input line each to `skills/data-model/SKILL.md` and `skills/plan-tests/SKILL.md`
  pointing at `docs/domain/**/*.md` as a source for numeric constraints/thresholds — explicit that an
  unverified (`<!-- TBD: verify -->`) fact must still be confirmed with the user, never trusted as a
  hard constraint.
- Re-ran `scripts/validate_plugin.py` after these edits: still **359/359 checks, exit 0** (link
  resolution intact, no orphaned reference introduced by the new template).

## 18. Phase 1 — executed

A background research agent compiled the four `docs/domain/embroidery/*.md` reference docs via web
search, citing **62 distinct sources** across the four files. Quality summary:

- **Well-sourced, cross-checked:** DST/EXP/JEF byte-level encoding (independently corroborated
  across pyembroidery, EduTechWiki, and the KDE Janome wiki), the DST 12.1 mm / EXP-Barudan 12.7 mm
  max-stitch-length pair (cross-checked against Wilcom's own documentation), PES/PEC's versioned
  two-layer container structure, the appliqué placement→tack-down→cover sequence, stabilizer
  type/use mapping, and the multi-head "every head runs the same design in parallel" production model.
- **Explicitly flagged `<!-- TBD: verify -->`, not guessed:** byte-level structure for XXX/ART/EMB/VP3
  (only vendor marketing copy found, no independently reverse-engineered spec), HUS's compression
  internals (secondhand ARJ/Greenleaf attribution, no primary source), a single universal minimum
  stitch-length constant (sources give risk bands, not one figure — the file documents the band
  instead of inventing a number), the 9×9/10×10 hoop family and 12–20-needle single-head machines
  (plausible from aggregated blogs, no specific spec sheet checked).
- Every production-time coefficient (efficiency %, per-swap minutes, hooping minutes) is flagged as
  industry-blog-sourced with an explicit recommendation to expose them as **configurable per-machine-
  profile parameters**, never hard constants — consistent with §12's domain-knowledge-accuracy risk
  mitigation.

Seeded repo-root `CONTEXT.md` with 18 core terms drawn from `stitch-vocabulary.md` (stitch, running/
satin/fill/motif stitch, underlay, density, pull compensation, jump/trim/color-stop/color-block,
appliqué, sequin, stabilizer, hoop, multi-head machine, digitizing) — batch-authored as content prep
rather than through the glossary skill's interactive per-term dialogue, per the project owner's
direction; each entry is a first draft to correct once real features are specified against it.

Re-ran `scripts/validate_plugin.py`: still **359/359 checks, exit 0** — the new `docs/domain/` and
`CONTEXT.md` files are additive and don't affect any structural check.

**Still open:** the numeric facts marked `<!-- TBD -->` above need real-source verification (§9 row
11) before `data-model`/`embroidery-export`/`embroidery-qa` should treat them as hard constraints —
this is the follow-up the project owner will supply.

## 19. Phase 3 — executed and verified

Built the four skills exactly as scoped in §6/§7 (4 skills, 0 new agents), each as a standalone
utility — never gated into the `specify → … → ship` backbone, matching `glossary`/`roadmap`/`fix`'s
existing pattern:

- **`skills/embroidery-digitize/`** — artwork/brief → a stitch plan (`stitch-plan.md` +
  `stitch-plan.json`, a human+machine pair on the same model, mirroring how `tasks` emits
  `_epic.md`/`tracker.md` + `tasks.json`). Decides stitch type/underlay/density/pull-compensation per
  region, every numeric value carrying a recorded source (`doc:<file>` or `user-confirmed`) — never
  a fabricated default.
- **`skills/embroidery-optimize/`** — reorders color blocks + regions to cut jump travel, forced
  trims, and color changes; a content-diff self-check proves only *sequence* changed, never the
  digitized content; hard-ordering constraints (e.g. appliqué's phase sequence) are never reordered.
- **`skills/embroidery-export/`** — the one skill that touches real tooling. Explicitly instructed to
  run an actual open-source library (`pyembroidery` by default, per `file-formats.md`) rather than
  hand-computing bytes, and to degrade honestly (a spec-only document, clearly labeled) rather than
  fabricate a file when no library is available. Round-trip validation (re-parse → compare stitch/
  color-stop count + extents) is its structural self-check — a file that hasn't round-tripped isn't
  considered exported.
- **`skills/embroidery-qa/`** — read-only, cite-or-drop findings against a fixed rule set (density
  range, underlay presence, pull-compensation presence, hoop fit, jump/trim budget, needle-count fit,
  lettering minimums, export round-trip) — the same discipline the `reviewer` agent applies to code,
  applied here to stitch-plan content instead.

Also added `skills/api/templates/embroidery-export.md` earlier (Phase 2) is the **contract** these
skills' actual output is checked against when a product feature declares "Embroidery file export" as
its interface kind — `embroidery-export` (the skill) reads that contract when present; the two are
deliberately distinct (spec-time contract vs. runtime execution).

**Validator-driven correctness, not just self-review:** before writing any skill file, I read
`scripts/validate_plugin.py` in full to find every structural rule it enforces on `skills/*/SKILL.md`
— exact frontmatter shape (`name` must equal the dir name; `model`/`effort`/`agents` required and
validated against fixed sets), the literal phrases `"stage-handoff block"` and `"structural
self-check"` must appear in every skill, dir names must match `^[a-z0-9-]+$`, and — the one that
would have silently broken 5 files — the skill count embedded in `"N atomic"` phrases across
README.md and all 4 plugin/marketplace manifests is dynamically checked against the real
`skills/*/SKILL.md` count. Updated all 5 to `"23 atomic"` (19 pipeline + 4 embroidery) before running
the validator, rather than after discovering the failure. Added an `## Embroidery capability skills`
section to README documenting the 4 new skills (not validator-required, but consistent with how the
rest of the pipeline is documented).

**Result:** `scripts/validate_plugin.py` passes clean at **399/399 checks, exit 0** (up from 359 —
the new skills triggered ~40 additional per-skill structural checks, all passing on the first full
run after the manifest updates).

## 20. Where things stand

All of Phases 0–3 are executed and independently verified by the repo's own validator, not just
self-reported. What's left, in order:

1. **You verify the `<!-- TBD -->` facts** in `docs/domain/embroidery/*.md` against real sources —
   not blocking, but should happen before `embroidery-export`/`embroidery-qa` treat any of those
   numbers as a hard constraint.
2. **A live dry run** of the 4 new skills against a real (even trivial) design — the validator proves
   structural correctness, not that the protocols actually produce sensible output; the first real
   invocation is the test that matters.
3. **Phase 4 (hardening, §13)** — revisit only if real usage shows a concrete need: a parallel
   digitizing-candidate agent, additional domain-doc content, or the GitHub-repo-rename follow-through
   once the separate repo exists.
4. **`server/bun.lock`** — still text-substituted rather than regenerated (§16); a `bun install` pass
   is cheap insurance before this ships anywhere real.
