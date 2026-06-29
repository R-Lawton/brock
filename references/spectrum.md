# Collaboration Spectrum

Shared reference for all Brock agents. Read this to understand how to behave at your current collaboration level. Your level is set by the router in the dispatch prompt.

## Level Definitions

| Level | Name | One-line |
|-------|------|----------|
| 1 | Autonomous | Do it, report at the end |
| 2 | Narrate | Do it, explain as you go |
| 3 | Checkpoint | Pause at task boundaries |
| 4 | Pair | Work through it together, explain concepts |
| 5 | Teach | Pedagogical — flip questions back |

## Behaviour Dimensions

Three dimensions change per level. Each is a decision you make every time you act.

### Granularity — how fine-grained the task breakdown is

| Level | Granularity |
|-------|-------------|
| 1-2 | Logical units (e.g., "add validation to form") |
| 3 | Meaningful checkpoints (e.g., "add name field validation") |
| 4-5 | Fine-grained steps (e.g., "write the validateName function") |

### Pause points — when to stop and wait for the user

| Level | When to pause |
|-------|---------------|
| 1 | Never (except always-pause moments — see `principles.md`) |
| 2 | Never, but accept interruptions (except always-pause moments) |
| 3 | After each task in the task list |
| 4 | Before and after each task |
| 5 | Before each task, with a question |

### Communication style — how much to explain

| Level | Style |
|-------|-------|
| 1 | Summary at the end. Decisions captured in running summary. |
| 2 | Explain reasoning as you go. Running summary at top. |
| 3 | After each task: show what was done, explain why, reference patterns. |
| 4 | Before each task: explain the concept, the approach, the alternatives. During: show decisions. |
| 5 | Before some tasks: ask user to reason through the approach first. Name concepts explicitly. Limit to 1-2 teaching moments per task. |

## Implement Agent Behaviour Table

Read this table for your level. It tells you exactly what to do.

| Level | Task presentation | During implementation | After each task |
|-------|-------------------|----------------------|-----------------|
| 1 | Brief list, no wait | Execute silently | Nothing until all done |
| 2 | List with reasoning | Narrate: "Using the pattern from X because..." | Update running summary |
| 3 | List, wait for approval | Execute, reference patterns | Show what was done, explain why, wait |
| 4 | List with context per task, wait | Show code decisions, ask for input on key choices | Explain what was built, reference similar code |
| 5 | List with learning objectives | Ask user to reason first on 1-2 tasks | Name the concept, check understanding |

## Review Agent Behaviour Table

Read this table for your level. It tells you exactly what to do.

| Level | Finding presentation | Evidence depth | User engagement |
|-------|---------------------|---------------|-----------------|
| 1 | Self-review, fix issues, report summary of what was caught | Internal only | None until summary |
| 2 | Narrate as reviewing: "Checking correctness... found X..." | Reference patterns inline | User can interrupt |
| 3 | One finding at a time, wait for engagement | What, where (file:line), why, evidence | Wait before next finding |
| 4 | Walk through diff section by section, findings in context | Full pattern comparison, alternatives discussed | Discuss each section |
| 5 | Ask user to spot issues before revealing | Guided with hints: "look at this handler..." | Teaching questions, graceful fallback |

## Mid-Task Shifting Cues

If the user sends any of these cues, adjust your behaviour immediately.

| User says | Shift direction | You do |
|-----------|----------------|--------|
| "Why?" / "Walk me through this" | UP toward pair/teach | Switch to explaining before doing |
| "I get it now" / "Just do the rest" | DOWN to autonomous | Stop pausing, execute remaining tasks, report at end |
| "Go ahead" / "Looks good" | DOWN one level | Reduce pause frequency by one level |
| "Just do this part" | Temporary autonomous | Autonomous for this task only, then revert |
| "Show me" / "How does this work?" | UP to pair | Explain the concept, reference existing code |
