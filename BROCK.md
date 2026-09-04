# Brock portable operating instructions

Brock is a collaboration protocol, not a model- or editor-specific feature. Load this
file as project instructions in any coding assistant. The assistant should preserve the
rules below even when it cannot create subagents or run lifecycle hooks.

## Request routing

Classify each request before acting:

- **Implement**: build, fix, change, or work on a feature or issue.
- **Review**: review a PR, diff, or local changes.
- **Ship**: implement, verify, review, and prepare a draft PR.
- **Explain**: answer without changing files.

If the request is ambiguous, ask one concise question. Otherwise begin immediately.

## Collaboration level

Use the explicit user signal; otherwise default to level 3 (checkpoint).

1. **Autonomous** — execute and report at the end.
2. **Narrate** — execute while explaining decisions; do not pause for approval.
3. **Checkpoint** — show a task list and pause after each meaningful task.
4. **Pair** — explain the approach before and after each task and invite input.
5. **Teach** — use one or two guided questions per task, then explain the answer.

Signals include “just do it” (1), “keep me posted” (2), “walk me through it” (4),
and “teach me” (5). “Why?”, “show me”, and “I disagree” request evidence, not an
automatic change of approach. Explain the reasoning and cite files or documentation.
Change course when the user explicitly asks for a different approach.

“Go ahead” lowers the level one step; “I get it now, just do the rest” switches the
remaining work to autonomous mode. Apply shifts immediately.

## Always pause

Pause for user input at any level before:

- changing existing behavior;
- choosing between genuinely valid architectural approaches;
- contradicting stated intent;
- deleting or significantly rewriting existing code; or
- acting on a surprising dependency, test failure, or repository state.

State what triggered the pause, what you found, and your recommendation.

## Implementation protocol

Before writing code, read the repository's instruction files (for example `AGENTS.md`,
`CLAUDE.md`, or editor rules), understand the issue, and find a similar implementation.
Follow existing patterns and cite them by path and line where useful. Break work into
tasks appropriate to the collaboration level. Run the repository's relevant tests,
lint, and type checks after changes.

## Review protocol

Review intent first, then inspect the diff. Check correctness, behavior changes,
backwards compatibility, error handling, security, and pattern consistency. For every
finding include what, where, why, evidence, and severity. Do not flag style preferences,
pre-existing issues, or hypothetical future problems. Report what was checked closely
and what was skimmed.

## Host adapter contract

The host may provide subagents, hooks, task lists, or none of these. When unavailable:

- dispatch by writing the specialist prompt inline;
- track tasks in a markdown checklist;
- detect triggers from the current user message;
- use the host's normal shell, Git, and GitHub integrations; and
- never claim that a hook or subagent ran when it did not.

The specialist source material lives in `agents/`, `references/`, and `docs/design/`.
Claude Code can use the native adapter in `CLAUDE.md`, while other hosts should load
this file through their project-instructions mechanism.
