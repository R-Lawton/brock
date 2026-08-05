# Review Guide

Detailed review methodology for the review agent. Read this alongside `spectrum.md` (for your collaboration level behaviour) and `principles.md` (for always-pause moments and hold-your-ground).

This guide covers *how* to review. The spectrum covers *how to communicate* your findings.

## Review Strategy

Three passes on every review:

### 1. Scan

Run `git diff --stat` (or `gh pr diff --stat`) to understand scope. How many files changed, what types, how large. This determines your approach and sets expectations with the user.

### 2. Understand Intent

Read the issue or PR description. Understand what the change is *supposed* to do. Every finding is relative to intent — "this doesn't handle errors" is only a finding if error handling is relevant to what was requested.

### 3. Review

Work through the diff:
- **Small diffs (1-5 files, < ~200 lines):** file-by-file, review every line. Check surrounding code for pattern consistency.
- **Large diffs (15+ files, 500+ lines):** group by concern — API/interface changes first, then implementation, then tests, then config, then docs.
- **Medium diffs:** in between — group by concern, focus on the core change first.

Within each file, read in this order: public API / interface changes first (widest blast radius), then implementation, then error handling, then edge cases.

## Review Type Detection

Auto-detect what's being reviewed from the diff. The user can override ("just review the API changes", "focus on the test coverage").

| Type | Detection | Primary dimensions |
|---|---|---|
| **Code** | `.go`, `.ts`, `.tsx`, `.py`, `.rs`, `.java`, etc. | Correctness, security, error handling, patterns, backwards compat |
| **Tests** | `*_test.go`, `*.test.ts`, `*.spec.ts`, `test_*.py` | Assertion quality, coverage, edge cases, test isolation, no false passes |
| **Docs** | `.md`, `.adoc`, `.rst` | Accuracy against codebase, clarity, completeness, broken links |
| **Config** | `.yaml`, `.json`, `Makefile`, `Dockerfile`, `.env*` | Valid syntax, security (secrets, exposed ports), environment consistency |
| **Design docs** | `docs/design/`, `docs/specs/`, `*-design.md` | Feasibility, scope, contradictions, missing edge cases |
| **Mixed** | Multiple types in one diff | Apply per-file dimensions, flag cross-cutting concerns |

### Cross-cutting concerns in mixed diffs

Flag when:
- Code changed but tests not updated
- API changed but docs not updated
- Config changed but no corresponding code change explains why

## Review Dimensions

### Code

- **Correctness** — does it do what the issue asked for? Logic errors, off-by-ones, nil/null handling
- **Behaviour changes** — does it modify existing behaviour? If so, is that intentional?
- **API contracts** — signature changes, return type changes, new required params — anything consumers depend on
- **Backwards compatibility** — will this break existing callers, configs, or data?
- **Error handling** — errors caught at system boundaries (user input, API calls, file I/O), not swallowed silently
- **Concurrency** — shared state, race conditions, lock ordering (only when the diff touches concurrent code)
- **Naming** — do new names match existing conventions in the repo?
- **Security** — see Security section below

### Tests

- **Assertion quality** — tests assert specific behaviour, not just "no error"
- **Edge cases** — boundary values, empty inputs, error paths
- **Test isolation** — no shared mutable state between tests, no order dependence
- **False passes** — would this test pass even if the feature was broken?

### Docs

- **Accuracy** — does it match the current code? Outdated examples, wrong function names
- **Completeness** — are new features/APIs documented?
- **Working examples** — do code samples actually work?

## Security

Always checked, regardless of review type. Not a deep audit — pragmatic checks for things the diff might introduce.

### Always check

- **Injection** — user input flowing into SQL, shell commands, templates, or HTML without sanitisation
- **Auth/authz gaps** — new endpoints or routes missing authentication or permission checks
- **Secrets** — hardcoded credentials, API keys, tokens in code or config (even in examples)
- **Input validation** — data from external sources (API requests, file uploads, URL params) trusted without validation
- **Sensitive data exposure** — logging PII, returning internal errors to users, verbose error messages in production

### Check when relevant to the diff

- **Dependency changes** — new dependencies with known vulnerabilities, unnecessary permissions
- **CORS/CSP changes** — overly permissive cross-origin or content security policies
- **Crypto** — rolling custom crypto, weak algorithms, hardcoded IVs/salts
- **Resource exhaustion** — unbounded loops, missing pagination, no rate limiting on new endpoints

## Diff Scoping

Adapt your depth based on diff size. Be honest about what you covered.

### Small diff (1-5 files, < ~200 lines)

- Review every line
- File-by-file, full context
- Check surrounding code for pattern consistency

### Medium diff (5-15 files, ~200-500 lines)

- Group by concern, not alphabetical
- Focus on the core change first, then ripple outwards
- State what you checked closely vs skimmed: "I checked the auth changes closely, skimmed the test updates"

### Large diff (15+ files, 500+ lines)

- Start with `--stat` summary and state your plan: "This is a large diff. I'll focus on [core changes] and flag areas I couldn't review deeply."
- Prioritise: API/interface changes → new logic → modified logic → tests → config → docs
- Explicitly note what was skimmed and why — no pretending to have reviewed everything
- If the user asks to go back and properly review skimmed areas, do a full pass on those

**Key principle:** a review that says "I checked everything" on a 1000-line diff is lying. Better to say "I focused on the auth changes and the new endpoint, skimmed the test updates because they follow the mechanical pattern from the new code."

## What Not to Flag

### Don't flag

- **Style/formatting** — the linter's job, not yours
- **Personal preference** — "I would have named this differently" isn't a finding unless it breaks a repo convention
- **Pre-existing issues** — problems in unchanged code that the diff didn't introduce or worsen
- **Hypothetical futures** — "this might not scale if you had 10x the data" without evidence it matters now
- **Obvious comments** — don't suggest adding comments that restate what the code already says

### Do flag even if it feels nitpicky

- Naming that contradicts an established repo convention (consistency, not preference)
- Pre-existing issues that the diff *makes worse* or that are directly adjacent to changed code
- Security concerns, no matter how small

**The line:** does this finding prevent a real problem, or does it just reflect how you'd write it? If the latter, don't flag it.

## Readiness Review

When the router sets your focus to **readiness**, run this checklist. Readiness is a superset of correctness — you do everything a normal review does, plus the checks below.

The goal: catch everything before an external reviewer sees it.

### Step 1: Pre-ship checks

Run the repo's test, lint, and type-check scripts locally. Discover what to run from the project's CLAUDE.md and build config (package.json, Makefile, etc.).

- Run each check and report results
- Fix issues you can fix (lint auto-fix, simple test failures)
- **Always-pause on failures you can't fix** — stop and tell the user what broke, regardless of collaboration level
- If no scripts are discoverable, skip and note: "No pre-ship checks found — skipping to cleanliness scan"

### Step 2: Cleanliness scan

Scan the diff for things an author should catch before requesting review:

- Debug statements (`console.log`, `print()`, `fmt.Println` used for debugging, `debugger`)
- TODO/FIXME/HACK/XXX comments introduced in this diff
- Commented-out code blocks (not individual explanatory comments — blocks of dead code)
- Hardcoded test values that should be constants or config
- Leftover merge conflict markers

### Step 3: PR hygiene

Check the presentation of the work:

- Is the PR description present and does it explain what was done and why?
- Are commit messages descriptive (not "wip", "fix", "fix fix", "asdf")?
- Are there unrelated changes mixed into the diff?
- Are there files that shouldn't be committed (`.env`, build artifacts, editor config)?

### Step 4: Full correctness review

Run the normal correctness review using all dimensions from this guide — code, tests, docs, security, type-aware. This is the same review you'd do with `focus: correctness`.

### Step 5: Completeness check

Compare the implementation against the issue or PR description:

- Are all requirements addressed?
- Are there requirements that were partially implemented?
- Is there work that was started but not finished?
- Does the implementation do what the issue asked for, or did it drift?
