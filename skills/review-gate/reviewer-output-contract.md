# Reviewer Output Contract

All reviewers dispatched by the review-gate must use this output format.
The findings-validator reads this format — inconsistent output wastes its context.

## Required Output Structure

```markdown
## Review: {reviewer name}

### Findings

#### Finding 1: {one-line summary}
- Severity: BLOCKER | OBSERVATION
- File: {path/to/file}:{line range}
- Issue: {what is wrong — specific, not vague}
- Evidence: {code snippet or reference that proves the issue}
- Suggested fix direction: {what should change — not a patch, just direction}

#### Finding 2: ...

### No Findings
If nothing was found, state explicitly:
- "No findings. Reviewed {N} files against {which reference docs}."

### Scope
- Files reviewed: {list}
- Reference docs used: {list}
- Areas outside scope: {anything relevant that was not reviewed and why}
```

## Rules

**Severity levels — only two:**
- `BLOCKER` — must be fixed before the branch can ship. Correctness issues, contract violations, security problems, accessibility failures.
- `OBSERVATION` — worth noting but not a gate. Style preferences, minor improvements, alternative approaches.

Do not invent intermediate levels. If you are unsure whether something is a blocker, it is an observation.

**Evidence is required.** Every finding must include the code or reference that proves the issue exists. A finding without evidence ("error handling seems weak") is not actionable and will be discarded by the validator.

**One finding per issue.** Do not bundle related issues into a single finding. The validator and implementer need to act on each independently.

**Stay in scope.** Review only the changed files and the reference docs you were given. Do not audit the entire codebase. If you notice something outside scope that is concerning, list it under "Areas outside scope."

**No patches.** Suggest a fix direction ("the aria-controls reference should use Element Reference API instead of IDREF"), not a code patch. The implementer decides HOW.
