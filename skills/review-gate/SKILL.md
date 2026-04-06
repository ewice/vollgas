---
name: review-gate
description: Use after all implementation tasks are done and before finishing a development branch, when code needs reviewer coverage beyond what tests protect
---

# Review Gate

## Overview

Quality gate between execution and branch completion. Reviews what tests do not fully
protect: conventions, API shape, accessibility, security, lifecycle correctness, and
architectural fit.

**Core principle:** Collect findings, validate them, write fix contracts, dispatch implementers, re-review. Never patch directly.

**Announce at start:** "I'm using the review-gate skill to run the post-implementation review."

## Model Selection

| Role | Model | Phase |
|------|-------|-------|
| Smallest-diff audit | Sonnet | Prep + Post-review |
| Project reviewers | Opus | Review rounds |
| Findings validator | Opus | Review rounds |
| Fix implementers | Sonnet | Review rounds |
| Code-simplifier | Opus | Post-review (once) |

## The Process

### Prep

1. Dispatch `smallest-diff` sub-agent (Sonnet) to audit the diff. Template: `./smallest-diff-prompt.md`. If verdict is NEEDS_CLEANUP, fix blockers before proceeding.
2. Select reviewers using the Reviewer Discovery process below.

### Reviewer Discovery

Follow this priority order to find reviewers:

1. **Project dispatch table:** Check `vollgas/refs/reviewer-dispatch.md` in the project root. This file maps change types to reviewer names and their context files.
2. **Project reviewer directory:** Scan `vollgas/reviewers/` for reviewer definitions. Each reviewer is a file (e.g., `accessibility-reviewer.md`) containing: name, description, context file paths, and review prompt.
3. **Fallback heuristics:** Use the change-type heuristics in `review-lifecycle.md` to select a general-purpose reviewer.

**Dispatch table format** (in `vollgas/refs/reviewer-dispatch.md`):

```markdown
| Change type | Reviewers | Context files |
|---|---|---|
| UI or DOM behavior | accessibility-reviewer, framework-reviewer | vollgas/refs/accessibility-contract.md, vollgas/refs/lit-patterns.md |
| Public API | api-reviewer | vollgas/refs/api-contract.md |
| Tests | test-reviewer | vollgas/refs/test-patterns.md |
```

Each reviewer file in `vollgas/reviewers/` must include the `reviewer-output-contract.md` as context so findings follow the standard format.

### Round (max 2)

3. Dispatch each selected reviewer as a parallel sub-agent (Opus). Each gets the changed files, relevant reference docs, and the `reviewer-output-contract.md` as context. **Fallback:** Run reviewers sequentially in the same session.
4. Collect all findings into a single findings document (see `review-lifecycle.md` for format).
5. Dispatch findings-validator sub-agent (Opus). See `findings-validator-prompt.md`. Fresh context — did not participate in the review. Classifies each finding as VALID, FALSE_POSITIVE, or ALREADY_HANDLED. **Fallback:** Validate sequentially in the same session.
6. If no valid findings remain: **PASS** — exit the round.
7. Write validated findings as fix task contracts (scope, invariant, validation command — not patches). If a fix contradicts the original plan (e.g., the plan defined a field that a reviewer correctly identifies as an anti-pattern), append a deviation entry to the plan file's `## Deviations` section (create the section if it doesn't exist). Format: `### Task N — {title}\n- **Deviated from:** {what the plan said}\n- **Actual approach:** {what was implemented instead}\n- **Why:** {reviewer finding that motivated the change}`. This keeps the plan accurate for future readers.
8. Dispatch implementer sub-agents (Sonnet) for fix tasks. Same execution path as plan tasks. **Fallback:** Fix sequentially in the same session.
9. Re-run only the reviewers that flagged findings, scoped to changed files.
10. If valid findings remain after round 2: **escalate to user** with the findings document.

### Post-Review

11. Dispatch `smallest-diff` sub-agent (Sonnet) to verify fixes didn't introduce noise. Template: `./smallest-diff-prompt.md`.
12. Dispatch `code-simplifier` sub-agent (Opus) as final polish. Template: `./code-simplifier-prompt.md`. Verify tests pass after refinements.
13. **Post-reviewer feedback-queue check.** Steps 11–12 run after all reviewers have passed. Any changes they produce are post-reviewer fixes and MUST be recorded in `vollgas/.feedback-queue.md` per the project's Post-Reviewer Protocol. After each sub-agent completes, diff its changes (`git diff HEAD~1`) and write a feedback-queue entry for each meaningful change. Skip this step only if the sub-agent made zero changes.
14. If any validated findings were produced during the review rounds, write them to `vollgas/review-findings/YYYY-MM-DD-<branch-name>.md` using the format below. One file per branch, written once. If zero validated findings were produced, skip this step.

## Sub-Agent Templates

| Template | Purpose | Model |
|----------|---------|-------|
| `./smallest-diff-prompt.md` | Audit diff for dead code, speculative additions, noise | Sonnet |
| `./code-simplifier-prompt.md` | Final polish — refine clarity, naming, consistency | Opus |
| `./findings-validator-prompt.md` | Classify reviewer findings as valid/false positive | Opus |

## Findings File Format

When the review rounds produce validated findings, persist them for retrospective analysis:

**Path:** `vollgas/review-findings/YYYY-MM-DD-<branch-name>.md`

**Format:**

```markdown
# Review Findings — <branch-name>

Date: YYYY-MM-DD
Branch: <branch-name>
Rounds: <number of review rounds>

## Findings

### Finding 1
- **Reviewer:** <reviewer-name>
- **Description:** <what was flagged>
- **Resolution:** fixed | escalated
- **Round:** <which round flagged it>

### Finding 2
...
```

Only include validated findings (VALID classification from findings-validator). Omit FALSE_POSITIVE and ALREADY_HANDLED findings. Capture the final resolution status after all review rounds complete (steps 3–10), not after post-review polish (steps 11–12).

If a findings file already exists for this branch, overwrite it — only one findings snapshot per branch is kept.

## Contracts

| File | Purpose |
|------|---------|
| `./reviewer-output-contract.md` | Output format all reviewers must follow — included as context in reviewer prompts |

## Common Mistakes

**Patching directly instead of writing fix contracts**
- Patches are band-aids. Fix contracts go through the same quality pipeline as original work.

**Skipping finding validation**
- False positives waste implementer cycles and add noise to the patch loop.

**Grinding past round 2**
- If the same finding returns twice, the issue is structural. Escalate.

**Running all reviewers on round 2**
- Re-run only the reviewers that flagged findings, scoped to changed files.

**Running code-simplifier before reviewers**
- Reviewers should see the implementer's actual code. Code-simplifier runs once, post-review, as final polish.

## Red Flags

**Never:**
- Patch findings directly without fix contracts
- Skip finding validation
- Run more than 2 rounds
- Proceed to finishing without resolving or escalating all findings
- Run code-simplifier before reviewers (it masks issues)

**Always:**
- Run smallest-diff before reviewers (removes code that shouldn't exist)
- Run smallest-diff + code-simplifier after all fixes (final polish)
- Write feedback-queue entries for any post-reviewer changes (step 13)
- Append plan deviations when fixes contradict the original plan (step 7)
- Validate every finding before acting
- Persist validated findings to `vollgas/review-findings/` after all rounds complete

## Integration

**Called by:**
- **vollgas:subagent-driven-development** — After all tasks and final code review
- **vollgas:executing-plans** — After all batches complete

**Uses:**
- **vollgas:dispatching-parallel-agents** — For parallel reviewer dispatch when sub-agents are available
- **vollgas:smallest-diff** — Diff audit (prep + post-review)
- **vollgas:code-simplifier** — Code refinement (post-review)

**Hands off to:**
- **vollgas:finishing-a-development-branch** — Only after review passes or escalation is resolved
