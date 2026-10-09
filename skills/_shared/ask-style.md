# Ask-style — junior-friendly bilingual `AskUserQuestion`

> **Reference-only.** Not a skill. Each skill that calls `AskUserQuestion` reads this file for the
> canonical shape of questions and options. The rule:
>
> - An **option label is the next mechanical step that the skill does**, not only a name.
> - The **description tells, in plain words, what will occur**.
> - Write them so that a first-year junior can select correctly without a senior near them.

> **Volume vs. style.** The **number** of questions that a Q&A skill asks changes with the
> interview-depth dial: easy asks few, hard asks all (see [`interview-depth.md`](./interview-depth.md)).
> The **explanation rule below for each question is the same at all depths**. Even a single
> easy-level question gets glosses and a full explanation. Depth changes the count. It never
> permits a dry question.

## The one rule that matters most

**Never ask dryly.** The most frequent failure is a short question with too much jargon: a few words, some acronyms and no context. To answer it, the user must already know the project. Correct this in two ways, each time:

1. **Gloss each technical term inline, at its first use.** Put the plain meaning in parentheses, at that position. Do not write «order by RICE». Write «order by RICE — a quick score, Reach × Impact × Confidence ÷ Effort, where higher = more value per unit of work». Do not write «forces a worktree». Write «forces a worktree — a separate working copy of the repo so two agents don't edit the same files». The reader must not have to look up a term to select an option.
2. **Use the words for the WHY and the trade-off**, not for the WHAT. A short label is correct. The *description* is the place for the explanation. In plain language, tell what occurs, what you get, what you lose and what the hidden risk is.

If a question looks like a config dump or a part of a spec, it is wrong. Write it as an explanation for a capable new colleague who does not know your acronyms yet. **More explanation is always better than less here.** A long, clear description is a feature, not unnecessary text.

## Shape

- **`question`** — 3–4 sentences in three blocks:
  - **CONTEXT** — why this decision is necessary, what scenario to think about, and what exactly we decide (one sentence with a concrete example).
  - **WHY IT MATTERS** — which quality goal / NFR / spec vector it touches. Is it reversible? Is it irreversible, multi-module, or does it affect performance / security / UX? What is the main trade-off?
  - **READ OPTIONS** — a short request to read the descriptions before the user selects.
- **Each option**:
  - `label` — 1–5 words, in **action form** = the next mechanical step: «Прийняти», «Виправити», «Винести у відкрите питання», «Викинути», «Зафіксувати як ADR». If you recommend the first option, add «(Recommended)» to it.
  - `description` — 3–5 sentences with four mandatory elements (below).

## The four mandatory elements of a `description`

1. **What occurs technically** — use concrete names: tables / endpoints / files / ADR numbers. Do not write «modify the API». Write «add field `is_active BOOLEAN` to table `members` and a new route in the module's handler».
2. **What you get / what you lose** — the trade-off in plain words. **Gloss each technical term**:
   - not «backfill migration» → «a script that walks every existing row and fills the new field; while it runs the rows are read-locked for writes»
   - not «cursor pagination» → «the client sends the last id it saw so the next page starts after it; avoids `OFFSET`, which slows down on large pages»
   - not «GIN index» → «a special index type that lets you search inside JSON columns, but takes 3–5× more space and writes slower»
3. **The next mechanical step of the skill** — «I spawn ADR-NNNN titled X, add a row to the §9 ADR table, the schema is locked for the data-model stage».
4. **Hidden trade-off** — sometimes a condition can make the choice fail. Examples: «only works if Redis is already in your stack», «in 6 months you will need downtime for a backfill», «existing users have to re-login». If such a condition exists, write it **in the description**, not in a later question. A junior will not see that risk without help.

## Language

- **Ukrainian for all text** — labels and descriptions. Technical identifiers stay in their original form (ADR, JSONB, JWT, UUID, FK, OpenAPI), because they are names. The *actions* are Ukrainian («Прийняти», «Відредагувати», «Винести у §11 OQ», «Видалити»).
- Ukrainian text is out of the scope of ASD-STE100. If a question or an option contains English text, write it in ASD-STE100 → [`ste100.md`](./ste100.md).
- You can use glossary roles and the **names** of domain invariants (natural-language phrases, for example «no published lessons»). They are business terms.
- This section controls only the **`AskUserQuestion` conversation** (question and option text). Two
  related topics have a different scope:
  - All *other* chat-facing text (banners, refuse and warn messages, the handoff block) is also
    always Ukrainian → [`chat-language.md`](./chat-language.md).
  - The language of the **documents** is a separate switch for each project: `artifact_language`
    in `.claude/sdd-emb.local.md` → [`artifact-language.md`](./artifact-language.md).

## Forbidden

- Short English labels without context («Approve», «Edit», «Drop», «Reword»).
- Descriptions of one line.
- Technical terms without a gloss (UNION, backfill, GIN, cursor, idempotent, transactional…).
- Trade-offs that you keep for a later question («if you pick this I'll later ask about X, which has complexity Y»).

## Counter-example (deprecated) vs correct

```
# DON'T — opaque next step, no gloss
- label: "Approve"
  description: "Apply decision."

# DO — action-form label, description names the concrete step + glossed trade-off
- label: "Прийняти JSONB-колонку (→ spawn ADR-0002)"
  description: "Одна колонка `body` типу jsonb зберігає весь масив блоків як JSON. ПЛЮСИ: редагування уроку одним UPDATE; новий тип блоку не потребує schema-migration. МІНУСИ: валідація блоків лягає на app-layer (БД не знає типів); пошук всередині body потребує GIN-індексу (спеціальний індекс Postgres для пошуку в JSON — у 3–5× більше місця, повільніший запис). НАСЛІДОК: спавню ADR-0002 з 3 розглянутими варіантами, додаю рядок у §9, схема фіксується для stage data-model."
```

## The 4-state actions, phrased this way (canonical set)

```
- label: "Прийняти як є"
  description: "Лишаю рішення дослівно, запускаю наступну перевірку (gate, якщо є для цієї секції)."
- label: "Виправити"
  description: "Ти даєш нове формулювання/значення; я регенерую рішення під нову умову і питаю ще раз (один раунд — друга відповідь фінальна)."
- label: "Винести у відкрите питання"
  description: "Прибираю рішення з секції і додаю рядок у таблицю Open-Questions з owner+due (питаю наступним кроком). Без обох — рішення стає Drop."
- label: "Викинути"
  description: "Прибираю рішення. Якщо воно обов'язкове — переформулюю опції і питаю ще раз; якщо опціональне — лишаю без заміни."
```

## Dry → explanatory (worked rewrite)

```
# TOO DRY (jargon-dense, no context — the failure to avoid):
Question: "Prioritize Next by RICE or manual?"
Options:
  - label: "RICE"
    description: "RICE score, ordered desc."
  - label: "Manual"
    description: "Manual order."

# EXPLANATORY (context + why + glossed terms — do this):
Question:
  "How do we set the ORDER of the ideas in the «Next» list of the roadmap? Nobody has started
   these ideas yet. This choice only sets which problem we do next. Nothing is final, and you can
   change the order at any time. The trade-off: a score formula is more objective, but it takes
   one minute for each idea. An order by hand is faster, but it changes with your mood. Read
   the two options below."
Options:
  - label: "Score each idea (Recommended)"
    description: "I give each Next idea a RICE score. RICE = Reach (how many users it touches) ×
      Impact (how much it helps, from 3 down to 0.25) × Confidence (how sure we are, as a %) ÷
      Effort (person-weeks, approximate). Each idea gets one number, so «Next» sorts itself by
      value for each unit of effort. You can still change any position by hand. It costs about
      one minute of estimates for each idea."
  - label: "Just order them by hand"
    description: "No formula. You (or I) move the ideas into the order that looks correct. The
      row position = the priority. This is faster and good for a short list. With many ideas,
      the order becomes subjective and it changes over time. If the list becomes long, you can
      change to scores later."
```

The dry version has no answer if you do not know what RICE is. The explanatory version teaches the term in the question and makes the trade-off clear.

## Why (feedback, 2026-05-23 + reinforced 2026-05-29)

The user is a PM, a methodist or a junior developer who opens the repo for the first time. Short English questions do not give them the content of the decision or the difference between the options. Verbatim (2026-05-23): «Треба щоб пояснення були ще більш зрозумілими для людей котрі буквально джуни в розробці». More feedback (2026-05-29): «при опитуваннях треба більш explanatory запитання і варіанти відповідей, бо зараз клод доволі сухо опитує і багато термінів на короткий текст». The dry questions and the high term density did not stop. Thus this file starts with the rule «never ask dryly / gloss every term» above.
