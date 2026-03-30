---
name: smallest-diff
description: Use after implementation to verify the diff is minimal and clean — checks for dead code, speculative additions, unnecessary touched files, scope creep, and diff noise. Run before committing or after reviewer patches. Read-only — reports findings but does not fix.
---

# Smallest Diff

## Overview

Audit the current diff for unnecessary changes. Every line in the diff should be traceable to an explicit requirement. Code that "might be useful later," files touched without meaningful change, and leftover debugging artifacts are flagged.

**Core principle:** The best diff is the smallest diff that achieves the goal. Read-only — report findings, never fix.

**Announce at start:** "I'm using the smallest-diff skill to audit the diff for unnecessary changes."

## When to Use

- After implementation, before committing
- Before dispatching reviewers (so reviewers focus on correctness, not cleanup)
- After reviewer patches (so patches didn't introduce noise)
- Before finishing a development branch (final cleanliness gate)
- Standalone anytime to audit uncommitted work

## Usage

Without `--base`: audit uncommitted changes (`git diff` + `git diff --cached`).
With `--base <ref>`: audit the full branch diff (`git diff <ref>...HEAD`).

## The Process

### Step 1 — Collect the diff

```bash
# Uncommitted mode (default)
git diff --stat
git diff --cached --stat
git diff
git diff --cached

# Branch mode (--base provided)
git diff <base>...HEAD --stat
git diff <base>...HEAD
```

### Step 2 — Analyze each changed file

For every file in the diff, check the following. Report findings per file.

#### Dead code (blocker)

- **Unreachable code paths:** Functions or branches that can never execute given the current call sites.
- **Unused imports:** Imports that are not referenced anywhere in the file.
- **Unused variables or parameters:** Declared but never read.

#### Speculative code (blocker)

- **Features not in the spec or task description:** Code that handles scenarios not required by the current task.
- **Premature abstractions:** Helpers, utilities, or base classes created for a single use site.
- **Configuration for hypothetical future requirements:** Feature flags, options objects, or parameters that have only one possible value in the current codebase.

#### Diff noise (warning)

- **Files touched without meaningful change:** Whitespace-only changes, import reordering without additions/removals, reformatting that lint didn't require.
- **Comments describing WHAT the code does:** Comments should explain WHY. Self-evident code needs no comment. Flag comments like `// Set the value` or `// Loop through items`.
- **Redundant type annotations:** When the language infers types, explicit annotations on internal code add noise unless they improve readability at call sites.
- **Console.log or debugger statements:** Leftover debugging artifacts.

#### Scope creep (warning)

- **Refactoring unrelated code:** Renaming variables, extracting functions, or restructuring code that was not part of the task.
- **Updating tests for changed behavior when the behavior change was not requested:** If a test was changed to make it pass (rather than the code being fixed), flag it.
- **Adding documentation to unchanged code:** Docstrings and annotations should only be added to new or modified code.

### Step 3 — Cross-file analysis

- **Are all new files necessary?** Flag any new file that could have been an edit to an existing file instead.
- **Are all touched files relevant to the task?** List each file and its relationship to the task. Flag files that don't have an obvious connection.

### Step 4 — Report

```
## Smallest Diff Audit

### Summary
Files changed: N
Lines added: N | Lines removed: N
Blockers: N | Warnings: N

### Blockers
- [file:line] Dead code: `unusedHelper()` is never called
- [file:line] Speculative: `retryCount` parameter added but not in the task spec

### Warnings
- [file:line] Diff noise: comment describes what code does, not why
- [file] Scope creep: reformatted imports in unrelated file

### Clean files
- [file] — minimal, relevant change

### Verdict
CLEAN — diff is minimal and every change is traceable to the task.
NEEDS_CLEANUP — N blockers must be addressed before commit.
```

## Common Mistakes

**Fixing issues found by this skill**
- This skill is read-only. Report and stop. The engineer or orchestrator decides what to clean up.

**Flagging test code as speculative**
- Tests for edge cases are NOT speculative — they're defensive. Only flag test code that tests functionality not present in the implementation.

**Flagging required project conventions as noise**
- Some projects require explicit annotations, specific comment styles, or documentation on all exports. Check CLAUDE.md before flagging.

## Red Flags

**Never:**
- Apply fixes — this skill is strictly read-only
- Audit files not in the diff
- Flag tests as speculative without checking the implementation
- Mark a diff as CLEAN when blockers exist

**Always:**
- Report every blocker
- Trace each change back to a requirement
- Check both staged and unstaged changes in uncommitted mode

## Integration

**Dispatched as sub-agent by:**
- **vollgas:review-gate** — Prep (before reviewers) and post-review (after fixes). Template: `review-gate/smallest-diff-prompt.md`. Model: Sonnet.

**Can also be invoked directly** via Skill tool for standalone diff auditing.

**Pairs with:**
- **vollgas:code-simplifier** — smallest-diff audits (read-only), code-simplifier refines (read-write). In review-gate: smallest-diff runs in prep, code-simplifier runs post-review.
- **vollgas:verification-before-completion** — smallest-diff checks diff cleanliness, verification checks functional correctness
