---
name: brock
description: Apply Brock's portable adaptive collaboration protocol to coding, review, and shipping requests.
---

Read `BROCK.md` at the repository root and follow it. It is the shared source of truth
for Brock's routing and collaboration behaviour. Use the specialist guidance in
`agents/` and `references/` when the request is an implementation, review, or shipping
task.

## Shared execution contract

1. Classify the request and parse its collaboration level using `BROCK.md`.
2. Select the matching specialist guidance: implement, review, or ship. Explain
   requests remain in the current conversation unless the host provides a more
   appropriate native response path.
3. If the host provides delegation, use its native delegation mechanism and pass the
   user's complete request, the classification, the collaboration level, and the
   relevant specialist instructions. The mechanism and agent names are host-defined.
4. If the host does not provide delegation, continue in the current conversation with
   the selected specialist guidance. Keep the routing decision and specialist work
   logically separate, and use a markdown checklist when the host has no task list.
5. Preserve Brock's collaboration level, always-pause rules, and verification
   requirements in either execution path.

This skill must remain host-neutral: it must not depend on a particular command,
environment variable, prompt format, lifecycle hook, or delegation syntax. Never claim
that delegation or a hook occurred unless the host actually performed it.
