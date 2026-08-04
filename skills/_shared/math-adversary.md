# Math-adversary overlay — conditional routing for the `mathematic` agent

> **Reference-only.** Not a skill. Gives the pipeline a shared, evidence-first way to route a
> mathematical/algorithmic decision to [`mathematic`](../../agents/mathematic.md) — the specialist
> that challenges the chosen method and proposes a better-grounded one. The normal SDD flow and
> every documentation template remain unchanged; this only adds a dispatch point.

## When to activate it

Activate when the spec, design, task, or code under review commits to a nontrivial algorithm,
numerical method, or geometric/statistical/signal-processing pipeline: an optimization/search or
clustering/fitting method, coordinate/unit conversion math, image or signal processing (smoothing,
edge/contour detection, curve fitting), a scheduling/packing heuristic, a probability or statistics
model, or a numeric threshold that is really a computed constant dressed up as a constant. Do
**not** activate for ordinary CRUD/business logic that carries no real mathematical content —
`mathematic` reviewing a straightforward field mapping is noise, not rigor.

## Why a skill dispatches it, not `critic`/`devils-advocate` themselves

Subagents cannot spawn subagents — only the calling skill (the lead) may dispatch an agent (see the
shared contract, point 3, in [`agent-roster.md`](./agent-roster.md)). So `critic` and
`devils-advocate` can never call `mathematic` mid-run even when they spot a math decision worth a
second opinion. The skill is the one that decides, before dispatch, whether the artifact carries a
math/algorithm decision worth routing, and if so dispatches `mathematic` itself — either as a
**companion pass** alongside `critic`/`devils-advocate`, or as a **direct pass** on its own.

## How a skill dispatches it

- **Companion pass (alongside `critic` / `devils-advocate`).** When the artifact under a critic or
  devils-advocate pass contains a flagged math/algorithm decision, dispatch
  [`mathematic`](../../agents/mathematic.md) — `subagent_type: "sdd-emb:mathematic"` — in the same
  round, with the same draft/diff + upstream files `critic`/`devils-advocate` received, plus the
  concrete question and the numeric/performance bound that decides what "best" means here (see
  [`mathematic.md`](../../agents/mathematic.md) — a dispatch with no bound gets a vague answer).
  State `math adversary: active` in **all three** dispatch prompts (`critic`/`devils-advocate` too)
  so each knows a specialized companion also ran and does not duplicate or silently drop the math
  question. Merge `mathematic`'s cited findings into the **same resolution flow** the skill already
  runs for `critic`/`devils-advocate` findings (`AskUserQuestion`: Accept revert / Accept amendment /
  Override-with-rationale).
- **Direct pass.** When a task or module *is* the mathematical decision (a `data-model` index/
  partitioning heuristic, a `tasks` estimation model, an `implement` numeric routine), dispatch
  `mathematic` directly on that code/spec slice. Treat its recommendation as you would `explorer`'s
  map — evidence to design/implement against, not a verdict to blindly obey; a genuine performance
  or scope constraint the dispatcher didn't mention to `mathematic` can still win.
- **Fallback.** If `sdd-emb:mathematic` is unavailable at runtime, fall back to a `general-purpose`
  Agent carrying the prompt body from [`mathematic.md`](../../agents/mathematic.md) — same rule as
  every other agent in the roster.

## Role-specific integration points

- **`survey`:** when the brownfield scan (`explorer`) surfaces a nontrivial algorithmic/numerical
  module (an existing vectorization, scheduling, or optimization routine), record it in the
  architecture map's Conventions as a cited `file:line` precedent and note "route through
  `mathematic` before extending" — `explorer` stays read-only/no-judgment; it flags, it doesn't judge.
- **`clarify`:** when the self-sweep or the `devils-advocate` Mode-A pass surfaces a spec clause
  that names or implies a specific algorithm/formula/numeric threshold, dispatch `mathematic` as a
  companion pass on that clause. Its finding does **not** become a ninth ambiguity class — fold it
  into the class that already fits (an unjustified constant is `vague-term` or `unmeasured-NFR`
  wearing a number instead of an adjective) and resolve/defer it through the normal flow.
- **`design`:** when a SAD decision (a §4/§5 building block) commits to an algorithm or numerical
  approach, dispatch `mathematic` as a companion to the step-7 `critic` pass (see
  [`design/references/critic.md`](../design/references/critic.md) for the companion trigger detail);
  a confirmed finding can itself cross the blast-radius gate into its own ADR if reversing the
  algorithm choice later would be expensive.
- **`data-model`:** dispatch `mathematic` directly when an index/partitioning/sharding strategy or a
  computed/derived column embeds a nontrivial formula — before it's written into `data-model.md`.
- **`tasks`:** dispatch `mathematic` directly when a task's Definition of Done depends on a specific
  algorithm being correct (e.g. a stitch-geometry or vectorization task) — its recommendation
  becomes the task's stated approach, so `test-author`/`implementer` build against a reviewed method
  instead of an unreviewed guess.
- **`plan-tests`:** when an acceptance criterion's correctness hinges on a numerical/algorithmic
  property (a convergence bound, a numerical tolerance, a complexity budget), dispatch `mathematic`
  to confirm the property is even testable as stated, and let its answer shape the test's assertion.
- **`implement`:** dispatch `mathematic` directly, mid-task, when `test-author`/`implementer` is
  about to write or has written a nontrivial numerical routine and no upstream artifact already
  reviewed the approach — cheaper to catch before the GATE than at `review`.

## What `mathematic` needs in the dispatch prompt

The concrete artifact slice (inlined, or a file path it must `Read` itself — it does not share your
context), the specific question ("is k-means the right clustering choice here, and is `k` picked
with justification?"), and the domain constraint that bounds "best" (numeric range, performance
budget, precision requirement). See [`mathematic.md`](../../agents/mathematic.md) for its full
contract and output format.
