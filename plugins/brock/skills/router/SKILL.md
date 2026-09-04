---
name: router
description: Route implementation, review, and shipping requests through Brock's host-neutral collaboration workflow.
---

# Brock router

Route the user's complete request before implementation or review. This router is
host-neutral: it must not depend on a particular CLI, command, environment variable,
prompt format, lifecycle hook, or delegation parameter.

1. Classify the request as implement, review, ship, or explain.
2. Parse the collaboration level from the user's wording; default to checkpoint.
3. Preserve the complete request, classification, level, and level assertions.
4. If the host provides native delegation, hand off through that host mechanism.
5. Otherwise continue inline with the matching workflow in the `brock` skill.

The router does not implement or review work itself. Do not claim native delegation
occurred when the host continued inline.
