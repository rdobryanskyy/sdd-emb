# Chat language — every skill speaks to the user in Ukrainian

> **Reference-only.** Not a skill. Every skill reads this for the one rule governing **chat-facing
> output** — the text printed directly to the user during a run: narration, banners, confirmations,
> refuse/warn/status messages, and the stage-handoff block. **Skill and agent instruction files
> themselves stay English** — this rule is about what a skill *says*, never about what a skill *is
> written in*. Two related, separately-scoped concerns live elsewhere: the language of
> `AskUserQuestion` prompts → [`ask-style.md`](./ask-style.md) (already Ukrainian); the language of
> **documents** written to disk (spec.md, sad.md, …) → [`artifact-language.md`](./artifact-language.md)
> (a per-project setting, default `en`, unrelated to this file).

## The rule

**All chat-facing output is Ukrainian, always — hardcoded, not a setting.** This is fixed for this
project the same way `ask-style.md` already hardcodes Ukrainian for questions; there is no
`chat_language` key to configure. Concretely, every one of these is Ukrainian:

- Narration and progress lines a skill prints while it works.
- Banners and status summaries (e.g. "here's what I detected / what mode I'm running in").
- Confirmations, warnings, and refuse messages (`«run X first»`, `«Y is undeclared»`, …).
- The stage-handoff block in full — see [`handoff.md`](./handoff.md) for the translated template.
- Any plain-text summary a skill emits instead of (or in addition to) `AskUserQuestion` — e.g.
  `interview`'s final summary, which writes no file and is 100% chat output.

**Skill instructions (`SKILL.md`, `references/*.md`) and agent files (`agents/*.md`) are unaffected**
— they stay English for whoever maintains this plugin. A `SKILL.md` line like `tell the user:
«run `specify <slug>` first»` is an *instruction* written in English that produces a *message*
written in Ukrainian; only the produced message changes.

## Agent reports stay language-neutral

Same precedent as `artifact-language.md`'s "Agent reports" section: a skill dispatching a
report-writing subagent carries the language in the **dispatch prompt** when the report will be
relayed to the user verbatim (e.g. «Write your findings in Ukrainian prose; keep file paths,
identifiers, and verdict literals as-is»). `agents/*.md` themselves stay language-neutral — don't
edit them for this rule.

## Never translate (reused from artifact-language.md's list, applies here too)

Even inside an otherwise-Ukrainian sentence, these stay literal:

- `/sdd-emb:<name> <slug>` commands and any fenced command block — copy-paste must keep working.
- File paths (`docs/features/<slug>/spec.md`, `.claude/sdd-emb.local.md`, …).
- Verdict literals: `PASS`, `CHANGES REQUESTED`, `REVIEW_CLEAN`, `NO_CONTESTED_DECISIONS`.
- Frontmatter keys **and** values (`status: approved`, `dashboard_enabled: true`, `.size` / `.route`
  token values), tracker states (`todo / in_progress / review / done`), task ids (`T<n>`).
- Skill/stage names (`specify`, `design`, `implement`, …), role/surface names
  (`backend-service`), ADR ids (`ADR-0002`), and any other machine token from
  [`artifact-language.md`](./artifact-language.md)'s own list.
- Technical identifiers per [`ask-style.md`](./ask-style.md) (ADR, JSONB, JWT, UUID, FK, OpenAPI, …).

## Diagnostic banners (the `implement` judgment call)

`implement` and its references print `key=value` status lines (e.g.
`mode=<…> tdd=<…> isolation=<…> parallel=<n> integration=<…>`) and a `detected commands:` dump.
Treat these the same way as frontmatter: the **keys and values stay literal English/lowercase
tokens** (they're closer to a machine status line than prose), but the **sentence introducing the
banner** is Ukrainian — e.g. "Виявлені команди:" before the `detected commands:` block, or
"Активний режим:" before the `mode=…` line. Don't translate the token names themselves.

## Precedence

1. **Chat narration is unconditionally Ukrainian** — it does not read `artifact_language` and has
   no override.
2. **A quoted example inside a `SKILL.md` instruction** (e.g. `«run `specify <slug>` first»`) is the
   literal text to print — write those examples in Ukrainian in the instruction file itself, same as
   `handoff.md`'s block template.
3. **When in doubt whether a string is "chat" or "document"**: if nothing on disk parses it back
   (no state-derivation, no dashboard, no downstream skill reads it from a file), it's chat — Ukrainian.
   If a file persists it and something re-reads that heading/token later, it's a document concern —
   defer to `artifact-language.md`.
