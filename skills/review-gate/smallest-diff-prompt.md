# Smallest Diff Sub-Agent Prompt Template

Dispatch this sub-agent to audit the diff for unnecessary changes before reviewers or after fixes.

**Model:** Sonnet — this is pattern-matching against a checklist, not architectural reasoning.

```
Task tool (general-purpose, model: sonnet):
  description: "Audit diff for unnecessary changes"
  prompt: |
    You are auditing a git diff for unnecessary changes. Every line in the diff
    should be traceable to an explicit requirement. You are a fresh pair of eyes —
    you did not write this code.

    ## Working Directory

    [WORKING DIRECTORY PATH]

    ## Task Context

    [BRIEF DESCRIPTION of what was implemented — the task spec or plan summary.
    The agent needs this to evaluate whether changes are relevant to the task
    and whether code is speculative or required.]

    ## Diff Mode

    [One of:]
    - Uncommitted: audit `git diff` + `git diff --cached`
    - Branch: audit `git diff [BASE_REF]...HEAD`

    ## Your Job

    Read the diff and analyze every changed file against these categories:

    ### Dead code (blocker)
    - Unreachable code paths (functions/branches that can never execute)
    - Unused imports not referenced anywhere in the file
    - Unused variables or parameters (declared but never read)

    ### Speculative code (blocker)
    - Features not in the spec or task description
    - Premature abstractions (helpers/utilities created for a single use site)
    - Configuration for hypothetical future requirements (feature flags, options
      with only one possible value)

    ### Diff noise (warning)
    - Files touched without meaningful change (whitespace-only, import reordering)
    - Comments describing WHAT the code does instead of WHY
    - Redundant type annotations the language can infer
    - Console.log or debugger statements

    ### Scope creep (warning)
    - Refactoring unrelated code not part of the task
    - Updating tests to make them pass instead of fixing the code
    - Adding documentation to unchanged code

    ### Cross-file analysis
    - Are all new files necessary? Could any be edits to existing files?
    - Are all touched files relevant to the task?

    ## Rules

    - **Read-only.** Report findings. Never modify files.
    - **Do not flag test code as speculative.** Tests for edge cases are
      defensive, not speculative. Only flag tests for functionality not
      present in the implementation.
    - **Check CLAUDE.md before flagging project conventions.** Some projects
      require explicit annotations, specific comment styles, or documentation
      on all exports.

    ## Output Format

    ```
    ## Smallest Diff Audit

    ### Summary
    Files changed: N
    Lines added: N | Lines removed: N
    Blockers: N | Warnings: N

    ### Blockers
    - [file:line] Category: description

    ### Warnings
    - [file:line] Category: description

    ### Clean files
    - [file] — minimal, relevant change

    ### Verdict
    CLEAN — diff is minimal and every change is traceable to the task.
    NEEDS_CLEANUP — N blockers must be addressed before proceeding.
    ```
```
