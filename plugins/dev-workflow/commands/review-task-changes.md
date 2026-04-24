---
name: "mrp-review-task-changes"
description: "Spawn mrp-reviewer sub-agents to review task branch changes, dedupe, and record findings as a new round"
---

Load the `mrp-dev-task` skill. Store `{task_name}`, `{task_dir}`, and `{tasks_dir}`. Then load `{task_description}` (required) and `{review_findings}` (optional) as described in the skill.

You are the **coordinator** for this review. You do not review code yourself — your responsibilities are dispatching reviewers, deduplicating and ranking their findings, and recording the result in `{review_findings}`.

## Step 1: Dispatch reviewers

Launch at least **two** `mrp-reviewer` sub-agents concurrently in a single batch of parallel tool calls (check your available tools for the one ending in `mrp-reviewer`). The reviewer pool must always consist of exactly one **holistic reviewer** plus one or more **code-focused reviewers**.

### Holistic reviewer — always dispatched, always full-diff

Always dispatch exactly one reviewer with the entire set of changes as scope — never a slice. This reviewer's focus areas, all of which require a whole-change view, are:

- Gaps between what `{task_description}` requires and what was actually implemented.
- Verification plan and testing coverage — whether `{verification_plan}` / `{verification_report}` cover the changes adequately, and whether added or changed tests catch the failure modes introduced by this change.
- Refactoring and simplification opportunities spanning the change as a whole.

### Code-focused reviewers — one or more

Dispatch one or more additional reviewers. Scale the count up with the size and complexity of the change, and partition the diff across them:

- **Larger changes:** assign each code-focused reviewer a distinct slice of the diff (e.g., a subset of files, modules, or logical areas) so they collectively cover the full diff without re-examining the same code.
- **Smaller changes:** a single code-focused reviewer examines the entire set of changes.

Every code-focused reviewer's base focus areas are:

- Bugs, correctness issues, edge cases, and error-handling gaps.
- Code quality issues and adherence to surrounding conventions.

### Additional focus areas — assign to specific code-focused reviewers when warranted

When the nature of the change calls for it, assign one of the following focus areas to a specific code-focused reviewer in addition to the base focus areas. Omit any focus area that is not relevant to the change; do not pad reviewers with focus areas that don't apply.

- **Scaling, performance, and concurrency** — concurrency hazards, hot paths, resource usage, scalability limits.
- **Security** — authz/authn, input validation, injection, secrets handling, unsafe deserialization, supply-chain risks.

### Common constraints — every reviewer

Each reviewer's scope statement must also make clear that:

- The review target is the diff of the task branch (`markp/{task_name}`) against the base branch (main or master), including any staged or unstaged changes — narrowed to the slice assigned to this reviewer when the diff has been partitioned.
- The only task artifacts they may consult are `task.md`, `verification-plan.md`, and `verification-report.md`. All other task artifacts are off-limits.
- They must return severity-ranked findings (`high` / `medium` / `low`) with file paths, line numbers, and clear rationale.
- They must not make any code changes, must not write into any task artifact, and must not launch further sub-agents.

Wait for all reviewers to return their reports before moving on.

## Step 2: Determine the new round number

If `{review_findings}` does not exist, the new round number is **1**.

Otherwise, find the highest existing round number `M` in `{review_findings}` and set the new round number to `M + 1`.

## Step 3: Merge and deduplicate findings

1. **Merge** the findings from all reviewer reports into a single pool.
2. **Dedupe within the pool** — combine findings that describe the same underlying issue (same file and logical concern). Keep the clearest wording and the highest severity among duplicates.
3. **Exclude previously-skipped findings** — if `{review_findings}` exists, drop any pooled finding that matches a finding marked `SKIPPED` in any prior round. A match is based on the underlying issue, not literal wording.
4. **Keep re-flagged findings that were previously addressed** — if a pooled finding matches one marked `ADDRESSED` in a prior round but the current reviewers still flag it, **include** it in the new round.

## Step 4: Order by severity

Sort the remaining findings by severity in descending order (`high` → `medium` → `low`). Preserve a stable order within each severity tier.

## Step 5: Write the new round to `{review_findings}`

Append a new round section to `{task_dir}/{review_findings}` (create the file with an appropriate title if it does not yet exist). The section must follow this format:

```
# Round N [NEW]

## Finding 1: [NEW]

  **Severity:** <high|medium|low>

  <one-line summary>

  **Details:** <rationale, suggested direction>

  **Files:** <files and line numbers>

## Finding 2: [NEW]

  **Severity:** <high|medium|low>

  <one-line summary>

  **Details:** <rationale, suggested direction>

  **Files:** <files and line numbers>
```

Leave a blank line between each finding and between each section within a finding, exactly as shown above.

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
