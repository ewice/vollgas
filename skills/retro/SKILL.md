---
name: retro
description: Use after finishing a development branch to reflect on review-gate findings, detect recurring patterns, and draft changes to project conventions or Vollgas skills — closes the learning loop
---

# Retro

## Overview

Retrospective skill that closes the learning loop. Analyzes what the review-gate caught, detects recurring patterns, and drafts concrete changes so the same issues stop appearing.

**Core principle:** Recurring review findings are process gaps. Close them.

**Announce at start:** "I'm using the retro skill to analyze review findings and identify recurring patterns."

## Model Selection

Runs in the main session (Opus). No subagents — analysis and approval require conversational back-and-forth with the user.

## When To Use

Invoke after `finishing-a-development-branch` when you want to reflect on what the review-gate caught. Opt-in — not every branch needs a retro. Use it when:
- The review-gate caught several findings
- You suspect a pattern is recurring
- You want to tighten project conventions

## Inputs

### Findings Files

Read from `vollgas/review-findings/`. Each file was written by the review-gate and contains validated findings for one branch.

### Retro Log

Read from `vollgas/retro-log.md`. Contains all previous retro entries with finding categories and occurrence counts. Created on first retro run.

## The Process

### Step 1: Read Context

Read `vollgas/retro-log.md` (if it exists) and scan `vollgas/review-findings/` for unprocessed files — any findings file not already referenced in the retro log.

A findings file is considered already processed if the retro log contains an entry whose heading includes the branch name from the filename (e.g., `2026-03-31-feat-login.md` is processed if the log has a `## 2026-03-31 — feat-login` entry).

**If no unprocessed findings files exist:**
```
No unprocessed review findings found. Nothing to analyze.
```
Exit. Do not proceed.

### Step 2: Analyze Findings

For each validated finding in unprocessed files:

1. **Assign a category** using the `reviewer/topic` format (e.g., `accessibility/aria-labels`, `api/error-shape`).
2. **Check existing categories first.** Scan `retro-log.md` for categories that match the finding. Reuse an existing category if the finding belongs to the same class. Only create a new category when no existing one fits.
3. **Count occurrences** across the full retro log including the current findings.

### Step 3: Log

Append an entry to `vollgas/retro-log.md` (create the file if it doesn't exist):

```markdown
## YYYY-MM-DD — <branch-name>

| Finding | Reviewer | Category | Occurrences |
|---------|----------|----------|-------------|
| Description of finding | reviewer-name | reviewer/topic | cumulative count |

Actions taken: <summary of drafted changes, or "none (below threshold)">
```

### Step 4: Skip or Act

- Findings with **< 2 total occurrences**: log only. Inform the user these are logged but below the recurrence threshold.
- Findings with **2+ total occurrences**: these are recurring patterns. Proceed to Step 5.

If no findings meet the threshold, report this and exit:
```
All findings logged. No recurring patterns detected yet (all below threshold of 2).
```

### Step 5: Classify

For each recurring pattern, classify as:

- **Project-scoped** — pattern is about a project-specific technology, domain, or convention
- **Global** — pattern is about the development process itself (TDD discipline, review quality, plan accuracy, skill gaps)

### Step 6: Draft Changes

**For project-scoped patterns**, draft concrete changes to one of:
- An existing reviewer contract in `vollgas/refs/` (add a rule)
- `vollgas/refs/reviewer-dispatch.md` (map a new change type)
- The project's `CLAUDE.md` (add a convention)
- A new reviewer file in `vollgas/reviewers/` (if no existing reviewer covers the pattern)

Read the target file before drafting. The draft should be a specific edit, not a vague suggestion.

**For global patterns**, draft a proposal file to:
```
vollgas/retro-proposals/YYYY-MM-DD-<description>.md
```

Each proposal describes: which Vollgas skill to change, what section to update (Common Mistakes, Red Flags, anti-patterns), and the exact text to add.

### Step 7: Present for Review

Present each draft to the user individually. For each one:
- Show the category and occurrence count
- Show the specific findings that triggered it
- Show the proposed change as a diff or new content
- Ask: approve or reject?

### Step 8: Apply on Approval

- **Approved project-scoped changes:** apply the edit and commit
- **Approved global proposals:** write the proposal file to `vollgas/retro-proposals/`
- **Rejected drafts:** discard. The pattern stays in the retro log and will surface again if it recurs.

Update — not append — the "Actions taken" line in the retro log entry (written in Step 3) with a summary of what was approved.

## Common Mistakes

**Inventing categories instead of reusing existing ones**
- Always scan `retro-log.md` for existing categories before creating a new one. `accessibility/aria-labels` and `accessibility/missing-attributes` for the same class of issue breaks recurrence detection.

**Acting on single occurrences**
- One finding is not a pattern. Log it and move on. The threshold exists to filter noise.

**Vague drafts**
- "Consider adding accessibility checks" is not a draft. A draft is: "Add to `vollgas/refs/accessibility-contract.md` under the ARIA section: 'All interactive elements must have an aria-label or aria-labelledby attribute.'"

**Modifying Vollgas skills directly from a project**
- Global changes go through proposal files, not direct edits. The user applies them to the Vollgas repo separately.

## Red Flags

**Never:**
- Act on findings below the recurrence threshold
- Create a new category when an existing one fits
- Draft vague recommendations instead of specific edits
- Modify Vollgas skill files directly from a project repo
- Run automatically — always wait for explicit invocation

**Always:**
- Read the retro log before analyzing findings
- Reuse existing categories when possible
- Present each draft individually for approval
- Log all findings regardless of whether action is taken
- Update the retro log "Actions taken" after applying changes

## Integration

**Called by:**
- User, after `finishing-a-development-branch` (opt-in)

**Reads from:**
- `vollgas/review-findings/` — written by `review-gate`
- `vollgas/retro-log.md` — written by previous retro runs

**Writes to:**
- `vollgas/retro-log.md` — appends entry each run
- `vollgas/refs/`, `vollgas/reviewers/`, `CLAUDE.md` — project-scoped changes on approval
- `vollgas/retro-proposals/` — global proposals on approval
