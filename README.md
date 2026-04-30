# claude-skills-marketplace

A personal Claude Code / Cowork plugin marketplace by Gleb. Currently ships **two coupled skills** that together cover the full lifecycle of a feature: from a non-technical partner's idea to a complete technical implementation plan.

## Install

```bash
/plugin marketplace add LumbagoG/claude-skills-marketplace
/plugin install task-planner@claude-skills-marketplace
/plugin install task-business-planner@claude-skills-marketplace
```

## Plugins

### task-business-planner

For the **non-technical product partner** who doesn't write code. Activates session-wide on persona-declaration phrases like "Я представляю бизнес". Asks 3–5 business-only clarifying questions (who / why / when / how does it look), produces an artifact in `wiki/tasks/` marked as `business-draft` — explicitly NOT ready for development.

No code, no schema, no jargon — pure product framing.

[See task-business-planner →](plugins/task-business-planner/README.md)

### task-planner

For the **developer**. Turns a vague feature request into a rigorous, FSD-aware implementation plan: clarifying questions → schema (verbatim) → API contracts → layer-mapping table → phased roadmap → risks/GOTCHA → wiki-task artifact.

**Auto-detects business-drafts** produced by `task-business-planner` and appends technical refinement to the same file (no duplicate artifacts, no re-asking the partner's already-answered questions).

[See task-planner →](plugins/task-planner/README.md)

## How they work together

```
┌─────────────────────────────────────────────────────────────────────┐
│  Partner (non-technical)                                            │
│  "Я представляю бизнес. Хочу чтобы тренеры могли..."                │
│         │                                                            │
│         ▼                                                            │
│  task-business-planner                                              │
│   • Reads project context (hot.md, index.md, domain glossary)       │
│   • Asks 3-5 product questions (no tech)                            │
│   • Writes wiki/tasks/002.md with status: business-draft            │
│   • Adds row to wiki/tasks/index.md → "🔮 Бизнес-черновики"         │
└─────────────────────────────────────────────────────────────────────┘

                              [days/weeks pass]

┌─────────────────────────────────────────────────────────────────────┐
│  Developer                                                          │
│  "Распиши техплан задачи 002" OR "Хочу добавить рассылки тренеров"  │
│         │                                                            │
│         ▼                                                            │
│  task-planner                                                       │
│   • Phase 1.5: scans index.md → finds matching business-draft       │
│   • Confirms with developer ("Беру черновик 002 в работу?")         │
│   • Skips business clarification (already answered)                 │
│   • Asks ONLY technical questions (modeling/cardinality/lifecycle)  │
│   • Appends schema + API + FSD table + phases to SAME file 002.md   │
│   • Changes status: business-draft → pending, ready_for_dev: true   │
│   • Moves row index.md → "🟡 Очередь" (ready for implementation)    │
└─────────────────────────────────────────────────────────────────────┘
```

The result is a single artifact that contains:
- The verbatim partner prompt
- The product framing (persona, JTBD, metrics, scenarios, UX)
- The technical plan (schema, API, FSD, phases, risks)
- The implementation result (commits, tests, links)

This is the same artifact pattern as a Spec-Driven-Development workflow, but split across two human roles cleanly.

## Repository layout

```
.
├── .claude-plugin/
│   └── marketplace.json               # marketplace manifest (lists both plugins)
├── plugins/
│   ├── task-planner/                  # developer skill
│   │   ├── .claude-plugin/plugin.json
│   │   ├── skills/task-planner/
│   │   │   ├── SKILL.md
│   │   │   └── references/
│   │   │       ├── clarifying-questions-guide.md
│   │   │       ├── architecture-mapping.md
│   │   │       ├── task-template.md
│   │   │       └── example-trainers-task.md
│   │   └── README.md
│   └── task-business-planner/         # non-technical partner skill
│       ├── .claude-plugin/plugin.json
│       ├── skills/task-business-planner/
│       │   ├── SKILL.md
│       │   └── references/
│       │       ├── business-clarifying-questions.md
│       │       ├── business-artifact-template.md
│       │       ├── elo-tt-domain.md
│       │       └── example-business-task.md
│       └── README.md
├── README.md
├── LICENSE
└── .gitignore
```

## Project context

Both skills are written for the **elo-tt** project (table tennis platform: clubs, tournaments, ELO, bookings, forum, chat, trainers in roadmap). The skills use project-specific vocabulary in their reference files. If you fork for another project, replace `elo-tt-domain.md` and `example-*.md` with your own equivalents.

## Contributing

Personal marketplace, but PRs welcome if you spot a bug. Plugin structure follows the [official Claude Code plugin spec](https://code.claude.com/docs/en/plugin-marketplaces).

## License

MIT — see [LICENSE](LICENSE).
