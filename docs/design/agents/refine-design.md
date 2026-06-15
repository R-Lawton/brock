# Brock Agent Design: Refine

**Status**: Draft
**Parent spec**: [collaborative-ai-plugin-design.md](../collaborative-ai-plugin-design.md)

## Purpose

The refine agent takes vague or underspecified issues and produces clear, implementable specs with testable acceptance criteria. Refinement is naturally collaborative — it's always a conversation, even in autonomous mode.

## Responsibilities

1. **Identify gaps** — what's missing, ambiguous, or assumed in the issue?
2. **Ask clarifying questions** — one at a time, not a wall of questions
3. **Explore the codebase** — answer questions by reading the code when possible, rather than asking the user
4. **Produce a spec** — structured output with acceptance criteria, scope, and known unknowns
5. **Update the issue** — write the refined spec back to the issue (with user approval)

## Inputs

- Issue details (title, description, comments)
- Collaboration level (from router)
- Repo's CLAUDE.md (for understanding patterns, architecture, and conventions)
- Codebase access (for answering questions by reading code)

## Outputs

- Structured spec with:
  - Clear problem statement
  - Acceptance criteria (testable)
  - Scope (what's in, what's out)
  - Known unknowns
  - Suggested task breakdown
- Updated issue (with user approval)

## Core Approach

**Explore before asking.** Many questions can be answered by reading the codebase. If the issue says "add validation to the form" and the refine agent can find the form and see what validation already exists, it should do that rather than asking the user "which form?"

**One question at a time.** Don't present a wall of 10 questions. Ask the most important one, get the answer, then decide if the next question is still relevant.

**Recommend answers.** When asking a question, provide a recommendation based on codebase patterns: "I think this should follow the same validation pattern as `AuthPolicyCreatePage.tsx` — does that sound right?"

## Behaviour by Collaboration Level

### Level 1 — Autonomous

- Read the issue and codebase
- Identify gaps, answer what it can from the code
- Produce a draft spec with assumptions clearly marked
- Present the spec: "Here's what I think this issue means. I made assumptions on X and Y — are these right?"

### Level 2 — Narrate

- Same as autonomous but explains its reasoning as it goes
- "Reading the issue... the acceptance criteria mention 'validation' but don't specify which fields... checking the form... I see 5 fields, 3 already have validation..."

### Level 3 — Checkpoint

- Present findings from the issue analysis
- Ask clarifying questions one at a time at natural breakpoints
- Build the spec incrementally: "Here's what we have so far. Next question: should this validation be client-side only or also server-side?"

### Level 4 — Pair

- Walk through the issue together
- "Let's look at this together. The issue says X — what does that mean to you?"
- Explore the codebase together: "Let me show you what exists already, then we can figure out what's missing"

### Level 5 — Teach

- Guide the user through refinement thinking
- "What makes a good acceptance criterion? Let's take this one — is it testable as written?"
- "What would you check to scope this issue? Let's look at the codebase together."

## Spec Structure

```markdown
## Problem
What's being solved and why.

## Acceptance Criteria
- [ ] Testable criterion 1
- [ ] Testable criterion 2

## Scope
- In scope: ...
- Out of scope: ...

## Known Unknowns
- Things we've identified but can't resolve yet

## Suggested Task Breakdown
- Task 1: ...
- Task 2: ...
```

## Open design questions

- Should refine automatically update the GitHub/Jira issue, or always present the spec for approval first?
- How does refine handle issues that need input from someone other than the current user?
- Should the suggested task breakdown include recommended collaboration levels per task?