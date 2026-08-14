# Cost Efficiency

Shared reference for model selection and token efficiency. The router reads this when deciding which model to dispatch agents with. All agents follow the token efficiency guidance.

## Model and Effort Tiers

Two levers control cost: **model** (capability) and **effort** (reasoning depth). Default to the lowest combination that can do the job.

### Models

| Tier | Model | Cost | Use for |
|------|-------|------|---------|
| 1 | **haiku** | Lowest | Routing, classification, orchestration, coordination |
| 2 | **sonnet** | Mid | Implementation, code review, pre-ship checks, most coding tasks |
| 3 | **opus** | Highest | Complex architectural work, large-scale refactors, security-critical changes |

### Effort levels

| Level | When to use |
|-------|-------------|
| **low** | Pattern matching, classification, dispatch — no deep reasoning needed |
| **medium** | Coordination, phase tracking, simple decisions |
| **high** | Code writing, code review, analysis — the ceiling for most work |

Avoid `xhigh` and `max` — `high` on opus is already the most capable combination.

### Agent defaults

| Agent | Model | Effort | Rationale |
|-------|-------|--------|-----------|
| Router | haiku | low | Just parsing intent and dispatching |
| Ship | haiku | medium | Coordinates phases, dispatches other agents |
| Implement | sonnet | high | Code writing needs solid reasoning |
| Review | sonnet | high | Analysis needs careful thought |

## When to Escalate to Opus

Opus is justified when the task requires holding large context and making novel judgment calls. Specific signals:

- **Architectural changes** — new subsystems, restructuring how components connect, altering shared interfaces
- **Large-scale refactors or migrations** — version bumps across many files, framework upgrades, API migrations
- **Security-critical changes** — auth flows, credential handling, permission models
- **Unfamiliar territory with high stakes** — first implementation in a new codebase where getting patterns wrong is costly
- **Multi-system coordination** — changes that span multiple services or repos simultaneously

## When Sonnet Is Enough

Sonnet handles the vast majority of coding work. Use it when:

- Following existing patterns in the repo
- Standard feature implementation with clear requirements
- Code review (all sizes — sonnet reads diffs well)
- Bug fixes with clear reproduction steps
- Test writing
- Documentation changes
- Config changes
- Pre-ship checks (running commands, reporting results)

## When Haiku Is Enough

Haiku is for work that doesn't touch code:

- Classifying requests (implement vs review vs ship)
- Parsing collaboration level from natural language
- Dispatching agents with the right parameters
- Coordinating phase transitions (ship agent orchestration)
- Simple Q&A about the project

## Router Model Selection

The router includes a model recommendation in every dispatch. The decision tree:

```
Task arrives
├── Routing/classification only?
│   └── haiku (router itself)
├── Ship orchestration?
│   └── haiku (ship agent), but dispatch implement/review at their appropriate tier
├── Implementation or review?
│   ├── Signals complexity? (see "When to Escalate" above)
│   │   └── opus
│   └── Standard work
│       └── sonnet
└── Unsure?
    └── sonnet (safe default — capable enough for almost everything)
```

The router communicates its model decision in the dispatch prompt: `"Model: sonnet"` or `"Model: opus — [reason]"`. The coordinator uses this when calling the Agent tool.

## Token Efficiency

Reduce token waste without reducing quality.

### For all agents

- **Don't re-read files you've already read** — track what you've seen this session
- **Scope reads to what you need** — use line ranges for large files, don't read entire files when you need one function
- **Match communication to collaboration level** — autonomous gets a summary at the end, not running commentary. Checkpoint gets one task at a time, not all tasks at once.
- **Skip unnecessary reference reads** — if you already know your level's behaviour, don't re-read spectrum.md line by line

### For the router

- **Dispatch fast** — classify and dispatch in one turn when the path is clear
- **Don't read code** — you're routing, not implementing

### For implement/review

- **Use targeted searches** — grep for specific patterns rather than reading entire directories
- **Stop when done** — don't keep reviewing after all findings are presented, don't keep implementing after all tasks are complete
