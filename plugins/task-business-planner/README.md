# task-business-planner

A Claude Code / Cowork skill for **non-technical product partners** to plan features without seeing code, schema, or engineering jargon. Pairs with [`task-planner`](../task-planner/) which auto-detects the business-drafts this skill produces and refines them into technical plans.

## Why this skill exists

When the technical co-founder (Gleb) is on vacation but the non-technical partner has a feature idea, what happens? Either:

1. The partner waits — bad for momentum.
2. The partner writes the spec themselves — bad because it ends up in tech terms ("API endpoint for...") which constrains the developer or, worse, locks in wrong assumptions.

This skill is option 3: the partner talks to Claude in plain Russian (or English), Claude asks 4 business-only clarifying questions (who / why / when / how does it look), and produces a structured artifact in `wiki/tasks/` marked as `business-draft` — explicitly NOT ready for development.

When the developer comes back, they don't re-ask business questions — they say "распиши техплан задачи 002" or just start working on a related feature, and `task-planner` automatically detects the matching draft and appends a technical plan to the same file.

## When it triggers

Persona-declaration phrases (Russian and English):

- "Я представляю бизнес" / "я бизнес-сторона"
- "Я не разработчик" / "я нетехнический"
- "Я партнёр" / "я co-founder" / "я продакт"
- "I'm the business" / "I'm not technical" / "I'm the product side"

Once activated, the skill **persists for the entire session** until the user explicitly says "я разработчик" / "switch to dev mode".

## What it produces

A `wiki/tasks/YYYY-MM-DD-NNN-name.md` artifact with:

- Verbatim partner prompt (never edited)
- Persona + JTBD
- Success metrics with concrete numbers
- 2–3 user-story scenarios
- UX described in words ("на странице появляется блок с заголовком...")
- Open business questions with temporary defaults
- Priority
- **Empty technical-plan section** reserved for the developer
- Status: `🔮 business-draft`, `ready_for_dev: false`

Plus an entry in `wiki/tasks/index.md` under a new section "🔮 Бизнес-черновики" — visually separated from the regular work queue.

## What it deliberately does NOT do

- ❌ Discuss schema, API, FSD, code, frameworks, libraries
- ❌ Estimate effort or timeline
- ❌ Choose technology
- ❌ Worry about implementation feasibility ("this might be hard")
- ❌ Write code or pseudocode

These belong to the developer, not the partner.

## How the handoff works

```
[partner session]
"Я представляю бизнес. Хочу..."
    ↓
task-business-planner activates session-wide
    ↓
Asks 3-5 business questions (who/why/when/how-it-looks)
    ↓
Creates wiki/tasks/002.md with status: business-draft
    ↓
Adds row to wiki/tasks/index.md → "🔮 Бизнес-черновики"
    ↓
[partner session ends]

[developer session, days/weeks later]
"Распиши техплан задачи 002" OR
"Хочу добавить рассылки тренеров" (semantic match)
    ↓
task-planner Phase 1.5 scans index.md
    ↓
Finds business-draft 002, confirms with developer
    ↓
Reads draft → skips business clarification → asks technical questions
    ↓
Appends schema + API + FSD-table + phases to SAME file
    ↓
Changes status: business-draft → pending, ready_for_dev: true
    ↓
Moves row from "🔮 Бизнес-черновики" to "🟡 Очередь" in index.md
    ↓
[ready for implementation]
```

## Example output

See [`skills/task-business-planner/references/example-business-task.md`](skills/task-business-planner/references/example-business-task.md) — full real-world example: partner asked for "групповые рассылки тренеров", got a complete business-draft including persona, metrics, three user scenarios, UX described in words, open questions with defaults, and a flagged dependency on the trainers feature being live first.

## Files

```
skills/task-business-planner/
├── SKILL.md                                    # main skill
└── references/
    ├── business-clarifying-questions.md        # 4 question classes + templates
    ├── business-artifact-template.md           # full template for the wiki/tasks/ artifact
    ├── elo-tt-domain.md                        # project glossary in plain Russian
    └── example-business-task.md                # real example: trainer bulk messages
```

## Project scope

This skill is **elo-tt-aware**: it knows the project's personas (player / club / trainer / admin / guest), the existing entities (clubs, tournaments, ELO, bookings, forum, chat), and what's already shipped vs. planned. The reference file `elo-tt-domain.md` is the partner's mental model in plain Russian — no Prisma, no FSD, no code.

If you fork this skill for another project, replace `elo-tt-domain.md` with your own glossary.

## Author

Gleb · `gleblubiskin.cursor@gmail.com`

## License

MIT.
