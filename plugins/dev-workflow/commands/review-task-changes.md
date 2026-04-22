---
name: "mrp-review-task-changes"
description: "Spawn mrp-reviewer sub-agents to review task branch changes, dedupe, and record findings as a new round"
---

Load the `mrp-dev-task` skill. Store `{task_name}`, `{task_dir}`, and `{tasks_dir}`. Then load `{task_description}` (required) and `{review_findings}` (optional) as described in the skill.

You are the **coordinator** for this review. You do not review code yourself — your responsibilities are dispatching reviewers, deduplicating and ranking their findings, and recording the result in `{review_findings}`.

## Step 1: Dispatch reviewers

Launch **5** `mrp-reviewer` sub-agents concurrently in a single batch of parallel tool calls (check your available tools for the one ending in `mrp-reviewer`). Each reviewer reviews the code changes on the task branch independently.

Each reviewer's scope statement should make clear that:

- The review target is the diff of the task branch (`markp/{task_name}`) against the base branch (main or master), including any staged or unstaged changes.
- The only task artifacts they may consult are `task.md`, `verification-plan.md`, and `verification-report.md`. All other task artifacts are off-limits.
- They must return severity-ranked findings (`high` / `medium` / `low`) with file paths, line numbers, and clear rationale.
- They must not make any code changes, must not write into any task artifact, and must not launch further sub-agents.

Wait for all 5 reviewers to return their reports before moving on.

## Step 2: Determine the new round number

If `{review_findings}` does not exist, the new round number is **1**.

Otherwise, find the highest existing round number `M` in `{review_findings}` and set the new round number to `M + 1`.

## Step 3: Merge and deduplicate findings

1. **Merge** the findings from all 5 reviewer reports into a single pool.
2. **Dedupe within the pool** — combine findings that describe the same underlying issue (same file and logical concern). Keep the clearest wording and the highest severity among duplicates.
3. **Exclude previously-skipped findings** — if `{review_findings}` exists, drop any pooled finding that matches a finding marked `SKIPPED` in any prior round. A match is based on the underlying issue, not literal wording.
4. **Keep re-flagged findings that were previously addressed** — if a pooled finding matches one marked `ADDRESSED` in a prior round but the current reviewers still flag it, **include** it in the new round.

## Step 4: Order by severity

Sort the remaining findings by severity in descending order (`high` → `medium` → `low`). Preserve a stable order within each severity tier.

## Step 5: Write the new round to `{review_findings}`

Append a new round section to `{task_dir}/{review_findings}` (create the file with an appropriate title if it does not yet exist). The section must follow this format:

```
# Round N [NEW]

* Finding 1: [NEW] <one-line summary — severity: high|medium|low>
  <details: file path and line, rationale, suggested direction>

* Finding 2: [NEW] <one-line summary — severity: high|medium|low>
  <details>
```

- Use the round number determined in Step 2.
- The round status starts as `NEW`.
- Every finding in the new round starts as `NEW`.
- Do **not** modify or rewrite prior rounds in the file — only append.

If, after merging and filtering, there are no findings to record, do not create an empty round. Instead, inform the user that no new findings were produced and **stop** without writing.

## Step 6: Confirm

Report to the user:

- The counts of raw findings collected, duplicates merged away, and findings excluded as previously `SKIPPED`.
- The final number of findings recorded in the new round, broken down by severity.
- The location of `{review_findings}` and a pointer to `/address-task-review-comments` for walking through the findings interactively.
