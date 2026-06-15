# Brock — Collaborative AI Pair Programming Plugin

**Date**: 2026-06-12
**Status**: Draft — brainstorming phase

## Problem Statement

AI coding tools create a knowledge gap. Code gets written but the human doesn't build the understanding needed to own it, debug it, extend it, or defend it in review. This is true for:

- Juniors learning a codebase, language, or domain
- Seniors joining a new team or working in unfamiliar areas
- Anyone onboarding to a new repo or tech stack

**Everyone is a junior somewhere — it's about context, not title.**

Autonomous AI tools optimise for throughput, but the review at the end is actually *harder* because you're reading code with zero context about the decisions behind it. You're doing the same cognitive work but compressed into the review phase instead of spread across the process.

## Philosophy

**The agent is a senior colleague who pair programs with you, not a contractor who delivers finished work.**

A senior colleague would:
- Explain *why* before *what*
- Point you at existing patterns: "look at how `CreatePage.tsx` does this, we'll follow the same approach"
- Name the concepts: "this is a reconcile loop pattern, here's why it works this way"
- Let you do some of the work: "try writing the validation function, I'll review it"
- Not just give answers: "what do *you* think should happen when the request fails?"

## Core Principle: Hold Your Ground

**Questions are requests for more information, not disagreement.**

AI tools have a tendency to fold the moment you question them. You say "is this actually a problem?" and the AI hears "you're wrong" and immediately backs down. In reality, you're asking "convince me."

This plugin's agents must:

- **Default to holding their ground** when questioned
- **Explain reasoning fully** — where they found the issue, what the risk is, show the evidence
- **Cite sources** — "this contradicts the pattern in `AuthPolicyCreatePage.tsx:87`" or "this violates the error handling convention used in the other 6 forms in this repo"
- **Only back down when explicitly told** — not when questioned

The distinction:

| What the user says | What they mean | Agent should |
|---|---|---|
| "Why did you flag this?" | Explain more | Make the case, show evidence |
| "Show me where" | Prove it | Point to specific code, patterns, docs |
| "Is this really a problem?" | Convince me | Explain the risk, give the reasoning |
| "I disagree, it's fine" | I'm overriding you | Accept, move on |
| "That's intentional" | You're missing context | Note it, move on |

This applies everywhere — not just review. During implementation, if the user questions an approach, the agent should explain why it chose it rather than immediately switching to whatever the user seems to be suggesting. Only an explicit correction ("do it this way instead") should change the approach.

## Core Design: The Collaboration Spectrum

Collaboration is **fluid, not binary**. Five levels on a spectrum, where the agent can shift between them mid-task based on user cues.

### Levels

| Level | Name | Behaviour | When to use |
|-------|------|-----------|-------------|
| 1 | **Autonomous** | Do it, report at the end | You know the code, know the domain, want throughput |
| 2 | **Narrate** | Do it, explain reasoning as you go, user can interrupt (accepts they may be late) | You have context, want visibility, don't need pauses |
| 3 | **Checkpoint** | Pause at natural breakpoints (task boundaries), let user steer | **DEFAULT** — you want to stay involved and build context |
| 4 | **Pair** | Work through it together step by step, explain concepts, reference patterns | New to this area, want to understand before code is written |
| 5 | **Teach** | Like pair but pedagogical — flip questions back, ask user to reason through things | Genuine learning moments, building deep understanding |

### Default: Checkpoint

Out of the box, with no signal from the user, the agent operates in **checkpoint mode**. This means:
- The agent breaks work into a task list upfront
- Each task is a natural checkpoint
- After completing each task, the agent checks in: shows what it did, explains why, and waits before moving on
- The user can steer: "looks good", "do that differently", "skip that, I'll do it", or "I get it now, just do the rest"

### How levels are set

**The user sets the starting point via natural language.** No flags, no config menus.

- "Ship #42" — no signal, start at checkpoint (default)
- "Ship #42, I know this area" — start at narrate or autonomous
- "Ship #42, new to this repo" — start at pair
- "Ship #42, can you teach me this pattern?" — start at teach

**No git-based inference.** Git blame is unreliable — you might have deep context from reviewing PRs, pair programming, working in forks, or knowing the patterns from other repos. The agent should never presume to know your context level.

### Mid-task shifting

The level is fluid. Within a single task the user can shift:

- **"Why?"** or **"Walk me through this"** — agent shifts up toward pair/teach
- **"I get it now, just do the rest"** or **"Go ahead"** — agent shifts down toward autonomous
- **"Just do this part"** — autonomous for this specific task, then back to previous level

The key moment: **"I get it now, just do the rest"** means the collaboration worked. The user built enough context that they trust what's coming next. The agent earned autonomy rather than assuming it.

### Always-pause checkpoints

Regardless of collaboration level, some things should **always** cause a pause:

- Changing existing behaviour (not just adding new code)
- Multiple valid architectural approaches
- Something that contradicts what the user said they wanted
- Deleting or significantly modifying code someone else owns
- Anything surprising or risky

These are about risk, not context level.

## Task List as Collaboration Surface

The task list is the core collaboration mechanism. It's where the user sees the plan, tracks progress, and steers.

### How it works

The agent breaks work into tasks at a **meaningful granularity**:
- For a form: each input/field is roughly a task
- For a controller: each function is roughly a task
- For a refactor: each logical change is a task

### Example flow

User: "Work on issue #42 — add validation to the create form"

Agent:
> Here's how I'd break this down:
> 1. Add name field validation (required, max length)
> 2. Add email field validation (format check)
> 3. Add error messages using the existing `AlertGroup` pattern from `AuthPolicyCreatePage.tsx`
> 4. Disable submit button until valid
>
> Starting with #1 — the other forms in this repo use `validateRequired()` from `utils/validation.ts`, I'll follow the same pattern.

After completing #1, the agent checks in, shows what it did, and waits. User can:
- "Looks good, next" — continue at checkpoint
- "Actually, let's do that differently" — redirect
- "Skip #2, I'll do that one" — user takes ownership
- "I get it now, just do the rest" — shift to autonomous for remaining tasks

### Granularity by level

- **Autonomous**: agent runs through the whole list, no pauses
- **Narrate**: agent runs through but explains each step as it goes
- **Checkpoint**: agent pauses after each task
- **Pair**: tasks are broken down more granularly, agent explains concepts at each step
- **Teach**: like pair, but agent asks user to reason through some tasks before showing the approach

## Review Workflow

All code is AI-generated — review is always reviewing code the agent wrote. The collaboration level during implementation directly affects how much review is needed afterward.

### Review effort scales with implementation involvement

- **You were at checkpoint/pair/teach during implementation** — you already have context. Review can be lighter: "here's the final diff, you saw each piece as it was built, anything to change?"
- **You were at autonomous/narrate during implementation** — you need more review. Checkpoint review with findings one at a time.
- **Someone else's agent wrote it, you have no context** — pair review, walk through the diff together.

### Review across the spectrum

| Level | Review behaviour |
|-------|------------------|
| **Autonomous** | Fan out reviewers, collect findings, present summary |
| **Narrate** | Same process, but you can follow along as each reviewer works, with running summary |
| **Checkpoint** | Present findings **one at a time**. Each finding is a checkpoint — engage, question, dismiss, or dig deeper before moving to the next |
| **Pair** | Walk through the diff section by section. Not just findings — explain what the code does, why, point out patterns |
| **Teach** | Ask user to spot issues before revealing them. "Look at the `onSubmit` handler. There's something missing. Can you see it?" |

### Checkpoint review flow (default)

Each finding presented individually:

> Finding 1/4: `handleSubmit` doesn't handle the case where the API returns a 409 conflict. The other forms in this repo handle this in `onSubmit` — see `AuthPolicyCreatePage.tsx:87`. Do you want to flag this or skip it?

User can respond:
- **"Is this actually a problem?"** — agent explains the risk, shows evidence (holds its ground)
- **"Show me where else this pattern is used"** — agent pulls up similar code so you can judge
- **"Good catch, tell me more"** — shifts toward pair/teach for that finding
- **"I disagree, skip it"** — moves on

### Review summary

After working through all findings:

> 4 findings reviewed:
> - 2 addressed (null check added, error handling fixed)
> - 1 dismissed (intentional pattern, not a bug)
> - 1 noted for future (refactor candidate, not blocking)

## Multi-Issue and Parallel Work

At the autonomous/narrate end of the spectrum, users may want multiple issues in flight simultaneously.

### Rules

- **Checkpoint, pair, teach** — one issue at a time. You're actively involved, context switching would break the collaboration.
- **Autonomous, narrate** — multiple issues can run in parallel in worktrees.
- **Mixed** — some issues at checkpoint, others autonomous. Per-issue, not global.

### Example

> "Ship #1, walk me through it. Also ship #2 and #3, I know those areas, just do them."

Agent:
> Got it. Starting #1 in checkpoint mode — here's the task breakdown.
> #2 and #3 are running autonomously in the background. I'll report back when they're done.

### Two layers of visibility for parallel work

**1. Summary dashboard** — stable, pinned, shows state of all issues at a glance:

```
#1 [checkpoint]  ████░░░░ 3/6  Waiting: "Add error messages — follow AlertGroup pattern?"
#2 [autonomous]  ██████░░ 5/8  Working: Adding unit tests
#3 [narrate]     ██░░░░░░ 2/7  Working: Implementing webhook handler
```

**2. Narration stream** — per-issue, detailed, scrolling. Each narration has a **running summary at the top** so when you context switch back to an issue, you catch up in two sentences:

> **Summary:** Completed name validation, email validation. **Note: chose client-side validation only — the API doesn't have a validation endpoint. Worth discussing if server-side is needed.** Currently working on error messages.

> **Narration:** Looking at the error message patterns... `AuthPolicyCreatePage.tsx` uses an `Alert` inside a `StackItem` with variant="danger"...

The summary captures decisions made and flags anything notable — so even autonomous work isn't a black box.

### Always-pause in parallel

If a background issue hits an always-pause moment, it surfaces in the summary and waits:

```
#2 [autonomous]  ██████░░ 5/8  ⚠ NEEDS INPUT: About to change existing auth behavior
```

## Scope

Full SDLC coverage:
- **Implement** — work on issues with the collaboration spectrum
- **Review** — review PRs with explanation of findings
- **Triage** — assess issues for readiness
- **Refine** — turn vague issues into specs
- **Ship** — full lifecycle from issue to merged PR

## Architecture

**Independent plugin** designed from first principles around the collaborative philosophy.

**Structure** (Claude Code plugin conventions):
```
agents/           subagent definitions (one .md per agent)
skills/           on-demand skills (SKILL.md per directory)
hooks/            lifecycle hooks
references/       supporting docs
docs/             architecture decisions
.claude-plugin/   plugin manifest
```

**Key design choice**: agents are not separate per-mode. The same implement agent operates across the full spectrum. Collaboration level is a parameter, not a different agent.

## Resolved Questions

1. **Name** — **Brock**. Named after Eddie Brock (Marvel's Venom). The symbiote isn't a tool you use — it's a partner that adapts to you. The arc is about going from fighting it to finding a symbiotic relationship that makes both of you better.
2. **How does the agent communicate task list + progress?** — **Combination approach**: Claude Code task system (`TaskCreate`/`TaskUpdate`) for structured progress tracking (the stable "where is everything" view), inline text for narration, reasoning, and summaries. Try this and adjust based on how it feels in practice.
3. **How does the teach mode "flip questions back" work in practice?** — **Guardrails for teach mode:**
   - Only flip questions when there's a learnable concept, not for trivia or implementation details
   - Always give enough context to reason from ("look at this file, compare to that file" not just "what do you think?")
   - Graceful fallback: if the user says "I don't know" or "just tell me", answer immediately without judgement
   - Don't overdo it: one or two per task, not every step
   - Works best during review ("spot the issue") and architectural decisions ("what pattern would you use here?")
4. **How opinionated should agents be about tech stack?** — **Generic**. Brock works with any repo, any language. Domain knowledge comes from the repo's CLAUDE.md (patterns, conventions, architecture), not baked into the plugin. The agent reads the repo's docs and uses them to reference patterns, explain concepts, and teach. This means any team with a CLAUDE.md gets rich, context-aware collaboration without Brock needing domain-specific code.
5. **Should there be a way to set defaults per-repo or per-team?** — **Yes**. Teams can set a default collaboration level in their CLAUDE.md (e.g. `brock-default: pair` for a complex repo where everyone should start collaborative). Individual users can always override per-task via natural language ("I know this area, just do it"). The repo default is a starting suggestion, not a lock.
