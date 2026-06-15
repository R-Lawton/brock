# Brock Agent Design: Ship

**Status**: Draft
**Parent spec**: [collaborative-ai-plugin-design.md](../collaborative-ai-plugin-design.md)

## Purpose

The ship agent manages the full lifecycle from issue to merged PR. It coordinates the other agents (implement → review → push → PR) and manages the transitions between phases. It's the orchestrator for end-to-end delivery.

## Responsibilities

1. **Coordinate the lifecycle** — implement, review, push, create PR
2. **Manage phase transitions** — decide when to move from implementation to review to shipping
3. **Handle multi-issue dispatch** — parallel shipping with per-issue collaboration levels
4. **Pre-ship checks** — verify tests pass, linting clean, compilation succeeds before pushing
5. **PR creation** — create draft PR with context from the implementation journey

## Inputs

- Issue number(s) or description
- Collaboration level (from router, per-issue for multi-issue)
- Repo's CLAUDE.md

## Outputs

- Completed implementation (via implement agent)
- Review findings addressed (via review agent)
- PR created with description capturing decisions and context

## Lifecycle Phases

### Phase 1: Implement

- Dispatch to implement agent with the collaboration level
- The implement agent does its work according to the spectrum
- Ship agent monitors progress via task list

### Phase 2: Pre-ship checks

- Run tests, linting, type checking
- At checkpoint level: present results and wait
- At autonomous level: fix issues silently if possible, report if not

### Phase 3: Self-review

- Dispatch to review agent
- Review agent operates at the collaboration level
- Findings are addressed (implement agent fixes, or user fixes)

### Phase 4: Push and PR

- Push branch
- Create draft PR
- PR description includes:
  - Link to issue
  - Summary of what was done
  - Key decisions made during implementation (captured from narration/checkpoints)
  - Any dismissed review findings with reasons
- Present PR to user for approval before creating

## Behaviour by Collaboration Level

### Level 1 — Autonomous

- Run the full lifecycle end to end
- Report back with: PR link, summary of what was done, any decisions made, review findings
- User reviews the finished PR

### Level 2 — Narrate

- Run the lifecycle, narrating each phase transition
- "Implementation complete, running pre-ship checks... tests pass, linting clean... starting self-review..."
- Running summary captures the journey

### Level 3 — Checkpoint

- Pause at each phase transition:
  1. "Here's the task breakdown for implementation. Ready to start?"
  2. (Implementation happens with checkpoint collaboration)
  3. "Implementation done. Pre-ship checks: tests pass, linting clean. Ready for self-review?"
  4. (Review happens with checkpoint collaboration)
  5. "Review complete, findings addressed. Here's the PR description — ready to push?"

### Level 4 — Pair

- Same phase transitions but with more context at each
- Walk through the PR description together
- Discuss what to include, what to leave out

### Level 5 — Teach

- Explain the shipping process: "Before we push, let's check X and Y. Why do you think these checks matter?"
- Guide through PR description writing
- Explain CI expectations

## Multi-Issue Shipping

When the user requests multiple issues ("ship #1, #2, #3"):

1. Router sets per-issue collaboration levels
2. Ship agent dispatches implement agents — parallel for autonomous/narrate, sequential for checkpoint/pair/teach
3. Maintains summary dashboard across all issues
4. Each issue goes through the full lifecycle independently
5. Reports a summary table when all complete

## PR Description

The PR description is a key output — it captures the journey, not just the result. This is especially valuable because all code is AI-generated.

Includes:
- **Summary**: what was done and why
- **Key decisions**: captured from checkpoint pauses and narration
- **Approach**: what patterns were followed, what alternatives were considered
- **Review notes**: what was found, what was fixed, what was dismissed and why
- **Testing**: what was tested, what passed

The user always approves the PR description before it's posted.

## Open design questions

- Should ship support `--resume` for picking up mid-lifecycle? If so, how does it recover collaboration context?
- At what collaboration level should the PR description be collaboratively written vs auto-generated?
- Should ship auto-merge for personal repos, or always leave that to the user?
- How does ship handle CI failures after push? Different behaviour at different collaboration levels?