---
name: router
description: MUST be invoked when the user asks to implement, fix, ship, or work on a feature or issue, or to review a PR or code changes. Triggers on 'implement #N', 'fix this', 'ship this', 'work on', 'lets work on', 'review this PR', 'review #N', any GitHub issue or PR URL. Routes through brock's adaptive collaboration system which adapts style based on how involved the user wants to be.
---

# Brock Router

Route this request through the brock router agent. Do not implement, review, read code, or do any analysis yourself.

## Steps

1. Dispatch an agent with `subagent_type: "brock:router"`
2. Pass the user's full message as the prompt — do not summarise or rewrite it
3. The router parses the collaboration level from natural language and dispatches the right specialist
4. Relay the router's output back to the user exactly as received
