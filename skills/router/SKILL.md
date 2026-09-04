---
name: router
description: MUST be invoked when the user asks to implement, fix, ship, or work on a feature or issue, or to review a PR or code changes. Triggers on 'implement #N', 'fix this', 'ship this', 'work on', 'lets work on', 'review this PR', 'review #N', any GitHub issue or PR URL. Routes through brock's adaptive collaboration system which adapts style based on how involved the user wants to be.
---

# Brock Router

Route this request through Brock's shared router. The router classifies and dispatches;
it does not perform the implementation or review itself.

## Shared routing contract

1. Treat the user's complete message as the routing input. Do not require a host-
   specific prompt variable or rewrite the request before routing it.
2. Read and apply the classification and collaboration-level rules in
   [`BROCK.md`](../../BROCK.md).
3. Select the matching specialist guidance from `agents/`: implement, review, or ship.
   Preserve the parsed level and its assertions when handing off the work.
4. If the host provides delegation, invoke the matching specialist through the host's
   native mechanism. Pass the complete user request, classification, collaboration
   level, and required specialist references; the invocation syntax is host-defined.
5. If the host does not provide delegation, continue in the current conversation with
   the selected specialist guidance. Keep routing and specialist work logically
   separate, and never imply that an external specialist ran.

The router must be usable on every supported host. It must not depend on a particular
delegation command, agent parameter, environment variable, prompt format, or lifecycle
hook.

## Steps

1. Apply the shared routing contract above.
2. Pass the user's full message through unchanged when handing off or continuing inline.
3. Parse the collaboration level from natural language and preserve it for the selected
   specialist.
4. Return the routing result through the host's normal conversation path, accurately
   reflecting whether native delegation occurred or the work continued inline.
