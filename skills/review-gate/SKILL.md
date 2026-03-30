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

## The Process

### Prep

1. Run the repo's smallest-diff audit or equivalent cleanup. Reviewers should see clean code.
2. Select reviewers from the diff using the repo's dispatch table, `workflow/reviewers/`, or the fallback heuristics in `review-lifecycle.md`.

### Round (max 2)

3. Dispatch each selected reviewer as a parallel subagent. Each gets the changed files, relevant reference docs, and the `reviewer-output-contract.md` as context. **Fallback:** Run reviewers sequentially in the same session.
4. Collect all findings into a single findings document (see `review-lifecycle.md` for format).
5. Dispatch findings-validator subagent (see `findings-validator-prompt.md`). Fresh context — did not participate in the review. Classifies each finding as VALID, FALSE_POSITIVE, or ALREADY_HANDLED. **Fallback:** Validate sequentially in the same session.
6. If no valid findings remain: **PASS** — exit the round.
7. Write validated findings as fix task contracts (scope, invariant, validation command — not patches).
8. Dispatch implementer subagents for fix tasks. Same execution path as plan tasks. **Fallback:** Fix sequentially in the same session.
9. Re-run only the reviewers that flagged findings, scoped to changed files.
10. If valid findings remain after round 2: **escalate to user** with the findings document.

### Post-Review

11. Run the smallest-diff audit again after all fixes.
12. Record any post-review change in the repo's feedback queue immediately.

## Common Mistakes

**Patching directly instead of writing fix contracts**
- Patches are band-aids. Fix contracts go through the same quality pipeline as original work.

**Skipping finding validation**
- False positives waste implementer cycles and add noise to the patch loop.

**Grinding past round 2**
- If the same finding returns twice, the issue is structural. Escalate.

**Running all reviewers on round 2**
- Re-run only the reviewers that flagged findings, scoped to changed files.

## Red Flags

**Never:**
- Patch findings directly without fix contracts
- Skip finding validation
- Run more than 2 rounds
- Proceed to finishing without resolving or escalating all findings

**Always:**
- Run smallest-diff before reviewers
- Validate every finding before acting
- Record post-review changes in the feedback queue

## Integration

**Called by:**
- **vollgas:subagent-driven-development** — After all tasks and final code review
- **vollgas:executing-plans** — After all batches complete

**Uses:**
- **vollgas:dispatching-parallel-agents** — For parallel reviewer dispatch when subagents are available

**Hands off to:**
- **vollgas:finishing-a-development-branch** — Only after review passes or escalation is resolved
