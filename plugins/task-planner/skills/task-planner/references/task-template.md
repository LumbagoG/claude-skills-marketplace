# Task Artifact — Fallback Template

Use this template only when the project doesn't have its own task/spec convention. If it does (e.g. `wiki/tasks/_template.md`), use that instead — consistency with the project beats consistency with this skill.

---

```markdown
---
type: task
project: <project-name>
category: tasks
tags: [<3-5 relevant tags>]
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
status: pending
id: <NNN>
---

# [<NNN>] <Concise task title>

## Status

`🟡 pending` | `🔵 in-progress` | `🔴 blocked` | `✅ done` | `❌ cancelled`

> Текущий статус: **🟡 pending**

---

## Исходный промпт пользователя

> Verbatim copy of the original request — never edited.

​```
<paste user prompt verbatim here>
​```

### Уточнения по итогам диалога

1. **<Topic 1>** — <one-line resolution>.
2. **<Topic 2>** — <one-line resolution>.
3. **<Topic 3>** — <one-line resolution>.

---

## Понимание задачи

<2–4 sentences interpreting what the user actually wants and what the deliverable is. This is your synthesis, not a paraphrase.>

---

## Высокоуровневый план

1. <step 1>
2. <step 2>
3. <step 3-7>

---

## Технический план

### Затронутые слои / файлы

<table from `references/architecture-mapping.md` for the project's architecture>

### Схема данных

​```prisma
// Full Prisma models verbatim — fields, types, defaults, relations, indexes, constraints.
​```

#### Изменения в существующих моделях

​```prisma
model ExistingModel {
  // + new field 1
  // + new field 2
}
​```

### API контракты

​```typescript
// POST /api/<path>
type <Name>Input = { ... };
type <Name>Output = { ... };

// PATCH /api/<path>/[id]
type <Name>UpdateInput = ...;
​```

### Email / Notifications (if applicable)

| Template | Trigger | Recipient |
|--|--|--|
| `<Name>Email` | <event> | <user> |

### Realtime events (if applicable)

| Event | Room | Audience |
|--|--|--|
| `<event:name>` | `<room>_{id}` | <who listens> |

### Routing (if applicable)

​```
/<path>                → <view>
/<path>/[id]           → <view>
/admin/<path>          → admin view
​```

### Архитектурные решения

- **<Decision A>** — chosen instead of <Alt> because <reason>.
- **<Decision B>** — chosen instead of <Alt> because <reason>.
- **<Decision C>** — chosen instead of <Alt> because <reason>.
- (4–8 entries total)

---

## Фазы выполнения

| # | Фаза | Что делает | Owner | Статус |
|--|--|--|--|--|
| 0 | Pre-flight | UX-плана + storybook-аудит для всех новых UI (если применимо) | <agents> | ⏳ |
| 1 | Schema + entities | Models + migration + entity layer | <agent> | ⏳ |
| 2 | <Phase 2 subject> | <what it produces> | <agent> | ⏳ |
| 3 | <…> | <…> | <agent> | ⏳ |
| N-1 | E2E happy path | <test name> | <agent> | ⏳ |
| N | QA-gate | lint + types + tests + screenshots | <agent> | ⏳ |

> Each phase should be independently reviewable and committable.

---

## Риски и GOTCHA

- ⚠️ <risk 1: race / transaction concern>
- ⚠️ <risk 2: cascade delete / nullability>
- ⚠️ <risk 3: project-specific GOTCHA from hot.md that applies>
- ⚠️ <risk 4: index choice that may not match query pattern>
- ⚠️ <risk 5: framework-version trap, if any>
- (5–12 entries — be honest, not exhaustive)

---

## Открытые вопросы

> Things deferred to v2 — captured here so they're not lost.

1. <question or feature deferred and why>
2. <…>

---

## Идеи для улучшения (опционально)

- <nice-to-have 1>
- <nice-to-have 2>

---

## Зависимости

- <upstream package / migration / approval needed>

---

## Связанные wiki-страницы / docs

> To be created/updated after implementation:

- `<path/to/page>.md`
- `<path/to/page>.md`

---

## Результат

> Filled in after completion.

- Commit(s): `<hash>`
- Tests: `N/N PASS`
- Schema changes: yes/no
- Related artifacts: <links>

---

## История изменений

| Дата | Действие |
|--|--|
| <YYYY-MM-DD> | Задача создана; план утверждён через N раундов уточнений. |
```

---

## How to use this template

1. Copy the block above into a new file.
2. Replace placeholders.
3. Fill **only** the sections that apply. If there's no realtime, delete the realtime table; don't leave it empty.
4. Don't add sections that aren't in the template — consistency across artifacts is more valuable than personalization.

## Versioning

When a task evolves significantly mid-flight (scope grows, decisions reverse), don't rewrite history. Add an entry to `## История изменений` and append a new section like:

```markdown
## Дополнение от <YYYY-MM-DD>

After Phase 3 it became clear that <X>. Decision: <Y>. Phases 4-7 adjusted accordingly.
```

This preserves the audit trail and makes it possible to understand how the plan evolved later.
