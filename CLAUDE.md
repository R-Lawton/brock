# Brock

Adaptive SDLC plugin for Claude Code. Adapts collaboration style from full autonomy to guided teaching based on how involved you want to be.

## How to use

Talk naturally. The router agent is the entry point for all tasks.

- "implement #42" — implement an issue (defaults to checkpoint mode)
- "implement #42, walk me through it" — pair mode
- "implement #42, just do it" — autonomous mode
- "review this PR" — review code changes
- "ship #42" — full lifecycle: implement, check, review, PR (defaults to checkpoint mode)
- "ship #42, just do it" — autonomous ship

## Collaboration levels

Brock adapts to how much involvement you want:

1. **Autonomous** — do it, report at the end
2. **Narrate** — do it, explain as you go
3. **Checkpoint** — pause at task boundaries (default)
4. **Pair** — work through it together, explain concepts
5. **Teach** — pedagogical, flip questions back

You don't pick a level from a menu. Just talk naturally — "I know this area", "teach me this pattern", "just do it" — and Brock adapts.

## Mid-task shifting

Change your mind at any time:
- "I get it now, just do the rest" — shifts to autonomous
- "Why?" or "Walk me through this" — shifts toward pair/teach
- "Go ahead" — shifts down one level

## Repo defaults

Teams can set a default collaboration level in their project's CLAUDE.md:

    brock-default: pair

Individual users can always override per-task via natural language.

## Architecture

- `agents/` — router, implement, review, ship agent definitions
- `references/` — shared spectrum and principles docs that agents reference
- `docs/design/` — design rationale and decisions
