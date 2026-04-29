# task-planner

A Claude Code skill that turns vague feature requests into rigorous, implementation-ready plans.

## What it does

Given a fuzzy "I want feature X" prompt, this skill enforces a **five-phase workflow** that produces:

1. A **clarification dialogue** — 2–4 specific questions per round, recommendations marked, max 2 rounds.
2. A **technical plan** — schema (verbatim), API contracts, FSD layer-mapping table, architectural decisions with rationale, risks/GOTCHA.
3. A **phased roadmap** — 8–14 numbered phases each with an owner and a clear deliverable.
4. A **wiki-task artifact** — a versioned Markdown file in the project's task convention, including the verbatim user prompt and resolved clarifications.

The skill is opinionated: it forces a slow start (clarify before drafting), surfaces hidden invariants (the user's stated requirements often contradict the choices they make in Q&A), and writes nothing to the codebase — only the plan artifact.

## When it triggers

Trigger phrases (Russian and English):

- "Спланируй мне задачу — хочу добавить функционал X"
- "Распиши план интеграции Y"
- "Помоги задизайнить новую фичу Z от и до"
- "Let's spec out [feature]"
- "Draft an implementation plan for [feature]"
- "I want to add [feature], give me a phased plan"

It explicitly **does not** trigger on:

- Single-file mechanical edits
- Quick factual questions
- Requests where the user already wrote a spec and just wants code

## Why it exists

Without structure, Claude tends to:

1. Start coding immediately on the most visible part of a request.
2. Miss cross-cutting concerns (auth, realtime, email, i18n, accessibility).
3. Produce plans that don't fit the project's existing patterns (FSD layering, naming conventions, glass-style cards, etc.).

The result is rework. This skill is the slow-start cure.

## Architecture support

The skill includes mapping templates for these architectures:

- **Feature-Sliced Design (FSD)** — the original use case (elo-tt project).
- **Hexagonal / Ports-and-Adapters**
- **Clean Architecture**
- **MVC / "Plain Rails-style"**
- **Plain Next.js (App Router, no FSD)**
- Generic fallback for custom architectures.

See `skills/task-planner/references/architecture-mapping.md` for the full table set.

## Files

```
skills/task-planner/
├── SKILL.md                              # main skill
└── references/
    ├── clarifying-questions-guide.md     # the 4 question classes + templates
    ├── architecture-mapping.md           # layer tables per architecture
    ├── task-template.md                  # fallback artifact template
    └── example-trainers-task.md          # real example: trainers integration in a TT platform
```

## Example output

See [`skills/task-planner/references/example-trainers-task.md`](skills/task-planner/references/example-trainers-task.md) for a full real-world run: turning "интегрировать функционал тренеров" into a 5-model schema, 13-phase roadmap, 12-risk list, and a wiki artifact.

## Author

Gleb · `gleblubiskin.cursor@gmail.com`

## License

MIT.
