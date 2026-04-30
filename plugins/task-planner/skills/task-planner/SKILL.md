---
name: task-planner
description: Turn a vague feature idea into a rigorous, implementation-ready plan. Triggers when the user asks to "spec out", "plan", "decompose", "architect", "design", or "break down" a feature, or says things like "I want to build X — give me a plan", "let's spec this", "draft an implementation plan", "написать план задачи", "спланируй мне фичу", "распиши задачу". Especially appropriate when the request is multi-area (DB + API + UI + email + realtime), when the project has an existing architecture (FSD, hexagonal, clean) that the plan must respect, or when the deliverable is a wiki/spec artifact. Use this skill instead of jumping straight to code — it forces clarifying questions first, surfaces hidden trade-offs, and produces a phased plan with explicit risks.
---

# Task Planner

A workflow for converting a fuzzy "I want feature X" into a complete implementation plan that respects the existing project architecture and ends with a versioned artifact (wiki task page).

## Why this skill exists

Without structure, Claude tends to start coding immediately on the most visible part of a request, miss cross-cutting concerns (auth, realtime, email, i18n), and produce plans that don't fit the project's existing patterns (FSD layering, naming conventions, glass-style cards, etc.). The result is rework.

This skill enforces a slow start — clarify, map to architecture, phase, document — so the eventual implementation is straight-line.

## When to skip this skill

- Pure conversation / factual answer ("what does this regex do?")
- Single-file mechanical edits ("rename this variable everywhere")
- The user already has a written spec and just wants code

If in doubt, run the **Phase 0 trigger check** below; if it fails on more than one criterion, don't trigger.

### Phase 0: trigger check

Skip this skill if ALL of:
- Single area (only UI, or only DB)
- No new domain concepts being introduced
- No new external integrations (email, socket, S3, third-party API)
- The user explicitly says "just write the code"

Use this skill if ANY of:
- New domain entity / Prisma model
- Cross-area (DB + API + UI + at least one of email/socket/i18n)
- "Plan" / "spec" / "design" / "architect" / "распиши" / "спланируй" verb
- The project has a wiki/spec convention to populate

## The five-phase workflow

The whole flow is: **discover → clarify → design → phase → document**. Don't compress phases. The clarify phase is where the most value is created.

### Phase 1 — Discover (silent project scan)

Before asking the user anything, read the project's hot context. The goal is to be briefed enough to ask **specific** questions instead of generic ones.

Read in this order, stopping when you have enough:
1. **Hot context** — typically `wiki/_meta/hot.md`, `CLAUDE.md`, or `README.md`. Get a feel for stack, current focus, and recent work.
2. **Task index** — `wiki/tasks/index.md` if exists. **CRITICAL — see Phase 1.5 below for the business-draft handoff protocol.**
3. **Project index** — `wiki/_meta/index.md` or equivalent. See what entities/features already exist that this task might touch.
4. **Existing similar artifacts** — if the user wants "feature X", scan for entities/features with similar shape. Examples: a "trainer application" task → read `db-club-application.md` + `feature-club-applications.md` first because the pattern is the same. A "leaderboard widget" → read existing widget patterns.
5. **Schema** — open the schema file (Prisma, SQL, etc.) and skim relevant sections. You don't need to memorize all 40 models — just the ones in the orbit of the new feature.

Budget: ≤ 8000 tokens of project context. Don't read the whole codebase.

If the project has no wiki / index / schema, just read the README and ask the user where to find architectural context.

### Phase 1.5 — Business-draft handoff (auto-detection)

**Always run this phase if `wiki/tasks/index.md` exists.** It bridges the non-technical partner's planning skill (`task-business-planner`) and this technical one.

#### 1.5.1 — Scan for business-drafts

Open `wiki/tasks/index.md` and look for the section **"🔮 Бизнес-черновики"** (or `business-drafts` in English). Each row references a task file with frontmatter `status: business-draft` and `ready_for_dev: false`.

Read the title and 1–2 sentence summary of each draft. You're checking whether the user's current request semantically matches one of them.

#### 1.5.2 — Match the request

Test for a match using the user's prompt:

- **Strong match** (recommended action: confirm and use): the user's request is essentially a paraphrase of an existing business-draft title or it explicitly references one ("распиши техплан задачи 002", "возьми бизнес-черновик о рассылках в работу").
- **Weak match** (recommended action: ask): the request shares a topic or persona with a draft but is broader / narrower (e.g. user asks about "уведомления тренеров" and there's a draft on "рассылка тренеров"). Surface it as a possibility rather than assuming.
- **No match**: proceed with regular Phase 2 clarification.

#### 1.5.3 — Strong match flow

If you found a strong match, **skip Phase 2 entirely** (the business questions are already answered in the draft) and tell the user:

> "Нашёл бизнес-черновик `wiki/tasks/<file>.md` от партнёра — '<title>'. Беру его за основу и сразу задам технические уточнения."

Then:
- Read the draft file in full.
- Treat the "Резюме фичи", "Кто пользователь", "Метрики успеха", "Сценарии использования", "Как выглядит" sections as **already-resolved business context**. Don't re-ask those.
- Treat the "Открытые бизнес-вопросы" section as **partner-deferred decisions** — surface them in the technical plan as risks or as questions for the user (the developer) to confirm.
- Jump directly into Phase 3 (technical clarification) — focused only on technical dimensions: schema modeling, cardinality, lifecycle, integration with existing models.

#### 1.5.4 — Weak match flow

If you found a weak match, ask one targeted question via the structured-question tool **before** Phase 2:

> "Нашёл похожий бизнес-черновик от партнёра: '<title>'. Это та же задача или другая?"
>
> Options:
> 1. Та же — берём в работу, дополняем техническим планом.
> 2. Связана, но это другая фича — учти её как зависимость.
> 3. Не связана.

Branch on the answer.

#### 1.5.5 — When you finish the technical plan

If you used a business-draft (strong match), the artifact is the **same file** the partner created. You don't make a new one — you append technical sections.

After writing the technical plan, also:
- Change frontmatter: `status: business-draft` → `status: pending`, `ready_for_dev: false` → `ready_for_dev: true`.
- Move the row in `wiki/tasks/index.md` from "🔮 Бизнес-черновики" to "🟡 Очередь".
- Append a row to "История изменений":
  > `| YYYY-MM-DD | Дополнено техническим планом через task-planner. Status: business-draft → pending. |`

This keeps a clean audit trail: the same file shows the partner's original framing, the technical refinement, and the eventual implementation result.

### Phase 2 — Clarify (interactive)

Use the project's `AskUserQuestion`-style tool (or numbered questions if no tool available). Ask **2–4 questions per round**, max **2 rounds** before drafting the plan. The questions must be:

- **Specific to the user's request, informed by Phase 1** — not generic ("public or private?"). Use names from the actual schema.
- **Decisive** — each question's answer must change a concrete part of the plan (a model, an enum, a phase order). If the answer doesn't change the plan, don't ask.
- **Pre-loaded with a recommendation** — mark the option you'd choose with "(Recommended)" and put it first. The user usually accepts the recommendation; the question exists to flush out the cases where they wouldn't.
- **Honest about trade-offs** — describe each option's downside, not just its upside. Users are better at picking from a real trade-off than from a marketing pitch.

See `references/clarifying-questions-guide.md` for the four classes of questions to cover and ready-to-adapt templates.

After Round 1, if the user's answer reveals an inconsistency (e.g., they want X but their other choice forbids X), call it out **before** drafting:

> "One nuance: you picked A and also said the system can never have B, but A implies B in case Y. Want to revisit?"

This is the highest-leverage moment in the whole workflow. Don't skip it.

### Phase 3 — Design (data + API + architecture)

Now produce the technical plan. Do it in this order — schema first, because everything downstream depends on it.

#### 3.1 Schema

For each new entity:
- Show the full Prisma (or SQL) model verbatim — fields, types, defaults, relations, indexes, constraints. Don't hand-wave with "// usual fields here".
- For each new enum, list every value and what triggers transitions to it.
- Note relations on existing models that need to grow (`User`, `Tournament`, etc.). Show the `+ field` diff.
- Include `@@unique` and `@@index` decisions and **explain why** — index choice is a small but real architectural decision.
- Call out any data migrations needed (backfill, NOT NULL after backfill, etc.).

#### 3.2 API contracts

For each new endpoint:
- Method + path
- Input TS type (or zod schema)
- Output TS type
- Auth requirements (role, ownership)

For server actions, the same in `"use server"` form.

#### 3.3 FSD / architecture mapping

Map the work to the project's layering. For FSD specifically, list every slice you'll touch:

| Layer | Path | Action |
|--|--|--|
| db | `prisma/schema.prisma` | + 5 models, + 4 enum |
| entity | `src/entities/<name>/` | create |
| feature | `src/features/<name>/` | create |
| widget | `src/widgets/<name>/` | create |
| view | `src/views/<name>/` | create |
| route | `app/[locale]/<path>/page.tsx` | create / edit |
| api | `app/api/<path>/route.ts` | create |

For non-FSD projects, adapt the column names — but keep the table. Seeing every touched file in one place is the value.

See `references/architecture-mapping.md` for templates per architecture (FSD, hexagonal, clean, MVC, plain Next.js).

#### 3.4 Architectural decisions

Write 4–8 bullet points of the form **"X — chosen instead of Y because Z"**. These are the trade-offs you've already resolved on the user's behalf. If you're not sure why a decision was made, it's not yet a decision.

Examples of what belongs here:
- Why a separate model vs. extending an existing one
- Why a denormalized counter vs. a query
- Why an enum vs. a string vs. a separate table
- Why an event over a direct call

#### 3.5 Risks and GOTCHA

A flat list of warnings the implementer will thank you for. Each is one line, starting with ⚠️. Cover at minimum:

- Race conditions on writes that need transactions
- Cascade-delete pitfalls
- Index choices that don't match query patterns
- Existing GOTCHA from the project's hot context that apply
- Unique-constraint vs. application-level uniqueness

Pull from the project's existing GOTCHA list (e.g. `wiki/_meta/hot.md` "GOTCHA" sections). If the project has known traps with the framework version (HeroUI v3 broken tokens, Tailwind v4 cascade, etc.), surface the ones that apply.

### Phase 4 — Phase the work

Decompose into 8–14 numbered phases, each with:
- A short subject ("Schema + entities", "Application flow", "Reviews")
- What it produces
- Which agent / role owns it (if the project uses an agent team)
- A status column (⏳ / 🔵 / ✅)

Order phases so each one is reviewable independently. Schema migration should usually be Phase 1 and committable on its own.

If the project has a UX-first rule (e.g. "always run ux-senior + storybook-expert before any UI feature"), respect it — slot Phase 0 with both agents in parallel.

End with a **QA gate** phase: lint + types + tests + screenshots. This is non-negotiable for any non-trivial change.

### Phase 5 — Document (the artifact)

Produce a single durable artifact. Look for the project's task-template convention:
- elo-tt-style: `wiki/tasks/YYYY-MM-DD-NNN-short-name.md` from `wiki/tasks/_template.md`
- Linear / Notion: a single Markdown doc the user can paste
- ADR: `docs/adr/NNN-decision-name.md`

If the project has a template, use it verbatim. If not, fall back to the structure in `references/task-template.md`.

The artifact MUST contain:
1. **Verbatim user prompt** — never edit
2. Resolved clarifications (from Phase 2)
3. High-level plan (3–7 bullets)
4. Technical plan (Phase 3 in full)
5. Phases table (Phase 4)
6. Risks / GOTCHA
7. Open questions (things deferred to v2)
8. Status header (`🟡 pending`)

After writing the artifact, also:
- Append a row to the project's task index (`wiki/tasks/index.md` or equivalent)
- If the project has a `hot.md`, add a one-line entry under "Current focus" — but only if the user signals they're going to start work soon.

## Output format conventions

- All technical writing: prose + tables. Avoid heavy bullet lists for explanation.
- Code/schema: fenced blocks with the right language (`prisma`, `typescript`, `sql`).
- Decisions: **bold-then-em-dash** style — `**Decision** — rationale`.
- Risks: warning emoji + one line each: `⚠️ <risk>`.
- Questions to user: ALWAYS via the structured-question tool when one is available; never free-text-buried.

## Quality bar

Before returning the plan, self-check:

- [ ] Every clarification answer changed something concrete in the plan
- [ ] Every new model has indexes that match its query patterns
- [ ] Every API endpoint has stated auth
- [ ] Every phase is independently reviewable
- [ ] Risks include at least one race / transaction concern (if writes are involved)
- [ ] Every non-trivial decision has a stated reason
- [ ] The artifact follows the project's existing template, not a generic one
- [ ] No phase is "implement everything" — that's not a phase
- [ ] If I matched a business-draft in Phase 1.5, I appended to the same file — did NOT create a new one — and changed status to `pending` + `ready_for_dev: true` AND moved the row in `wiki/tasks/index.md` from "Бизнес-черновики" to "Очередь"

If any check fails, revise before producing the final artifact.

## Anti-patterns to avoid

- **Asking 8 questions at once.** Max 4 per round, max 2 rounds.
- **Yes/no questions.** Always offer 2–4 concrete options.
- **"It depends" answers in decisions.** Pick one. The whole point of the skill is to resolve ambiguity, not relocate it.
- **Vague phases.** "Build the UI" is not a phase. "Public catalog page + filter widget + i18n keys" is.
- **Ignoring the project's GOTCHA list.** Re-deriving them is slower and worse than reading them.
- **Compressing two phases into one to "save time".** Sequential phases are easier to review and roll back.
- **Skipping the artifact step.** A plan that lives only in chat is lost in 24 hours.
- **Skipping Phase 1.5 (business-draft scan).** Even if you "feel sure" the user wants a fresh task — if they have business-drafts from a partner, you're at risk of creating a duplicate plan and ignoring the partner's framing. Always scan, even if it adds 30 seconds.
- **Re-asking business questions when matching a draft.** The partner already answered them. The draft IS the business spec. Re-asking insults the partner's work and burns the developer's time.
- **Creating a new task file when a business-draft matches.** Append to the existing file. The verbatim partner prompt + your technical plan in one artifact preserves history.

## Reference files

- `references/clarifying-questions-guide.md` — the four classes of questions every plan must cover, with ready-to-adapt templates.
- `references/architecture-mapping.md` — how to structure the layer-mapping table for FSD, hexagonal, clean, MVC, and plain Next.js projects.
- `references/task-template.md` — fallback artifact template when the project has no convention of its own.
- `references/example-trainers-task.md` — a real example: turning "интеграция функционала тренеров" into a complete plan, including all five phases. Read this to calibrate the level of detail expected.

## Example trigger phrases

These should reliably activate the skill:

- "Спланируй мне задачу — хочу добавить функционал X"
- "Распиши план интеграции Y"
- "Помоги задизайнить новую фичу Z от и до"
- "Let's spec out [feature]"
- "Draft an implementation plan for [feature]"
- "I want to add [feature], give me a phased plan"
- "Decompose this into a roadmap"
- "Architect [feature] for our project"

These should NOT activate it:

- "Fix the typo in line 42"
- "What does this function do?"
- "Run the tests"
- "Rename this variable"
- "Update the dependency"

If the request is borderline, default to running the **Phase 0 trigger check** explicitly and showing the user your decision.
