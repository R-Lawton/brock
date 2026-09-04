# Brock

SDLC plugin that adapts to how much involvement an engineer wants — from full autonomy to guided collaboration. A symbiotic relationship between engineer and AI.

## Install in Claude Code

```bash
claude plugins add <repo-url>
```

Claude Code gets the native router, agents, and prompt hook. The hook is optional: the
plugin still works when hooks are disabled, but the assistant must load `BROCK.md`.

## Use with other coding assistants

There is no universal plugin API across coding assistants. The portable part of Brock is
[`BROCK.md`](BROCK.md), which contains the routing, collaboration levels, pause rules,
implementation protocol, and review protocol. Add that file to the assistant's project
instructions, rules, or system prompt:

- **Codex**: install the plugin or add `BROCK.md` to the repository instructions.
- **Cursor, Windsurf, Cline, Continue, and similar tools**: copy or reference `BROCK.md`
  from their project-rules/instructions configuration.
- **Any model or custom agent**: include `BROCK.md` in the system/developer prompt and
  expose `agents/` and `references/` as readable context.

The host-independent behavior works everywhere. Native subagent dispatch, hooks, task
lists, and slash commands remain host-specific and are emulated inline when unavailable.

## Install in Codex

From a clone of this repository, configure the repository marketplace and install Brock:

```bash
codex plugin marketplace add .
codex plugin add brock@brock-codex
```

The canonical Codex marketplace is [.agents/plugins/marketplace.json](.agents/plugins/marketplace.json),
and the packaged plugin lives in [plugins/brock](plugins/brock). After changing the local
Codex plugin, refresh its cache before testing:

```bash
python3 <codex-home>/skills/.system/plugin-creator/scripts/update_plugin_cachebuster.py plugins/brock
codex plugin add brock@brock-codex
```

Start a new thread after reinstalling so the host reloads the updated skills. The
available Codex models and their intended use are listed in [docs/codex-models.md](docs/codex-models.md).

## Usage

Talk naturally. Brock adapts.

- `implement #42` — implement an issue (defaults to checkpoint mode)
- `implement #42, walk me through it` — pair mode
- `implement #42, just do it` — autonomous mode
- `review this PR` — review code changes
- `ship #42` — full lifecycle: implement, check, review, PR (defaults to checkpoint mode)
- `ship #42, just do it` — autonomous ship

See [BROCK.md](BROCK.md) for portable usage details and [CLAUDE.md](CLAUDE.md) for
Claude Code integration details.

## Collaboration Levels

1. **Autonomous** — do it, report at the end
2. **Narrate** — do it, explain as you go
3. **Checkpoint** — pause at task boundaries (default)
4. **Pair** — work through it together, explain concepts
5. **Teach** — pedagogical, flip questions back

## Design

Design docs in [docs/design/](docs/design/).
