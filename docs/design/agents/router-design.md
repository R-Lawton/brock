# Brock Agent Design: Router

**Status**: Draft
**Parent spec**: [collaborative-ai-plugin-design.md](../collaborative-ai-plugin-design.md)

## Purpose

The router is the entry point for all user requests. It classifies the task, sets the collaboration level, dispatches to the right agent, and maintains visibility across parallel work. It never writes code.

## Responsibilities

1. **Classify requests** — determine what kind of work is being asked for (implement, review, triage, refine, ship)
2. **Set collaboration level** — parse user cues and repo defaults to determine starting level
3. **Dispatch** — hand off to the right agent with the collaboration level as context
4. **Track parallel work** — maintain the summary dashboard when multiple issues are in flight
5. **Detect shifts** — watch for user cues that change the collaboration level mid-task and relay to the active agent

## Inputs

- User's natural language request
- Repo's CLAUDE.md (for `brock-default` setting and domain context)
- Current state of any in-flight issues

## Outputs

- Dispatches to specialist agents with collaboration level
- Summary dashboard for parallel work
- Shift instructions to active agents when user cues change

## Collaboration Level Management

### Setting the starting level

Priority order:
1. Explicit user signal in the request ("walk me through this", "just do it")
2. Repo default from CLAUDE.md (`brock-default: pair`)
3. Global default: **checkpoint (3)**

### Parsing user cues

| User signal | Starting level |
|---|---|
| No signal | Checkpoint (default) |
| "I know this area" / "just do it" | Autonomous (1) or Narrate (2) |
| "New to this" / "unfamiliar" / "first time" | Pair (4) |
| "Teach me" / "explain as you go" | Teach (5) |
| "Big PR, keep me involved" | Checkpoint (3) |

### Mid-task shifting

The router watches for cues and instructs the active agent to shift:

| User cue | Action |
|---|---|
| "Why?" / "Walk me through this" | Shift UP toward pair/teach |
| "I get it now, just do the rest" | Shift DOWN to autonomous |
| "Go ahead" / "Looks good, keep going" | Shift DOWN one level |
| "Just do this part" | Autonomous for this task only, then revert |

### What the router does NOT do

- **No git inference** — never uses git blame, commit history, or file authorship to guess context level. Git data is unreliable.
- **No code writing** — the router orchestrates, it does not implement.
- **No opinion on implementation approach** — that's the implement agent's job.

## Parallel Work Management

When multiple issues are in flight, the router maintains a summary dashboard:

```
#1 [checkpoint]  3/6  Waiting: "Add error messages — follow AlertGroup pattern?"
#2 [autonomous]  5/8  Working: Adding unit tests
#3 [narrate]     2/7  Working: Implementing webhook handler
```

Rules:
- **Checkpoint/pair/teach** — one active at a time, user is involved
- **Autonomous/narrate** — can run in background alongside an active collaborative issue
- **Always-pause moments** surface in the dashboard regardless of level

## Multi-issue dispatch

When the user requests multiple issues with different levels:

> "Ship #1, walk me through it. Also ship #2 and #3, I know those areas."

The router:
1. Starts #1 in checkpoint mode, presents task breakdown
2. Dispatches #2 and #3 to autonomous worktree workers in background
3. Maintains dashboard showing all three
4. Surfaces any always-pause moments from #2/#3 immediately

## Interactions with other agents

| Agent | Router's role |
|---|---|
| Implement | Dispatches with collaboration level, relays shift cues |
| Review | Dispatches with collaboration level, activates review dimensions based on CLAUDE.md |
| Triage | Dispatches, receives assessment |
| Refine | Dispatches, refine is naturally collaborative |
| Ship | Orchestrates the full lifecycle, coordinates implement → review → push → PR |

## Open design questions

- How does the summary dashboard render in Claude Code? Task system, inline text, or both?
- When shifting levels, does the router tell the agent directly or does it modify the agent's system context?
- How does the router handle conflicting signals? ("Just do it" but also "walk me through the auth part")