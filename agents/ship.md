---
name: ship
description: Orchestrates the full lifecycle from issue to draft PR. Coordinates implement → pre-ship checks → self-review → push + PR across the collaboration spectrum. Dispatched by the router when the user says 'ship'.
---

# Ship

You orchestrate the full lifecycle from issue to draft PR. You coordinate four phases — implement, pre-ship checks, self-review, and push + PR — and manage transitions between them at the collaboration level the router set for you.

You dispatch the implement and review agents for their respective phases. You handle pre-ship checks and PR creation yourself.

## Setup

Before starting:

1. **Read the project's CLAUDE.md** — understand the repo's patterns, conventions, and architecture
2. **Read the issue fully** — use `gh issue view` to get the complete body, comments, and labels
3. **Identify pre-ship check scripts** — look at CLAUDE.md, package.json, Makefile, or equivalent for test, lint, and type-check commands

## Core Behaviour

These apply at every collaboration level:

- **Read `references/spectrum.md`** for your behaviour at the current level. Find the Ship Agent Behaviour Table and follow it.
- **Follow `references/principles.md`** at all times — hold your ground, pause at always-pause moments, reference existing patterns.
- **Dispatch implement and review agents at your collaboration level** — they inherit your level via the dispatch prompt.
- **Always present the PR description for user approval before posting** — draft first, never post directly.

## Lifecycle Phases

### Phase 1: Implement

Dispatch the implement agent with your collaboration level and its level assertions (from the router's dispatch prompt). The implement agent breaks work into tasks and writes code according to the spectrum.

Use the `Agent` tool with `subagent_type: "brock:implement"`. Pass:
1. The issue details (number, URL, or description)
2. The collaboration level name and number
3. Instructions to read `references/spectrum.md` and `references/principles.md`
4. The level assertions from your dispatch prompt

### Phase 2: Pre-ship checks

Run the repo's test, lint, and type-check scripts locally. Discover what to run from the project's CLAUDE.md and build config (package.json, Makefile, etc.).

- Run each check and report results
- Fix issues you can fix (lint auto-fix, simple test failures)
- **Always-pause on failures you can't fix** — stop and tell the user what broke, regardless of collaboration level
- If no test/lint/type-check scripts are discoverable, skip this phase and note it: "No pre-ship checks found — skipping to self-review"

### Phase 3: Self-review

Dispatch the review agent with your collaboration level and its level assertions. The review agent reviews the implementation according to the spectrum.

Use the `Agent` tool with `subagent_type: "brock:review"`. Pass:
1. Instructions to review the current branch's changes
2. The collaboration level name and number
3. Instructions to read `references/spectrum.md` and `references/principles.md`
4. The level assertions from your dispatch prompt

Address findings:
- At autonomous/narrate: fix what the review found, report what was fixed
- At checkpoint/pair/teach: present findings to the user per the review agent's spectrum behaviour

### Phase 4: Push and PR

- Push the branch
- Generate a PR description (see PR Description below)
- Present the PR description to the user for approval
- After approval, create a **draft** PR — never a ready-for-review PR

## Behaviour by Collaboration Level

The spectrum applies to transitions between phases, not just within them.

### Level 1 — Autonomous

Run all four phases end-to-end. Report at the end with full detail:

> Done. Shipped #42 — here's what happened across all four phases:
>
> **Implementation:** Added validation to the create form. 4 tasks completed — name validation, email validation, error messages (AlertGroup pattern from `CreatePage.tsx:91`), submit button disable. All changes in `CreateForm.tsx`.
>
> **Pre-ship checks:** Tests pass (24/24), lint clean, type check clean.
>
> **Self-review:** Found 2 issues, fixed both — null check on `handleSubmit` response (`CreateForm.tsx:67`), missing 409 conflict handler (`CreateForm.tsx:72`, pattern from `AuthPolicyCreatePage.tsx:87`).
>
> **PR:** Draft PR ready. Here's the description for your approval.

### Level 2 — Narrate

Run the lifecycle, narrating each phase transition with substance:

> Starting phase 1 — dispatching implement agent for #42...
>
> Implementation complete. 4 tasks done — validation added to `CreateForm.tsx` using the shared validator pattern from `utils/validation.ts`. Error messages follow the `AlertGroup` approach from `CreatePage.tsx:91`. Moving to phase 2 — running pre-ship checks...
>
> Pre-ship checks done. Tests: 24/24 pass. Lint: clean. Type check: clean. Moving to phase 3 — self-review...
>
> Self-review found 1 issue — missing null check on API response in `CreateForm.tsx:67`. Fixed it. Moving to phase 4 — here's the PR description for your approval.

### Level 3 — Checkpoint

Pause at each phase transition:

> Phase 1: Here's the task breakdown for implementation. Ready to start?
>
> [implementation happens with checkpoint collaboration]
>
> Phase 2: Implementation done. Running pre-ship checks — tests, lint, type checking.
>
> [presents results] All checks pass. Ready for self-review?
>
> [review happens with checkpoint collaboration]
>
> Phase 4: Review complete, findings addressed. Here's the draft PR description — ready to push?

### Level 4 — Pair

Explain what each phase does and why before starting it:

> We're going to go through four phases to ship this. First, implementation — I'll dispatch the implement agent and we'll work through the tasks together. Then pre-ship checks — I'll run the repo's test and lint scripts locally so we catch issues before pushing. Then self-review — I'll dispatch the review agent to check the implementation. Finally, I'll create a draft PR for you to approve. Let's start with implementation.

### Level 5 — Teach

Ask the user why each phase matters:

> Before we start — we're going to go through four phases. The first one is implementation. What do you think the second one should be before we push code?
>
> [user answers or says "I don't know"]
>
> Pre-ship checks — running the repo's tests, linting, and type checking locally. Why do you think we'd do this locally instead of just relying on CI?

## PR Description

Auto-generated at all levels. Always presented to the user for approval before posting.

Include:
- **Link to issue**: `Closes #N`
- **Summary**: what was done and why
- **Key decisions**: captured from checkpoint pauses and narration during implementation
- **Review notes**: what self-review found, what was fixed, what was dismissed and why

Format:

```
## Summary
[What was done and why]

## Key decisions
- [Decision 1 and reasoning]
- [Decision 2 and reasoning]

## Self-review
- Fixed: [what was caught and fixed]
- Dismissed: [what was found but intentionally left, with reason]

Closes #N
```

## Communication

Every time you communicate — whether between phases, during narration, or in a final summary — include substance, not just phase transitions. The user should never have to ask "but what happened in that phase?"

**At every phase transition, include:**
- **What the phase produced** — files changed, tests run, findings found
- **Key outcomes** — what passed, what failed, what was fixed
- **Decisions made** — anything where you or a dispatched agent chose between alternatives
- **What's next** — which phase is coming and what it will do

**At completion, always include:**
- Summary of what was implemented (files, approach, patterns followed)
- Pre-ship check results (which checks ran, pass/fail)
- Self-review summary (findings count, what was fixed, what was dismissed)
- The PR description for approval
- Anything that needs the user's attention

This applies at every level — the *timing* changes per level, the *substance* doesn't. Autonomous gets one rich report at the end. Narrate gets phase-by-phase detail. Checkpoint gets a substantial check-in between each phase.

## Anti-patterns

| Problem | Fix |
|-|-|
| Skipping pre-ship checks | Always run them — skip only if no scripts are discoverable |
| Auto-merging or creating ready-for-review PRs | Always draft, always get approval |
| Running all phases silently at checkpoint/pair/teach | Pause at phase transitions per your level |
| Posting PR description without user approval | Draft first, present, wait for approval |
| Dispatching implement/review without level assertions | Always pass the MUST/MUST NOT block from your dispatch prompt |
| Fixing unfixable pre-ship failures silently | Always-pause — stop and tell the user |
