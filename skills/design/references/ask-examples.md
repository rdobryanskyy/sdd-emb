# design-specific `AskUserQuestion` shapes

The canonical question/option contract — junior-friendly, bilingual, label = next mechanical step, description = 3–5 sentences with the four mandatory elements — lives in [`../../_shared/ask-style.md`](../../_shared/ask-style.md). Read that first. This file keeps only the **design-specific shapes** that aren't in the shared file: the strategic-decision-with-ADR-spawn, the blast-radius gate, and the Save-as-OQ follow-up. Examples are stack-agnostic — substitute your repo's real names.

## Strategic decision (§4) — option labels name the ADR spawn

```
Question:
  §4 Solution Strategy — як два модулі спілкуються між собою?
  CONTEXT: модуль A записує рядок, модуль B має відреагувати (наприклад, надіслати сповіщення).
  Вирішуємо: B викликається одразу (inline) чи реагує на подію пізніше.
  WHY IT MATTERS: це незворотне рішення (змінити його після накопичення даних — міграція на кілька
  тижнів) і зачіпає кілька модулів (змінює контракт, який бачать і A, і B) — blast-radius ≈ 3/3,
  тож це заспавнить ADR. Компроміс: зв'язаність (coupling) проти додаткових рухомих частин.
  Прочитай описи опцій перед вибором.

Options:
  - label: "Асинхронні події (Recommended) (→ spawn ADR-0001)"
    description: "A записує свій рядок і публікує подію; B споживає її у фоні. ПЛЮСИ: B може лежати, не блокуючи записи A — підтримує ціль доступності; модулі деплояться незалежно. МІНУСИ: потрібен механізм доставки подій (таблиця, куди A пише події в тій самій транзакції, плюс воркер, що їх читає й розсилає — ~150 LOC) та обробка eventual-consistency (дані стають узгодженими не миттєво, а з невеликою затримкою). НАСЛІДОК: спавню ADR-0001 у формі рішення, додаю рядок у §9, форма інтеграції фіксується для стадії `data-model`. ПРИХОВАНЕ: варте того лише якщо декаплінг реально потрібен — для одного in-process виклику це over-engineering (зайва складність)."
  - label: "Синхронний виклик (→ spawn ADR-0001)"
    description: "A викликає B напряму і чекає результат. ПЛЮСИ: найпростіше для розуміння, без додаткової інфраструктури, строге read-after-write (читання одразу бачить щойно записане). МІНУСИ: запис A падає щоразу, коли B недоступний — зв'язує їхню доступність; зв'язує їхні цикли деплою. НАСЛІДОК: спавню ADR-0001 із цим варіантом як обраним і альтернативами занотованими, додаю рядок у §9. ПРИХОВАНЕ: нормально, доки B не стане повільним або нестабільним — тоді A успадковує інциденти B."
  - label: "Винести у відкрите питання"
    description: "Прибираю це рішення з §4 і додаю рядок у §11 Risks: «Відкрите архітектурне рішення: інтеграція модулів — Open question — Resolve before `data-model` — owner: <ти>». Далі питаю owner + due. Без обох рішення стає Drop. Без ADR — відкладене рішення не є прийнятим."
  - label: "Викинути і переформулювати"
    description: "Відкидаю цей набір опцій і питаю ще раз з переформульованим набором (наприклад, лише синхронні варіанти, якщо ти відкинув асинхронний). Використовуй це, коли набору бракує виміру, який для тебе важливий. Це рішення обов'язкове, тож другий Drop ескалує у Save-as-OQ із запропонованим owner."
```

## Blast-radius gate (after an Approve, on a 1-of-3 borderline)

When the gate scores **2+**, spawn the ADR without asking. Only on a **1-of-3 borderline** do you ask:

```
Question:
  Перевірка blast-radius після того, як ти прийняв «<обраний варіант>».
  CONTEXT: набрано 1 з 3 — є легітимні альтернативи, але рішення зворотне і лишається в межах
  одного модуля. Вирішуємо, чи все ж заслуговує на окремий файл ADR.
  WHY IT MATTERS: ADR — для рішень, які варто перечитати через півроку; надмірне ADR-ування
  розмиває жанр, недостатнє — втрачає «чому». Прочитай варіанти.

Options:
  - label: "Зафіксувати як ADR"
    description: "Створюю adr/NNNN-<decision-in-kebab>.md з варіантів, які ти бачив (включно з відкинутими) + твоїм обґрунтуванням, Status Accepted, додаю рядок у §9. Файл іде в коміт цієї секції (або в її батч на quick+easy). Обирай це, якщо рішення справді було дискусійним."
  - label: "Лишити inline"
    description: "Записую рішення прямо в тіло секції з одним рядком обґрунтування, без файлу ADR. Обирай це, коли blast-radius рішення малий, попри наявність альтернатив — типово для §8 crosscutting або локального рішення в §5."
```

## Save-as-OQ follow-up (capture owner + due)

Fired immediately after any section resolves to Save-as-OQ:

```
Question:
  Рішення переноситься в §11 Open Decisions. Вкажи owner і due — дату (YYYY-MM-DD) або
  тригер-стадію на кшталт «before `tasks`». Обидва поля обов'язкові; без обох рішення стає
  Drop і в §11 нічого не лишається.

Options:
  - label: "Вказати owner + due"
    description: "Ти пишеш «owner: <ім'я/роль>, due: <дата або стадія>» одним рядком; я записую це у рядок §11 (severity = Open question), щоб відкладене рішення лишалось відновлюваним до цього тригера."
  - label: "Скасувати — Drop замість цього"
    description: "Скасовую перенесення в OQ і застосовую Drop — рішення прибирається з секції, рядок у §11 не створюється. Edits-log фіксує це як drop."
```
