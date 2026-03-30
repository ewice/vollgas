---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write comprehensive implementation plans assuming the implementer has zero context for our codebase. Document everything they need to know: which files to touch for each task, test code, implementation contracts, docs they might need to check, how to verify it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain. Assume they don't know good test design very well.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Context:** This should be run in a dedicated worktree (created by brainstorming skill).

**Save plans to:** `docs/vollgas/plans/YYYY-MM-DD-<feature-name>.md`
- (User preferences for plan location override this default)

## Scope Check

If the spec covers multiple independent subsystems, it should have been broken into sub-project specs during brainstorming. If it wasn't, suggest breaking this into separate plans — one per subsystem. Each plan should produce working, testable software on its own.

## File Structure

Before defining tasks, map out which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.

- Design units with clear boundaries and well-defined interfaces. Each file should have one clear responsibility.
- You reason best about code you can hold in context at once, and your edits are more reliable when files are focused. Prefer smaller, focused files over large ones that do too much.
- Files that change together should live together. Split by responsibility, not by technical layer.
- In existing codebases, follow established patterns. If the codebase uses large files, don't unilaterally restructure - but if a file you're modifying has grown unwieldy, including a split in the plan is reasonable.

This structure informs the task decomposition. Each task should produce self-contained changes that make sense independently.

## Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**
- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step

## Plan Code Granularity

Plans are executed by implementers with no prior context. What you write determines whether
they reason or copy-paste. Three tiers:

**Tests — verbatim.** Tests define the contract. They must be exact, copy-paste ready code.
The implementer should be able to paste the test, run it, see it fail, and know exactly what
to build.

**Trivial mechanical code — verbatim.** Type definitions, barrel exports, registration guards,
context interfaces, declarative wiring (route tables, config maps, DI registrations).
There's nothing to reason about — just write the code.

**Anything with logic — contracts, not code.** State machines, algorithms, component wiring,
integration logic. Describe WHAT the file is responsible for, not HOW to write it:
- Exact method signatures and return types
- Invariants ("selected must never contain a disabled id")
- Pseudocode for non-obvious algorithms
- References to existing patterns to follow ("same approach as AccordionEngine.toggle")

**Why not exact code for logic?** When plan code is copy-paste ready, implementers copy
without judgment — bugs in plan code propagate undetected — and reviewers degrade to
"does this match the plan?" instead of "is this correct?" Contracts distribute reasoning:
architecture in the plan, tactical judgment during implementation, correctness during review.
Implementers are expected to ask questions when contracts are ambiguous — that's by design.

**Self-review checklist for code granularity** — run this as you write each task, not as
a final pass. Rewrite if any of these are true:
- The section contains more than 5 lines of copy-paste code with logic (not tests, not type definitions or other tier-2 mechanical code)
- The section describes HOW to implement rather than WHAT the file is responsible for
- An implementer could copy the section verbatim without making any design decisions

Rewrite offending sections as: interface definitions, invariants, or pseudocode.

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use vollgas:subagent-driven-development to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

## Task Structure

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

`exact/path/to/file.py` — responsible for [one-sentence purpose].

Contract:
- `function(input: InputType) -> ReturnType` — [what it does, not how]
- Invariant: [condition that must always hold]
- Follow the pattern in `exact/path/to/reference.py`

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## No Placeholders

Every step must contain the actual content an engineer needs. These are **plan failures** — never write them:
- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above" (without actual test code)
- "Similar to Task N" (repeat the content — each task is dispatched in isolation)
- Steps that describe what to do without showing how (test steps need code; implementation steps need contracts)
- References to types, functions, or methods not defined in any task
- Vague contracts ("implement the state logic") — contracts must have method signatures, invariants, or pseudocode

## Remember
- Exact file paths always
- Tests and trivial code: verbatim. Implementation logic: contracts (see Plan Code Granularity)
- Exact commands with expected output
- DRY, YAGNI, TDD, frequent commits

## Self-Review

After writing the complete plan, look at the spec with fresh eyes and check the plan against it. This is a checklist you run yourself — not a subagent dispatch.

**1. Spec coverage:** Skim each section/requirement in the spec. Can you point to a task that implements it? List any gaps.

**2. Placeholder scan:** Search your plan for red flags — any of the patterns from the "No Placeholders" section above. Fix them.

**3. Type consistency:** Do the types, method signatures, and property names you used in later tasks match what you defined in earlier tasks? A function called `clearLayers()` in Task 3 but `clearFullLayers()` in Task 7 is a bug.

If you find issues, fix them inline. No need to re-review — just fix and move on. If you find a spec requirement with no task, add the task.

## Execution Handoff

After saving the plan:

**"Plan complete and saved to `docs/vollgas/plans/<filename>.md`. Ready to execute with subagent-driven development — fresh subagent per task with spec and code quality review between tasks. Proceed?"**

- **REQUIRED SUB-SKILL:** Use vollgas:subagent-driven-development
- Fresh subagent per task + two-stage review (spec compliance, then code quality)

**Fallback:** If the environment does not support subagents, use vollgas:executing-plans instead.
