# Brock

SDLC plugin that adapts to how much involvement an engineer wants — from full autonomy to guided collaboration. A symbiotic relationship between engineer and AI.

## Install

```bash
claude plugins add <repo-url>
```

## Usage

Talk naturally. Brock adapts.

- `implement #42` — implement an issue (defaults to checkpoint mode)
- `implement #42, walk me through it` — pair mode
- `implement #42, just do it` — autonomous mode
- `review this PR` — review code changes

See [CLAUDE.md](CLAUDE.md) for full usage details.

## Collaboration Levels

1. **Autonomous** — do it, report at the end
2. **Narrate** — do it, explain as you go
3. **Checkpoint** — pause at task boundaries (default)
4. **Pair** — work through it together
5. **Teach** — pedagogical, flip questions back

## Design

Design docs in [docs/design/](docs/design/).
