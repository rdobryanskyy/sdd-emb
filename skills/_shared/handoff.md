# Stage handoff — what every skill prints when it finishes (the output contract)

> **Reference-only.** Not a skill. **Each** skill ends with the handoff block that this file defines.
> The block is the **last output** of the skill, after the skill proposes its commit. The format is
> only in this file. Each skill keeps a one-line pointer and supplies its own *Що я зробив* /
> *Перевір перед тим як продовжити* / *next command*. This file exists because a bare «Далі: …» line
> is difficult to use. Without the block, the user must scroll back to find what changed, which files
> to open, and what to run next.

Do you use Codex CLI or Cursor? Then the `/clear` and `/sdd-emb:<next>` forms map to the equivalents
of the host tool, per [`tool-adapters.md`](./tool-adapters.md).

> **Language.** The skill prints the block below directly to the user. Thus, per
> [`chat-language.md`](./chat-language.md), the prose of the block is always in Ukrainian. For this
> reason, the template in *The block (sectioned format)* is in Ukrainian. Keep the commands, file
> paths and slugs in the block verbatim. The rules in the rest of this file are instructions for the
> author of a skill. They stay in English and follow ASD-STE100 → [`ste100.md`](./ste100.md).

## TL;DR (короткий вступ українською)

Кожен крок (skill) наприкінці **завжди** друкує однаковий хендоф-блок із трьох секцій:

1. **Що я зробив** — що стадія зробила + який коміт запропонувала (не змушуй гортати вгору).
2. **Перевір перед тим як продовжити** — посилання на файли, які стадія створила/змінила і які треба
   глянути на цьому геті (реальні `docs/features/<slug>/…` шляхи — клікабельні/копіювані).
3. **Що далі** — спершу `/clear` (обов'язково для forward-переходу — наступна стадія перечитує
   все з диска), потім наступна команда `/sdd-emb:<next> <slug>` у **fenced-блоці** (копіюється в один
   клік) + альтернатива-пропуск, якщо вона є.

Це прибирає головний біль: «погано виводить, незручно копіювати і перевіряти».

---

## The block (sectioned format)

```md
## ✅ <skill> — <slug>

**Що я зробив**
- <1–3 пункти: який(і) артефакт(и) створено/змінено + який коміт запропоновано>

**Перевір перед тим як продовжити**
- `docs/features/<slug>/<file>` — <що тут перевірити>
- `docs/features/<slug>/<file2>` — <…>

**Що далі**
1. `/clear` — обов'язково (чистий контекст; наступна стадія перечитує все з диска)
2. потім запусти:
   ```
   /sdd-emb:<next> <slug>
   ```
   ↳ або `/sdd-emb:<alt> <slug>`, щоб <умова пропуску>   ← лише коли реально є пропуск
```

Rules for the block:

- **Always write the block** as the final output, one time for each run, after the skill proposes the
  commit. Never end a skill on a bare «Далі: X».
- The prose of the handoff block is in Ukrainian, per [`chat-language.md`](./chat-language.md).
  Ukrainian text is out of the scope of ASD-STE100. If the block contains English text, write it in
  ASD-STE100 → [`ste100.md`](./ste100.md).
- **Що я зробив** — make it concrete and complete. Name the files that the stage wrote and the proposed
  commit message, so that the user does not have to scroll up to find them.
- **State the size + route used.** *Що я зробив* names the `feature_size` AND the route of the stage:
  «розмір M + маршрут standard (з `.size`/`.route`)».
  - If the stage had to use a **default** because a file was missing, say so clearly:
    «розмір M (за замовчуванням — немає `.size`; запусти `/sdd-emb:classify-size <slug>`)»,
    «маршрут standard (за замовчуванням — немає `.route`)».
    Thus the user sees a missing size or route at this gate, not three stages later.
  - A missing `.route` always means `standard` (the behavior before routes existed; fully back-compatible).
  - `specify` sets both at the start, so this occurs rarely.
- **Перевір перед тим як продовжити** — list **each artifact that this stage wrote or changed**. Give each one
  as a real `docs/features/<slug>/…` path (or a repo-root path such as `docs/architecture-map.md`).
  Add one line about what to examine. This list *is* the review checklist for the gate.
- **Що далі** — put the next command in **`/sdd-emb:<name> <slug>`** form inside a fenced code block (so
  that the user can copy it in one click). `/clear` is step 1, and it is **mandatory** for a forward backbone handoff.
  - Add a `↳ або …` skip alternative **only** when one really exists (see the table).
  - The skip alternatives come from the **fast-lane N/A conditions** in [`size-matrix.md`](./size-matrix.md).
  - **The result of each skip alternative depends on the route**: auto-skip on `quick`, offered on
    `standard`, removed on `full` (see the *Route-resolved forward handoff* variant below).
- Replace `<slug>` with the real slug. Never leave the literal `<slug>` in the printed block.

## Variants

- **Backbone forward handoff** (`survey → … → review → ship`): `/clear` is mandatory + the next stage.
- **Route-resolved forward handoff** (a backbone stage whose next stage is *optional*:
  `specify`, `clarify`, `design`, `sequences`, `data-model`, `tasks`): before you print *Що далі*,
  find the next stage from `docs/features/<slug>/.route` and the Routes table in
  [`size-matrix.md`](./size-matrix.md):
  - **`quick`** — examine the N/A condition of the next optional stage yourself.
    - If the condition is true, *Що далі* names the stage after the skipped stage. *Що я зробив* states
      «автоматично пропущено `<stage>`: <reason>». The `↳ або` line **inverts**: it offers the skipped
      stage («запустити повний шлях»).
    - If the condition is not true, use a normal forward handoff (the stage does not skip).
  - **`standard`** — normal forward handoff. When the N/A condition is true, add the `↳ або` skip
    alternative (the user selects).
  - **`full`** — normal forward handoff. **Never** print an `↳ або` skip line.
  Missing `.route` → `standard`. The route controls handoffs only. A stage that you start directly
  always runs.
- **Loop-back** (`review → implement` on `CHANGES REQUESTED`): **no `/clear`**, because you stay in the
  context to iterate. *Що далі* = `/sdd-emb:implement <slug>` (fix), then review the changed surface again.
- **Terminal** (`ship`): there is no `/sdd-emb` successor. *Що далі* becomes **Готово**: the PR command/URL
  + «мердж у main — твоє рішення». Still print *Що я зробив* + *Перевір* (the changelog + PR).
- **Utility** (`classify-size`, `glossary`, `decide-adr`, `roadmap`, `fix`): the user starts these when
  necessary. They are not a gate.
  - `/clear` is **optional**. Recommend it only if the context is large.
  - *Що далі* = «повернись до своєї backbone-стадії». Name the probable stage (for example `/sdd-emb:design <slug>`).
  - Print *Що я зробив* + *Перевір* (the one file that the skill wrote).
  - One exception: only `fix` adds a **conditional** recommendation. When the fix touched >5 files or
    crossed a module boundary, *Що далі* also offers `/sdd-emb:review <slug>`. This is a recommendation,
    never a gate.

## Canonical sequence (stage → review-files → next)

| Stage | Перевір перед тим як продовжити (files written) | Що далі |
|---|---|---|
| `survey` | `docs/architecture-map.md` (+ scaffold `tasks.json` on greenfield) | `/sdd-emb:specify <slug>` |
| `specify` | `docs/features/<slug>/spec.md` | `/sdd-emb:clarify <slug>` ↳ or `/sdd-emb:design <slug>` (XS/S, zero §8 OQ — fast lane) |
| `clarify` | `docs/features/<slug>/spec.md` (tightened) | `/sdd-emb:glossary <slug>` ↳ or `/sdd-emb:design <slug>` |
| `design` | `sad.md` (C4 §3/§5 + `target_surfaces`) + `adr/` | `/sdd-emb:sequences <slug>` ↳ or `/sdd-emb:data-model <slug>` (XS/S, no multi-step flow — fast lane) |
| `sequences` | `sad.md` §6 (flows) | `/sdd-emb:data-model <slug>` ↳ or `/sdd-emb:api <slug>` (XS/S, no schema change — fast lane) |
| `data-model` | `data-model.md` + staged `migrations/` | `/sdd-emb:api <slug>` ↳ or `/sdd-emb:tasks <slug>` (XS/S, no contract change — fast lane) |
| `api` | `contracts/openapi.yaml` (+ `events.md`, `api-sync-report.md`) | `/sdd-emb:tasks <slug>` |
| `tasks` | `tasks/` + `tasks.json` | `/sdd-emb:plan-tests <slug>` ↳ then `/sdd-emb:implement <slug>` |
| `plan-tests` | `test-plan.md` (or `spec.md` `## Test plan` for XS/S) | `/sdd-emb:implement <slug>` |
| `implement` | the committed diff (code + tests) + `tasks/tracker.md` | `/sdd-emb:review <slug>` |
| `review` | `_review/review-<date>.md` | `/sdd-emb:ship <slug>` (PASS) · `/sdd-emb:implement <slug>` (CHANGES, no `/clear`) |
| `ship` | `CHANGELOG` + the PR | **Done** — PR command/URL; merge is your call |
| `classify-size` | `.size` + `.route` | resume — e.g. `/sdd-emb:specify <slug>` |
| `glossary` | `CONTEXT.md` | resume — e.g. `/sdd-emb:design <slug>` |
| `decide-adr` | `adr/NNNN-<title>.md` | resume — `/sdd-emb:tasks <slug>` or `/sdd-emb:plan-tests <slug>` |
| `roadmap` | `docs/roadmap.md` | resume your backbone stage |
| `fix` | `_fixes/<date>-<short>.md` + the diff (+ the spec patch if any) | resume — or `/sdd-emb:review <slug>` when the fix was wide (>5 files / cross-module) |

The `↳ or` cells above show the output for the `standard` route. On `quick`, the stage auto-skips (and
the `↳ or` inverts). On `full`, the `↳ or` line is removed. This is per the *Route-resolved* variant.

## Discipline

- **The block is the last output: each run, no exceptions.** If a skill ends on prose without the block,
  that is a regression.
- **Real paths, not descriptions.** The user cannot review «the SAD». The user can review `docs/features/<slug>/sad.md`.
- **The next command is ready to copy**: `/sdd-emb:<name> <slug>` in a fenced block, with the real slug.
- **Use `/clear` only where it is correct.** It is mandatory on a forward backbone handoff. Do not use it on a
  loop-back (you iterate). It is optional after a utility.
- **The format is canonical here.** If a skill makes its own block shape, it duplicates the contract.

## Where each skill calls this

The final protocol step of each skill ends with: «emit the **stage-handoff block** per
[`handoff.md`](./handoff.md)» + its own next command from the table above. The format and the variants are
in this file. The skill supplies only the content of the run.
