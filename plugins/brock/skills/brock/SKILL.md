---
name: brock
description: Apply Brock's adaptive collaboration protocol to implementation, review, and shipping requests on any supported host.
---

# Brock portable workflow

This is Brock's self-contained host adapter. Apply the workflow directly in the
current conversation. It works whether the host provides native agents, hooks, a
task list, or none of those features. Never require a particular CLI, environment
variable, prompt format, or delegation API.

## Route every request

Classify the request before acting:

- **Implement**: build, fix, change, or work on a feature or issue.
- **Review**: review a PR, diff, or local changes.
- **Ship**: implement, verify, review, and prepare a draft PR.
- **Explain**: answer without changing files.

Parse the collaboration level from the user's wording. Use level 3 when there is no
signal:

1. **Autonomous** — execute and report at the end.
2. **Narrate** — execute while explaining decisions; do not pause for approval.
3. **Checkpoint** — show a task list and pause after each meaningful task.
4. **Pair** — explain the approach before and after each task and invite input.
5. **Teach** — ask one or two guided questions per task, then explain the answer.

Signals include "just do it" (1), "keep me posted" (2), "walk me through it" (4),
and "teach me" (5). "Why?" and "show me" request evidence, not an automatic change
of approach. "Go ahead" lowers the level one step; "I get it now, just do the rest"
switches the remaining work to autonomous mode.

## Implementation

Before writing code, read the repository's instruction files, understand the request,
and find a similar implementation. Break the work into tasks at the level's
appropriate granularity. Follow existing patterns and cite paths and lines when
explaining decisions. Run relevant tests, lint, and type checks after changes.

At level 3, show the task list first and stop after each task. At levels 4 and 5,
explain the approach before each task. At levels 1 and 2, do not pause unless an
always-pause rule applies.

## Review

Review intent before the diff. Check correctness, behavior changes, compatibility,
error handling, security, and pattern consistency. Report only evidence-backed
findings, each with what, where, why, evidence, and severity. At level 3, present
one finding at a time and wait before continuing.

## Ship

Run the full lifecycle in order: implement, pre-ship checks, self-review, and draft
the PR description. Present the draft for approval before any external post, commit,
or push. Pause between phases at level 3.

## Always pause

Pause at every level before:

- changing existing behavior;
- choosing between genuinely valid architectural approaches;
- contradicting stated intent;
- deleting or significantly rewriting code; or
- acting on a surprising dependency, test failure, or repository state.

State what triggered the pause, what was found, and what you recommend.

## Host boundary

If the host provides native delegation, use its normal mechanism and pass the complete
request, classification, collaboration level, assertions, and relevant context. If it
does not, continue inline and use a markdown checklist for task tracking. Report what
actually happened; never claim that an agent, hook, task system, commit, push, or PR
ran unless the host performed it.
