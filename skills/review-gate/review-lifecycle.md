# Review Lifecycle Reference

## Findings Document Format

```markdown
## Round N Findings

### Finding 1: one-line summary
- Reviewer: which reviewer flagged this
- File: path/to/file
- Issue: what is wrong
- Validation: VALID | FALSE_POSITIVE | ALREADY_HANDLED
- Reasoning: why this classification
- Fix contract (if valid):
  - Scope: files to change
  - Invariant: what must be true after the fix
  - Validation: command to verify
```

## Fallback Reviewer Heuristics

When the repo has no `vollgas/refs/reviewer-dispatch.md` or `vollgas/reviewers/` directory, select reviewers by change type:

| Change type | Reviewers |
|---|---|
| UI or DOM behavior | accessibility, framework, security |
| Public API | API review |
| Tests | test review |
| Shared logic or controllers | code quality, security |
| Build or configuration | code quality, security |

When no project-specific reviewers exist, dispatch a single general-purpose code quality
reviewer as the minimum gate.

## Post-Review Fix Record

If code changes after reviewers pass and the repo provides no queue format, use:

```markdown
## Entry: one-line summary

- Fixed file: path/to/file
- Pre-fix state: what was wrong
- Fix applied: what changed
- Reviewer scope: reviewer or concern area
- Classification: quality | correctness
- Reasoning: why this mattered
```
