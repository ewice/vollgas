# Project Integration Guide

Vollgas provides a generic development pipeline. Projects extend it with domain-specific concerns — reviewers, reference docs, skills, and pipeline variants.

## How It Works

Vollgas provides the **process skeleton:**

```
brainstorm → spec → plan → implement (TDD) → review → finish
```

Projects provide **domain-specific extensions** in a `vollgas/` directory at the project root:

```
your-project/
├── vollgas/
│   ├── reviewers/       # Reviewer agent prompts
│   ├── refs/            # Reference docs for reviewers and implementers
│   └── skills/          # Project-specific skills
├── CLAUDE.md            # Pipeline variants and project conventions
└── ...
```

## Directory Structure

### `vollgas/reviewers/`

One file per reviewer. Each reviewer is auto-discovered by the review-gate skill and must follow the `reviewer-output-contract.md` format for findings.

Example reviewer file (`vollgas/reviewers/accessibility-reviewer.md`):

```markdown
# Accessibility Reviewer

## Context Files
- vollgas/refs/accessibility-contract.md

## Prompt
You are reviewing changes for accessibility compliance. Read the context
files above, then review the changed files against those rules.

Use the reviewer output contract format for all findings.
```

### `vollgas/refs/`

Reference documents injected as context for reviewers and implementers. These are the source of truth for domain-specific rules.

Examples:
- `accessibility-contract.md` — ARIA patterns, keyboard navigation, focus management
- `api-contract.md` — Public API conventions, naming, event contracts
- `test-patterns.md` — Shared test utilities and required test categories
- `reviewer-dispatch.md` — Maps change types to reviewers (see below)

### `vollgas/refs/reviewer-dispatch.md`

Maps change types to which reviewers should run and what context they receive:

```markdown
| Change type | Reviewers | Context files |
|---|---|---|
| New component | all reviewers | all refs |
| API change | api-reviewer, test-reviewer | api-contract.md, test-patterns.md |
| Bug fix | test-reviewer, code-quality-reviewer | test-patterns.md |
```

The review-gate skill reads this file to determine which reviewers to dispatch. If this file doesn't exist, review-gate falls back to change-type heuristics.

### `vollgas/skills/`

Project-specific skills that slot into the pipeline between vollgas stages. These handle domain-specific implementation phases that the generic pipeline can't cover.

Example: a component library might have:
- `/component-spec` — Generates a component specification from requirements
- `/scaffold` — Creates the file structure for a new component
- `/build-elements` — Guides element implementation with domain-specific patterns
- `/build-stories` — Creates Storybook stories for the component

### CLAUDE.md Pipeline Variants

CLAUDE.md defines pipeline variants for common task shapes in your project. This lets the agent pick the right level of ceremony for each task.

**Skill prefix:** Vollgas skills are invoked with `vollgas:` (e.g., `vollgas:brainstorming`). Project-local skills use `/` (e.g., `/component-spec`).

```markdown
## Development Pipelines

**Complex feature:**
vollgas:brainstorming → /component-spec → /scaffold →
vollgas:writing-plans → vollgas:subagent-driven-development →
vollgas:finishing-a-development-branch

**Simple addition:**
/component-spec → /scaffold → /build-elements →
vollgas:finishing-a-development-branch

**Bug fix:**
vollgas:systematic-debugging → /fix-bug →
vollgas:finishing-a-development-branch
```

## Recommended Hook Patterns

Configure hooks in `.claude/settings.json` to automate quality checks. These complement (not replace) the verification-before-completion skill.

### PostToolUse: Lint and format on file changes

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "fileMatcher": "src/**/*.{ts,tsx}",
        "command": "eslint --fix $CHANGED_FILE && prettier --write $CHANGED_FILE",
        "async": true,
        "statusMessage": "Linting and formatting..."
      }
    ]
  }
}
```

### Stop: Run tests on session exit

```json
{
  "hooks": {
    "Stop": [
      {
        "command": "npm run test:run",
        "async": true,
        "statusMessage": "Running tests..."
      }
    ]
  }
}
```

Adapt the commands and file patterns to your project's toolchain.

## Example: Component Library Setup

A Lit-based component library might have:

```
my-library/
├── vollgas/
│   ├── reviewers/
│   │   ├── accessibility-reviewer.md
│   │   ├── api-reviewer.md
│   │   ├── headless-reviewer.md
│   │   ├── lit-reviewer.md
│   │   ├── test-reviewer.md
│   │   └── code-quality-reviewer.md
│   ├── refs/
│   │   ├── accessibility-contract.md
│   │   ├── api-contract.md
│   │   ├── headless-contract.md
│   │   ├── lit-patterns.md
│   │   ├── test-patterns.md
│   │   └── reviewer-dispatch.md
│   └── skills/
│       ├── component-spec/
│       ├── scaffold/
│       ├── build-elements/
│       └── build-stories/
├── CLAUDE.md
├── .claude/settings.json
└── src/
```

The vollgas pipeline handles process (TDD, planning, review gates). The project's `vollgas/` directory handles domain (what makes a good component, how to test it, what to review for).
