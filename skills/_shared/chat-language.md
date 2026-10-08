# Chat language — every skill speaks to the user in Ukrainian

> **Reference-only.** Not a skill. Each skill reads this file for the one rule that controls
> **chat-facing output**. This is the text that a skill shows directly to the user during a run:
> narration, banners, confirmations, refuse, warn and status messages, and the stage-handoff block.
> **The skill and agent instruction files stay in English.** This rule is about what a skill
> *says*, not about the language of the skill file. Two related concerns have a different scope and
> are in other files:
> - The language of `AskUserQuestion` prompts → [`ask-style.md`](./ask-style.md) (already Ukrainian).
> - The language of **documents** on disk (spec.md, sad.md, …) → [`artifact-language.md`](./artifact-language.md)
>   (a per-project setting, default `en`, not related to this file).

## The rule

**All chat-facing output is in Ukrainian, always. This is hardcoded, not a setting.** It is fixed
for this project, the same as `ask-style.md` hardcodes Ukrainian for questions. There is no
`chat_language` key. Each of these items is in Ukrainian:

- The narration and progress lines that a skill shows while it works.
- Banners and status summaries (for example, "here's what I detected / what mode I'm running in").
- Confirmations, warnings and refuse messages (`«run X first»`, `«Y is undeclared»`, …).
- The full stage-handoff block. For the translated template, see [`handoff.md`](./handoff.md).
- Each plain-text summary that a skill gives in place of `AskUserQuestion`, or together with it.
  An example is the final summary of `interview`. It writes no file and is 100% chat output.

**This rule has no effect on skill instructions (`SKILL.md`, `references/*.md`) and agent files
(`agents/*.md`).** They stay in English for the maintainers of this plugin. A `SKILL.md` line such
as `tell the user: «run `specify <slug>` first»` is an *instruction* in English. It makes a
*message* in Ukrainian. Only the message changes.

## Agent reports stay language-neutral

This is the same precedent as the "Agent reports" section of `artifact-language.md`. If a skill
dispatches a subagent that writes a report, and the skill gives this report to the user verbatim,
the **dispatch prompt** sets the language. Example: «Write your findings in Ukrainian prose; keep
file paths, identifiers, and verdict literals as-is». The `agents/*.md` files stay
language-neutral. Do not change them for this rule.

## Never translate (reused from artifact-language.md's list, applies here too)

These items stay literal, also in a sentence that is in Ukrainian:

- `/sdd-emb:<name> <slug>` commands and each fenced command block. Copy-paste must continue to work.
- File paths (`docs/features/<slug>/spec.md`, `.claude/sdd-emb.local.md`, …).
- Verdict literals: `PASS`, `CHANGES REQUESTED`, `REVIEW_CLEAN`, `NO_CONTESTED_DECISIONS`.
- Frontmatter keys **and** values (`status: approved`, `dashboard_enabled: true`, `.size` / `.route`
  token values), tracker states (`todo / in_progress / review / done`), task ids (`T<n>`).
- Skill and stage names (`specify`, `design`, `implement`, …), role and surface names
  (`backend-service`), ADR ids (`ADR-0002`), and all other machine tokens from the list in
  [`artifact-language.md`](./artifact-language.md).
- Technical identifiers per [`ask-style.md`](./ask-style.md) (ADR, JSONB, JWT, UUID, FK, OpenAPI, …).

## Diagnostic banners (the `implement` judgment call)

`implement` and its references show `key=value` status lines (for example,
`mode=<…> tdd=<…> isolation=<…> parallel=<n> integration=<…>`) and a `detected commands:` dump.
Use the same rule as for frontmatter. The **keys and values stay literal English/lowercase
tokens**, because they are machine status lines more than prose. The **sentence before the
banner** is in Ukrainian. Examples: "Виявлені команди:" before the `detected commands:` block, or
"Активний режим:" before the `mode=…` line. Do not translate the token names.

## Precedence

1. **Chat narration is always in Ukrainian.** It does not read `artifact_language` and has no
   override.
2. **A quoted example in a `SKILL.md` instruction** (for example, `«run `specify <slug>` first»`) is
   the literal text to show. Write these examples in Ukrainian in the instruction file, the same as
   the block template in `handoff.md`.
3. **If you are not sure if a string is "chat" or "document", use this test.** If nothing on disk
   parses it again (no state derivation, no dashboard, no downstream skill reads it from a file), it
   is chat: write it in Ukrainian. If a file keeps it and a component reads that heading or token
   again later, it is a document concern: use `artifact-language.md`.
