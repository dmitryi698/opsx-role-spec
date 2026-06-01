# Development Workflow

Практическая инструкция по использованию связки **Karpathy-derived skills + OpenSpec** для разработки в этом проекте.

Для кого: разработчик, работающий в Claude Code CLI. Не для AI — AI читает сами SKILL.md файлы.

## Оглавление

- [Карта инструментов](#карта-инструментов)
- [Сценарий 1: Новая фича от идеи до архива](#сценарий-1-новая-фича-от-идеи-до-архива)
- [Сценарий 2: Мелкая правка или багфикс](#сценарий-2-мелкая-правка-или-багфикс)
- [Сценарий 3: Просто подумать](#сценарий-3-просто-подумать)
- [Шпаргалка: когда что вызывать](#шпаргалка-когда-что-вызывать)
- [Антипаттерны](#антипаттерны)
- [Первый прогон](#первый-прогон)

---

## Карта инструментов

| Этап | Команда | Что делает |
|------|---------|-----------|
| Подумать перед стартом | `/opsx:explore` или `/openspec-explore` | Думающий партнёр, диаграммы, без записи в код |
| Формализовать требования | `/analyst <описание>` | Brownfield read `openspec/specs/`, требования как `[ADDED]/[MODIFIED]/[REMOVED]` |
| Создать change со всеми артефактами | `/opsx:propose <name>` | Генерирует `proposal.md` + `specs/` + `design.md` + `tasks.md` |
| Уточнить план/декомпозицию | `/planner` | Пишет в `design.md` + `tasks.md` (checkbox-формат) |
| Реализовать ОДНУ задачу | `/coder` или `/opsx:apply` | Читает scenarios, пишет код, гоняет Verify, тикает чекбокс |
| Проверить код против контракта | `/reviewer` | Маппит WHEN/THEN → diff, пишет в Decisions log |
| Проверить поведение | `/tester` | Test cases из WHEN/THEN, Vitest + Playwright |
| Финализировать | `/opsx:archive <name>` | Двигает дельты в `openspec/specs/`, переносит change в archive |

---

## Сценарий 1: Новая фича от идеи до архива

Пример сквозной: «добавить отмену заказа со статусом `CANCELLED` и причиной отмены».

### Шаг 0 — (опционально) Подумать вслух

Если идея сырая:

```
/opsx:explore хочу добавить отмену заказа, не уверен куда сохранять причину
```

Claude нарисует state-машину статусов, прочитает существующие спеки, задаст вопросы. Никакого кода не напишет.

**Выход**: понимание скоупа в голове, иногда — drafts в существующих `proposal.md` если change уже есть.

### Шаг 1 — Формализовать требования

```
/analyst добавить возможность отмены заказа со статусом CANCELLED и причиной отмены
```

Claude:
- прочитает `openspec/specs/orders/spec.md` (чтобы не дублировать существующие требования)
- прочитает `openspec/config.yaml` (workspaceId scoping, RBAC, Zod-валидация)
- выдаст требования в форме `[ADDED] / [MODIFIED] / [REMOVED]` с цитированием существующих строк спеки
- завершит хендоффом: «Ready for `/opsx:propose order-cancellation`»

**Выход**: список требований и имя будущего change.

### Шаг 2 — Создать change со всеми артефактами

```
/opsx:propose order-cancellation
```

Claude:
- создаст директорию `openspec/changes/order-cancellation/`
- сгенерирует `proposal.md`, `specs/orders/spec-delta.md` со сценариями в формате WHEN/THEN, `design.md`, `tasks.md`
- покажет coverage check (что покрыто сценариями, что нет)

**Выход**: полный набор артефактов на диске. С этого момента активный change есть, и negative-trigger'ы в скилах работают.

### Шаг 3 — (опционально) Уточнить план

Если `tasks.md` от `/opsx:propose` нарезан крупно или не хватает Verify-команд:

```
/planner уточни план для order-cancellation
```

Claude перепишет `tasks.md` в checkbox-формате с `What/Verify/Depends/Scenario` и добавит tradeoffs в `design.md` → `## Decisions log`.

Обычно этот шаг пропускается — `/opsx:propose` уже даёт рабочий план. Заходи сюда только если декомпозиция неудачная.

### Шаг 4 — Реализовать задачи

Два варианта:

**Вариант А — автономно через OpenSpec:**

```
/opsx:apply order-cancellation
```

Скил идёт по `tasks.md` сверху вниз: для каждой задачи читает scenarios → реализует → запускает Verify → отмечает `- [x]`. Останавливается на блокерах и непонятных моментах.

**Вариант Б — задача за задачей вручную:**

```
/coder реализуй задачу 1 из order-cancellation
```

После задачи 1 — обязательно review (шаг 5) перед задачей 2. Coder сам не лезет в задачу 2.

**Когда что выбрать**:
- `/opsx:apply` — для линейных фич без сюрпризов, простые CRUD'ы
- `/coder` поштучно — когда каждая задача нетривиальна и хочется review/test после каждой

### Шаг 5 — Review после каждой задачи

```
/reviewer
```

Claude:
- прочитает `git diff` против базы change'а
- прочитает `specs/orders/spec-delta.md` со сценариями
- построит обязательную таблицу **Contract coverage** (каждый WHEN/THEN → строка кода)
- запишет non-obvious решения в `design.md` → `## Decisions log` префиксом `[review YYYY-MM-DD]`
- выдаст `APPROVE` / `REQUEST CHANGES` / `APPROVE WITH COMMENTS`

**Если `REQUEST CHANGES`** — возвращаешься на coder:

```
/coder fix R-3, R-5
```

Coder адресует именно эти ID, не расширяет скоуп. Потом снова `/reviewer`.

**Если `APPROVE`** или `APPROVE WITH COMMENTS` → идёшь на tester.

### Шаг 6 — Test после approve

```
/tester
```

Claude:
- возьмёт WHEN/THEN сценарии из `spec.md` как seed для test cases (одна строка таблицы на сценарий, с ID `S-N`)
- запустит `npm test` (Vitest) для логики, `npm run test:e2e` (Playwright) для UI flow
- выдаст таблицу с PASS/FAIL/manual по каждому сценарию
- `PASS` → «Ready for `/opsx:archive order-cancellation`»
- `FAIL` → «Fix bug 2 — re-enter coder»

**Re-entry на FAIL**: `/coder fix bug 2`, затем снова `/tester`. Не возвращайся на analyst — test failures про код, не про требования.

### Шаг 7 — Архивация

Когда tester дал PASS и все чекбоксы в `tasks.md` стоят `[x]`:

```
/opsx:archive order-cancellation
```

Claude:
- проверит что артефакты завершены
- предложит синхронизировать дельты из `changes/order-cancellation/specs/` в основной `openspec/specs/orders/spec.md` (соглашайся — это и есть продвижение спеки в истину)
- перенесёт change в `openspec/changes/archive/YYYY-MM-DD-order-cancellation/`

С этого момента следующий `/analyst` будет читать `specs/orders/spec.md` уже с отменой заказа как baseline.

---

## Сценарий 2: Мелкая правка или багфикс

Когда правка маленькая (баг, опечатка, обновление константы) — короткий путь:

```
/opsx:propose fix-order-totals-rounding
# Claude задаст пару вопросов и создаст артефакты

/opsx:apply fix-order-totals-rounding
# одна задача, реализуется автоматически

/reviewer
# обычно APPROVE с одной строкой issue

/tester
# одна-две test row из спеки

/opsx:archive fix-order-totals-rounding
```

Это **5 команд** на багфикс. Минимум, который сохраняет контракт-дисциплину.

### Совсем без OpenSpec

Допустимо только для:
- опечаток в строковых литералах
- обновлений констант, не меняющих поведение
- стилей/форматирования без изменения логики

```
/coder поправь округление в src/lib/api/orders.ts строка 234
```

Coder в standalone mode пропустит чтение спек, сделает правку, прогонит Verify. **Без review/test/archive — на свой риск.** Любое функциональное изменение должно идти через OpenSpec.

---

## Сценарий 3: Просто подумать

```
/opsx:explore
```

Описывай идею в свободной форме. Можно несколько итераций. Когда созреет — выходи к `/analyst` или сразу `/opsx:propose`.

В explore-режиме Claude **не пишет код**, но может создать OpenSpec артефакты (proposal, design, specs) если попросишь.

---

## Шпаргалка: когда что вызывать

```
Идея сырая             → /opsx:explore
Идея ясна, нет change  → /analyst → /opsx:propose <name>
Change есть, плохой план → /planner
Change есть, готов писать код → /opsx:apply  ИЛИ  /coder
После каждой задачи    → /reviewer → /tester
Все [x] и PASS         → /opsx:archive
REQUEST CHANGES        → /coder fix R-N
FAIL                   → /coder fix bug N
```

### Re-entry правила (важно)

| Ситуация | Re-entry на | НЕ на |
|----------|-------------|-------|
| `REQUEST CHANGES` от reviewer | `/coder fix R-N` | analyst, planner |
| `FAIL` от tester | `/coder fix bug N` | analyst, planner |
| Требования оказались неверны | `/analyst` + правка `proposal.md` | coder |
| Архитектура оказалась неверна | `/planner` + правка `design.md` | coder |

Правило: feedback по коду = re-entry на coder. Feedback по требованиям/архитектуре = re-entry на соответствующую верхнюю роль с явной правкой артефакта.

---

## Антипаттерны

1. **Не вызывать `/coder` напрямую при активном change без `tasks.md`** — он не поймёт, какую задачу делать. Сначала `/opsx:propose`.
2. **Не пропускать `/reviewer` между задачами** — это разрушает Decisions log и пропускает blocker'ы. Даже на простых задачах reviewer выдаёт APPROVE за 30 секунд.
3. **Не запускать `/analyst` повторно после FAIL** — re-entry это `/coder`, не `/analyst`. Иначе перезапустишь весь пайплайн с нуля.
4. **Не править `openspec/specs/` руками** — они продвигаются только через `/opsx:archive`. Если правишь руками — следующий `/analyst` прочтёт неконсистентное состояние.
5. **Не игнорировать `Verify run` у coder** — он реально запускает `npm test`/`npm run build`. Если падает — значит задача не закрыта, не помечай `[x]` пока не пройдёт.
6. **Не писать в `design.md` Decisions log руками** — туда пишут reviewer и coder по ходу работы. Ручные записи путают auto-generated формат и могут быть переписаны.
7. **Не вызывать две роли одновременно** — у каждой роли есть negative trigger «Do not invoke if active OpenSpec change exists and a different role's artifact is the next expected output». Уважай его: вызывай только ту роль, чей артефакт следующий.

---

## Первый прогон

Чтобы убедиться что вся связка работает на проекте — выбери реальную небольшую задачу из бэклога и пройди её через все 7 шагов сценария 1. Например:

- «добавить поле `notes` в заказ»
- «добавить фильтр по дате в список клиентов»
- «добавить тип `EMERGENCY` в приоритеты заказа»

Это занимает ~30 минут и закрывает все стыки: brownfield read, propose, scenarios, checkbox flip, Decisions log, archive. После прогона будет видно, где пайплайн жмёт лично у тебя и где надо подкрутить.

---

## Где живут какие решения

| Тип решения | Файл |
|-------------|------|
| Per-change tradeoffs, invariants, constraints | `openspec/changes/<name>/design.md` → `## Decisions log` |
| Project tech conventions (стек, структура, naming) | `openspec/config.yaml` |
| Pipeline-meta правила (re-entry, hand-off, scope правил) | `.claude/skills/karpathy-guidelines/SKILL.md` |
| Канонические сценарии capability | `openspec/specs/<capability>/spec.md` |
| Что было сделано / прогресс по задачам | `openspec/changes/<name>/tasks.md` чекбоксы |

Других мест для решений нет. Если решение не помещается в одну из категорий — surface для обсуждения, не создавай новых файлов.
