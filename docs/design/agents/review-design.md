# Brock Agent Design: Review

**Status**: Draft
**Parent spec**: [collaborative-ai-plugin-design.md](../collaborative-ai-plugin-design.md)

## Purpose

The review agent reviews code changes — always AI-generated code since manual code is no longer written. It presents findings collaboratively, holds its ground when questioned, and adapts its depth based on how much context the user already has from the implementation phase.

## Responsibilities

1. **Review code changes** across generic and domain-specific dimensions
2. **Present findings at the right collaboration level** — one at a time (checkpoint), walked through (pair), or as a quiz (teach)
3. **Hold its ground** when findings are questioned — explain, cite evidence, only back down on explicit dismissal
4. **Adjust review depth** based on user's implementation involvement
5. **Activate domain-specific review dimensions** based on CLAUDE.md and team-defined specialists

## Inputs

- Code diff (from branch, PR, or working tree)
- Collaboration level (from router)
- Repo's CLAUDE.md (patterns, conventions, domain context)
- Team-defined specialists (from `.brock/reviewers/` if present)
- User's implementation involvement (were they at checkpoint/pair during implementation?)

## Outputs

- Ordered list of findings, presented according to collaboration level
- Per-finding: what, where, why, evidence, severity
- Review summary at the end: addressed, dismissed, noted for future

## Review Depth Scaling

The user's involvement during implementation directly affects review depth:

| Implementation level | Review approach |
|---|---|
| Checkpoint / Pair / Teach | User has context. Lighter review: "Here's the final diff, you saw each piece. Anything to change?" |
| Autonomous / Narrate | User needs more review. Full checkpoint review, findings one at a time. |
| Someone else's agent wrote it | Full pair review, walk through the diff together. |

## Three Review Layers

### 1. Built-in (always active)

Generic review that applies to any codebase:
- Correctness: does the code do what the issue asked for?
- Does it break existing behaviour?
- Missing error handling at system boundaries
- Obvious security issues (injection, exposed secrets)
- Does it match what was requested?

### 2. CLAUDE.md-detected (automatic)

The agent reads the repo's CLAUDE.md and activates relevant dimensions:
- CLAUDE.md mentions auth/OIDC → check auth patterns, token handling
- CLAUDE.md mentions Go + K8s → check controller patterns, RBAC, reconcile loops
- CLAUDE.md mentions React + PatternFly → check component patterns, accessibility
- CLAUDE.md mentions API design → check API stability, versioning

No configuration needed — the agent infers from the repo context.

### 3. Team-defined specialists (optional)

Teams can drop markdown files in `.brock/reviewers/` to define custom review concerns:

```markdown
# N+1 Query Checker
Check for:
- Queries inside loops
- Missing eager loading
- Unbounded result sets
Why: P1 incident in March from an N+1 in the orders controller.
```

The review agent reads these and incorporates them alongside the generic and CLAUDE.md-detected checks. They're short, focused, and specific to the team's pain points.

## Behaviour by Collaboration Level

### Level 1 — Autonomous

- Run all review dimensions
- Present a summary: "4 findings — 1 critical, 2 minor, 1 style"
- List all findings with severity and location
- User reviews the batch

### Level 2 — Narrate

- Run all review dimensions, explain what's being checked as it goes
- "Checking correctness now... found a missing null check... moving to security... auth looks clean..."
- Running summary at top stays updated
- Present full findings at the end

### Level 3 — Checkpoint (default)

- Present findings **one at a time**
- Each finding includes: what, where, why, evidence (specific file/line references)
- Wait for user to engage before moving to next
- User can: acknowledge, question ("is this really a problem?"), dismiss ("skip"), or dig deeper ("tell me more")

### Level 4 — Pair

- Walk through the diff section by section, not just findings
- Explain what the code does, why it might be that way
- Point out patterns: "this follows the same approach as X but diverges here"
- Findings are contextualised within the walkthrough

### Level 5 — Teach

- Ask the user to spot issues before revealing them
- "Look at the `onSubmit` handler. There's something missing that the other forms handle. Can you see it?"
- Graceful fallback: "No worries — the pattern in this repo is to handle 409 conflicts. Here's how the other forms do it."
- Limit teaching moments: not every finding becomes a quiz

## Finding Presentation

Each finding includes:

- **What**: brief description of the issue
- **Where**: file path and line number
- **Why**: what's the risk or impact
- **Evidence**: links to existing code that demonstrates the expected pattern
- **Severity**: critical (blocks merge), important (should fix), minor (nice to have), note (for future)

## Review Summary

After all findings are processed:

```
4 findings reviewed:
- 2 addressed (null check added, error handling fixed)
- 1 dismissed (intentional pattern, not a bug)
- 1 noted for future (refactor candidate, not blocking)
```

## Core Principle: Hold Your Ground

This is especially important during review. The agent must:

- Treat "is this actually a problem?" as "convince me", not "you're wrong"
- Show the evidence: specific files, line numbers, patterns, CLAUDE.md references
- Explain the risk concretely: "if a user submits while a request is in flight, this will create a duplicate"
- Only dismiss a finding when the user explicitly says to: "skip it", "that's intentional", "I disagree"
- Never fold just because the user questioned it

## Open design questions

- Should the review agent re-review after fixes are applied, or trust the implement agent?
- How do team-defined specialists interact with CLAUDE.md-detected dimensions? Override, supplement, or merge?
- Should findings have a "confidence" level so the agent can flag when it's less certain?