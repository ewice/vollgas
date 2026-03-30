---
name: code-simplifier
description: Use as a final polish pass after review fixes are applied, or standalone to refine recently modified code for clarity, consistency, and maintainability while preserving all functionality
---

# Code Simplifier

## Overview

Simplify and refine recently modified code for clarity, consistency, and maintainability without changing what it does.

**Core principle:** Readable, explicit code over clever compact code. Preserve all functionality — only change how it's expressed.

**Announce at start:** "I'm using the code-simplifier skill to clean up recently modified code."

## When to Use

- After all review fixes are applied (final polish before finishing branch)
- Standalone: after writing or modifying code when not in a review-gate flow
- **Not before reviewers** — reviewers should see the implementer's actual code, not a polished version

## The Process

### 1. Identify Recently Modified Code

Find all files written or edited in the current session. Focus only on those files — do not audit the entire codebase.

### 2. Analyze for Simplification Opportunities

Per file, check for:

- **Unnecessary complexity:** Nested conditions that could be flattened, abstractions that add indirection without clarity
- **Redundancy:** Duplicated logic, variables that restate what's already clear, comments that describe obvious code
- **Naming:** Variables, functions, and types that don't clearly describe their purpose
- **Dead code:** Unused imports, unreachable branches, leftover debug statements
- **Consistency:** Does this code follow the patterns established elsewhere in the project? Check CLAUDE.md for project standards.

### 3. Apply Refinements

Rules:
- **Never change behavior.** All original features, outputs, and side effects must remain identical.
- **Follow project standards** from CLAUDE.md (naming conventions, import style, function style, error handling patterns).
- **Prefer explicitness over brevity.** Avoid nested ternaries — use `if/else` or `switch`. Avoid dense one-liners that sacrifice readability.
- **Don't over-simplify.** Do not remove useful abstractions, combine unrelated concerns, or reduce code to fewer lines at the cost of clarity.
- **One concern per change.** If a file is doing too much, note it — but don't restructure it. Scope is recently modified code only.

### 4. Verify Functionality Unchanged

After applying refinements, confirm:
- The code still compiles / passes type checks
- Tests still pass (run if the session has a test command available)
- No behavior was changed

### 5. Report Changes

State only significant changes that affect understanding. Skip cosmetic changes.

## Common Mistakes

**Changing behavior while "simplifying"**
- Extracting a function changes closure scope and can break things
- Renaming a public export is a breaking change
- Reordering operations with side effects changes behavior
- Fix: Re-read the original code before touching it

**Over-simplifying**
- Inlining a well-named helper makes the call site harder to read
- Merging two functions that handle different concerns creates coupling
- Fix: Ask "does this make the code easier to understand?" If no, leave it

**Auditing beyond scope**
- The goal is cleaning up what changed, not improving the whole codebase
- Fix: Only touch files modified in the current session

## Red Flags

- About to rename a public API surface
- About to inline a function used in more than one place
- About to change logic while "simplifying" a condition
- Refactoring code that wasn't touched this session

## Integration

**Dispatched as sub-agent by:**
- **vollgas:review-gate** — Post-review only (after all fixes, as final polish). Template: `review-gate/code-simplifier-prompt.md`. Model: Opus.

**Can also be invoked directly** via Skill tool for standalone code refinement.

**Pairs with:**
- **vollgas:smallest-diff** — smallest-diff removes code that shouldn't exist (read-only), code-simplifier improves how remaining code is expressed (read-write). In review-gate: smallest-diff runs first, code-simplifier runs after.
