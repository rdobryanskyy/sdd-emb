# Stage handoff — what every skill prints when it finishes (the output contract)

> **Reference-only.** Not a skill. **Every** skill ends by emitting the handoff block defined here —
> as its **last output**, after it has proposed its commit. The format lives only in this file; each
> skill keeps a one-line pointer and supplies its own *Що я зробив* / *Перевір перед тим як
> продовжити* / *next command*. This exists because a bare «Далі: …» line is hard to act on — the
> user can't tell what changed, which files to open, or what to run next without scrolling back.

Running under Codex CLI or Cursor? The `/clear` and `/sdd-emb:<next>` forms map to the host tool's
equivalents per [`tool-adapters.md`](./tool-adapters.md).

> **Language.** The block below is printed straight to the user, so per
> [`chat-language.md`](./chat-language.md) its prose is always Ukrainian — that's why the template
> skeleton in *The block (sectioned format)* is written in Ukrainian. Commands, file paths, and
> slugs inside it stay verbatim. The rest of *this file* (the rules that follow) is instructional
> prose for whoever writes a skill and stays English.

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

Rules for filling it:

- **Always emit it** as the final output, once per run, after the commit is proposed. Never end a
  skill on a bare «Далі: X».
- **Що я зробив** — concrete and self-contained: name the files written and the proposed commit
  message, so the user doesn't scroll up to reconstruct it.
- **State the size + route used.** *Що я зробив* names the `feature_size` AND the route the stage
  worked at — «розмір M + маршрут standard (з `.size`/`.route`)»; if the stage had to **default**
  because a file was missing, say so loudly — «розмір M (за замовчуванням — немає `.size`; запусти
  `/sdd-emb:classify-size <slug>`)», «маршрут standard (за замовчуванням — немає `.route`)» — so a
  missing size/route surfaces at this gate, not three stages later. A missing `.route` always means
  `standard` (the pre-route behaviour — fully back-compatible). (`specify` establishes both at the
  start, so this should be rare.)
- **Перевір перед тим як продовжити** — list **every artifact this stage wrote or changed**, each as
  a real `docs/features/<slug>/…` path (or repo-root path like `docs/architecture-map.md`) plus a
  one-liner on what to eyeball. This *is* the per-gate review checklist.
- **Що далі** — the next command in **`/sdd-emb:<name> <slug>`** form inside a fenced code block (so the
  user copies it in one click). `/clear` is step 1 and **mandatory** for a forward backbone handoff.
  Add a `↳ або …` skip-alternative **only** when one genuinely exists (see the table). The
  skip-alternatives come from the **fast-lane N/A conditions** in [`size-matrix.md`](./size-matrix.md);
  **how each resolves is route-dependent** — auto-skip on `quick`, offered on `standard`,
  suppressed on `full` (see the *Route-resolved forward handoff* variant below).
- Keep the `<slug>` substituted with the real slug — never leave the literal `<slug>` in the printed
  block.

## Variants

- **Backbone forward handoff** (`survey → … → review → ship`): `/clear` mandatory + the next stage.
- **Route-resolved forward handoff** (a backbone stage whose successor is an *optional* stage —
  `specify`, `clarify`, `design`, `sequences`, `data-model`, `tasks`): before printing *Що далі*,
  resolve the next stage per `docs/features/<slug>/.route` and the Routes table in
  [`size-matrix.md`](./size-matrix.md):
  - **`quick`** — evaluate the next optional stage's N/A condition yourself. Holds → *Що далі*
    names the post-skip stage, *Що я зробив* states «автоматично пропущено `<stage>`: <reason>», and
    the `↳ або` line **inverts** — it offers the skipped stage («запустити повний шлях»). Doesn't
    hold → normal forward handoff (the stage is not skipped).
  - **`standard`** — normal forward handoff; add the `↳ або` skip-alternative when the N/A
    condition holds (the user picks).
  - **`full`** — normal forward handoff; **never** print an `↳ або` skip line.
  Missing `.route` → `standard`. The route steers handoffs only — a stage invoked directly always
  runs.
- **Loop-back** (`review → implement` on `CHANGES REQUESTED`): **no `/clear`** — you stay in context
  to iterate; *Що далі* = `/sdd-emb:implement <slug>` (fix), then re-review the changed surface.
- **Terminal** (`ship`): there is no `/sdd-emb` successor. *Що далі* becomes **Готово** — the PR command/URL
  + «мердж у main — твоє рішення»; still print *Що я зробив* + *Перевір* (the changelog + PR).
- **Utility** (`classify-size`, `glossary`, `decide-adr`, `roadmap`, `fix`): called ad-hoc, not a
  gate. `/clear` is **optional** (recommend it only if the context is large); *Що далі* = «повернись
  до своєї backbone-стадії», naming the likely one (e.g. `/sdd-emb:design <slug>`). Print *Що я зробив* +
  *Перевір* (the one file it wrote). One exception: `fix` alone adds a **conditional** recommendation —
  when the fix touched >5 files or crossed a module boundary, *Що далі* also offers
  `/sdd-emb:review <slug>` (a recommendation, never a gate).

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

The `↳ or` cells above show the `standard`-route rendering; on `quick` the stage auto-skips (and
the `↳ or` inverts), on `full` the `↳ or` line is dropped — per the *Route-resolved* variant.

## Discipline

- **The block is the last thing printed — every run, no exceptions.** A skill that ends on prose
  without it has regressed.
- **Real paths, not descriptions.** «the SAD» is not reviewable; `docs/features/<slug>/sad.md` is.
- **The next command is copy-ready** — `/sdd-emb:<name> <slug>` in a fenced block, slug substituted.
- **`/clear` only where it's correct** — mandatory on a forward backbone handoff, omitted on a
  loop-back (you're iterating), optional after a utility.
- **Format canonical here** — a skill that hand-rolls its own block shape has duplicated the contract.

## Where each skill calls this

Every skill's final protocol step ends with: «emit the **stage-handoff block** per
[`handoff.md`](./handoff.md)» + its own next command from the table above. The format + variants live
here; the skill supplies only the run-specific content.
