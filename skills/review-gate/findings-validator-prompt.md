# Findings Validator Prompt Template

Dispatch this subagent after all reviewers return and before acting on any findings.

```
Task tool (general-purpose):
  description: "Validate reviewer findings"
  prompt: |
    You are validating reviewer findings against actual code. You did not
    participate in the review — you are a fresh pair of eyes.

    ## Reviewer Findings

    [PASTE the combined findings from all reviewers]

    ## Changed Files

    [LIST of files that were reviewed — the validator reads them directly]

    ## Reference Docs

    [LIST of reference doc paths the reviewers were given — validator reads
    them to understand what rules apply]

    ## Your Job

    For each finding:

    1. Read the actual code at the file and line range cited.
    2. Read the evidence the reviewer provided.
    3. Read any reference doc the finding relies on.
    4. Classify:

    - **VALID** — the issue exists in the code and the reference supports it.
    - **FALSE_POSITIVE** — the code is actually correct; the reviewer
      misread the code, misapplied the reference, or flagged something
      that is already handled.
    - **ALREADY_HANDLED** — the issue exists but is addressed elsewhere
      in the codebase (different file, different mechanism).

    5. Write your reasoning for each classification. Be specific — cite
       the line or reference that proves your classification.

    ## Rules

    - Do not trust the reviewer's description of the code. Read the code.
    - Do not trust your assumptions about what the code does. Read it.
    - If the finding lacks evidence (no code snippet, no reference), classify
      as FALSE_POSITIVE with reasoning "no evidence provided."
    - If you cannot determine whether the finding is valid after reading
      the code and references, classify as VALID — err on the side of
      fixing real issues.
    - Do not suggest fixes. Your job is classification only.

    ## Output Format

    For each finding:

    ```
    ### Finding N: {original one-line summary}
    - Reviewer: {who flagged it}
    - Classification: VALID | FALSE_POSITIVE | ALREADY_HANDLED
    - Reasoning: {specific evidence for your classification}
    ```

    Then a summary:

    ```
    ## Summary
    - Total findings: N
    - Valid: N (list by number)
    - False positives: N (list by number)
    - Already handled: N (list by number)
    ```
```
