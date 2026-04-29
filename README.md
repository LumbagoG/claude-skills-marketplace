# claude-skills-marketplace

A personal Claude Code / Cowork plugin marketplace by Gleb. Currently ships one plugin: **task-planner**.

## Install

```bash
/plugin marketplace add LumbagoG/claude-skills-marketplace
/plugin install task-planner@claude-skills-marketplace
```

## Plugins

### task-planner

Turn vague feature requests into rigorous, FSD-aware implementation plans.

**Triggers on phrases like:**
- "Спланируй мне задачу — хочу добавить функционал X"
- "Распиши план интеграции Y"
- "Let's spec out [feature]"
- "Draft an implementation plan for [feature]"

**The five-phase workflow:**
1. **Discover** — silent scan of project context (`wiki/_meta/hot.md`, schema, similar artifacts).
2. **Clarify** — 2–4 questions per round, max 2 rounds. Covers modeling, cardinality, integration depth, lifecycle.
3. **Design** — schema (verbatim), API contracts, layer-mapping table (FSD-aware), architectural decisions, risks/GOTCHA.
4. **Phase** — 8–14 numbered phases with owner agent and status, ending in QA gate.
5. **Document** — produces a wiki-task artifact following the project's existing template.

See [`plugins/task-planner/README.md`](plugins/task-planner/README.md) for full details and [`plugins/task-planner/skills/task-planner/references/example-trainers-task.md`](plugins/task-planner/skills/task-planner/references/example-trainers-task.md) for a full real-world example.

## Repository layout

```
.
├── .claude-plugin/
│   └── marketplace.json          # marketplace manifest
├── plugins/
│   └── task-planner/
│       ├── .claude-plugin/
│       │   └── plugin.json       # plugin manifest
│       ├── skills/
│       │   └── task-planner/
│       │       ├── SKILL.md
│       │       └── references/
│       │           ├── clarifying-questions-guide.md
│       │           ├── architecture-mapping.md
│       │           ├── task-template.md
│       │           └── example-trainers-task.md
│       └── README.md
├── README.md
├── LICENSE
└── .gitignore
```

## Contributing

This is a personal marketplace, but plugins follow the [official Claude Code plugin structure](https://code.claude.com/docs/en/plugin-marketplaces). PRs welcome if you find a bug or want to suggest an improvement.

## License

MIT — see [LICENSE](LICENSE).
