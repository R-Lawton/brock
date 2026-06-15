# Brock Agent Design: Triage

**Status**: Draft
**Parent spec**: [collaborative-ai-plugin-design.md](../collaborative-ai-plugin-design.md)

## Purpose

The triage agent assesses issues for readiness. It determines whether an issue is clear enough to implement, needs refinement, should be split, or needs human input. It recommends a workflow path.

## Responsibilities

1. **Assess issue clarity** — are acceptance criteria clear and testable?
2. **Estimate scope** — is this one task or several? Should it be split?
3. **Check for dependencies** — does this depend on other issues or external work?
4. **Recommend workflow** — implement, refine, split, or flag for human review
5. **Label and prioritise** — suggest labels and priority based on content

## Inputs

- Issue details (title, description, comments, labels)
- Collaboration level (from router)
- Repo's CLAUDE.md (for understanding what kinds of issues this repo handles)

## Outputs

- Readiness assessment
- Recommended workflow path
- Suggested labels and priority
- If splitting: proposed sub-issues

## Behaviour by Collaboration Level

### Level 1 — Autonomous

- Assess the issue
- Return a verdict: "Ready to implement", "Needs refinement", "Should be split into X issues", "Needs human input because Y"
- Apply labels/priority

### Level 2 — Narrate

- Walk through the assessment: "Checking acceptance criteria... these are testable... checking scope... this touches auth and UI, might be worth splitting..."
- Return verdict with reasoning visible

### Level 3 — Checkpoint

- Present the assessment step by step:
  1. "Here's what the issue is asking for — does this match your understanding?"
  2. "I think this is ready to implement / needs refinement because..."
  3. "Recommended workflow: X. Sound right?"
- User can agree, redirect, or add context

### Level 4 — Pair

- Work through the assessment together
- "Let's look at the acceptance criteria. Are these testable? What would 'done' look like?"
- Discuss scope: "This touches three areas. Do you think that's one PR or should we split?"

### Level 5 — Teach

- Ask the user to assess first: "What do you think — is this issue ready to implement? What would you check?"
- Guide them through triage thinking: "Good point about scope. Another thing to check is whether there are dependencies..."

## Workflow Recommendations

| Assessment | Recommendation |
|---|---|
| Clear acceptance criteria, bounded scope, no blockers | → Implement |
| Vague requirements, missing acceptance criteria | → Refine (dispatch refine agent) |
| Too large, touches multiple areas | → Split into sub-issues |
| Needs external input, blocked on decision | → Flag for human review with specific questions |
| Duplicate or already addressed | → Close with reference |

## Open design questions

- Should triage automatically dispatch to refine if it recommends refinement, or just recommend and wait?
- How does triage handle issues from different sources (GitHub vs Jira) at different collaboration levels?
- Should triage suggest an estimated collaboration level for implementation? ("This is complex, recommend pair mode")