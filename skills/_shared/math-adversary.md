# Math-adversary overlay — conditional routing for the `mathematic` agent

> **Reference-only.** Not a skill. This file gives the pipeline one shared, evidence-first way to
> route a mathematical or algorithmic decision to [`mathematic`](../../agents/mathematic.md). This
> specialist examines the selected method and recommends a better-grounded method. The normal SDD
> flow and all documentation templates do not change. This file only adds a dispatch point.

## When to activate it

Activate it when the spec, the design, the task or the code under review commits to a nontrivial
algorithm, numerical method, or geometric, statistical or signal-processing pipeline. Examples:

- an optimization, search, clustering or fitting method;
- coordinate or unit conversion math;
- image or signal processing (smoothing, edge/contour detection, curve fitting);
- a scheduling or packing heuristic;
- a probability or statistics model;
- a numeric threshold that is really a calculated value shown as a constant.

Do **not** activate it for ordinary CRUD or business logic that has no real mathematical content.
A `mathematic` review of a simple field mapping is noise, not rigor.

## Why a skill dispatches it, not `critic`/`devils-advocate` themselves

Subagents cannot start subagents. Only the calling skill (the lead) can dispatch an agent (see the
shared contract, point 3, in [`agent-roster.md`](./agent-roster.md)). Thus `critic` and
`devils-advocate` can never call `mathematic` during a run, also when they see a math decision that
needs a second opinion. Before dispatch, the skill decides if the artifact has a math or algorithm
decision to route. If it has one, the skill dispatches `mathematic` itself. It does this as a
**companion pass** together with `critic`/`devils-advocate`, or as a **direct pass** alone.

## How a skill dispatches it

- **Companion pass (alongside `critic` / `devils-advocate`).** If the artifact under a critic or
  devils-advocate pass has a flagged math or algorithm decision, dispatch
  [`mathematic`](../../agents/mathematic.md) — `subagent_type: "sdd-emb:mathematic"` — in the same
  round. Give it the same draft or diff and upstream files that `critic`/`devils-advocate` got.
  Also give the concrete question and the numeric or performance limit that defines "best" here
  (see [`mathematic.md`](../../agents/mathematic.md): a dispatch without a limit gets a vague answer).
  Write `math adversary: active` in **all three** dispatch prompts (also for `critic`/`devils-advocate`).
  Then each agent knows that a specialized companion also ran. It does not do the math question
  again, and it does not drop it silently. Merge the cited findings of `mathematic` into the
  **same resolution flow** that the skill uses for `critic`/`devils-advocate` findings
  (`AskUserQuestion`: Accept revert / Accept amendment / Override-with-rationale).
- **Direct pass.** If a task or a module *is* the mathematical decision, dispatch `mathematic`
  directly on that code or spec slice. Examples: a `data-model` index/partitioning heuristic, a
  `tasks` estimation model, an `implement` numeric routine. Use its recommendation as you use the
  map from `explorer`: it is evidence for the design or the implementation, not a verdict to obey
  without thought. A real performance or scope constraint that the dispatcher did not give to
  `mathematic` can still have priority.
- **Fallback.** If `sdd-emb:mathematic` is not available at runtime, use a `general-purpose` Agent
  with the prompt body from [`mathematic.md`](../../agents/mathematic.md). This is the same rule as
  for all other agents in the roster.
- In each dispatch prompt, tell `mathematic` to write its report in ASD-STE100 →
  [`ste100.md`](./ste100.md).

## Role-specific integration points

- **`survey`:** if the brownfield scan (`explorer`) finds a nontrivial algorithmic or numerical
  module (an existing vectorization, scheduling or optimization routine), record it in the
  Conventions of the architecture map as a cited `file:line` precedent. Add the note "route through
  `mathematic` before extending". `explorer` stays read-only and gives no judgment: it flags the
  module, it does not judge it.
- **`clarify`:** if the self-sweep or the `devils-advocate` Mode-A pass finds a spec clause that
  names or implies a specific algorithm, formula or numeric threshold, dispatch `mathematic` as a
  companion pass on that clause. Its finding does **not** become a ninth ambiguity class. Put it in
  the class that already fits: an unjustified constant is a `vague-term` or an `unmeasured-NFR`
  with a number in place of an adjective. Resolve or defer it through the normal flow.
- **`design`:** if a SAD decision (a §4/§5 building block) commits to an algorithm or a numerical
  approach, dispatch `mathematic` as a companion to the step-7 `critic` pass. For the companion
  trigger, see [`design/references/critic.md`](../design/references/critic.md). If a later reversal
  of the algorithm is expensive, a confirmed finding can cross the blast-radius gate into its own ADR.
- **`data-model`:** if an index, partitioning or sharding strategy, or a computed or derived column,
  contains a nontrivial formula, dispatch `mathematic` directly. Do this before the formula goes
  into `data-model.md`.
- **`tasks`:** if the Definition of Done of a task depends on a correct algorithm (for example, a
  stitch-geometry or vectorization task), dispatch `mathematic` directly. Its recommendation becomes
  the stated approach of the task. Then `test-author`/`implementer` build against a reviewed method,
  not against a guess.
- **`plan-tests`:** if the correctness of an acceptance criterion depends on a numerical or
  algorithmic property (a convergence bound, a numerical tolerance, a complexity budget), dispatch
  `mathematic`. It makes sure that the property is testable as stated. Use its answer to write the
  assertion of the test.
- **`implement`:** if `test-author`/`implementer` will write or wrote a nontrivial numerical routine,
  and no upstream artifact reviewed the approach, dispatch `mathematic` directly during the task. It
  costs less to find the problem before the GATE than at `review`.

## What `mathematic` needs in the dispatch prompt

Give these items:

1. The concrete artifact slice: inlined, or a file path that it must `Read` itself. It does not
   share your context.
2. The specific question ("is k-means the right clustering choice here, and is `k` picked with
   justification?").
3. The domain constraint that sets the limit of "best" (numeric range, performance budget,
   precision requirement).

For its full contract and output format, see [`mathematic.md`](../../agents/mathematic.md).
