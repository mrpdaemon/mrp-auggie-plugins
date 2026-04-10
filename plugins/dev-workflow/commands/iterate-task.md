---
name: "mrp-iterate-task"
description: "Detect material changes since last iteration, record them in iterations.md, and update task artifacts"
---

Load the `mrp-dev-task` skill. Store `{task_name}`, `{task_dir}`, and `{tasks_dir}`. Then load `{task_description}` (required), `{design_spec}` (optional), `{implementation_spec}` (optional), `{verification_plan}` (optional), and `{iterations}` (optional) as described in the skill.

## Step 1: Gather change information

Collect all available information about what has been done since the last documented state:

### 1a: Git state

Infer what has changed by examining commits on the task branch (using main or master as the base) as well as any staged or unstaged changes in the working tree. Also note the current HEAD SHA for recording in the iteration metadata.

### 1b: Conversation context

Use the current conversation context to understand the intent and reasoning behind any changes that have been made. This includes decisions, trade-offs, and context that may not be captured in code or commit messages.

## Step 2: Analyze for material changes

Compare the gathered change information against the existing artifacts (`{design_spec}`, `{implementation_spec}`, `{verification_plan}`) to identify **material deltas** — changes that affect:

- Architecture or component design documented in `{design_spec}`
- File-level implementation details documented in `{implementation_spec}`
- Test coverage or methodology documented in `{verification_plan}`

Use AI judgment to distinguish material changes from noise (formatting, whitespace, trivial refactors). Material changes include modifications to logic, architecture, APIs, behavior, data models, or test coverage that diverge from what the artifacts describe.

### 2a: Check for already-documented iterations

If `{iterations}` exists, check the **Git state** recorded in the most recent iteration section. Compare it against the current git state to determine whether the changes since the last iteration are already documented. If the HEAD SHA matches and there are no uncommitted or staged changes beyond what was documented, there are no new material changes.

### 2b: Gate on material changes

If **no material changes** are detected, inform the user:

> No material changes detected since the last documented state. Nothing to record.

Then **stop** without writing anything.

## Step 3: Present changes and confirm

If material changes are found, present a summary to the user describing:

1. What material changes were detected
2. Which artifacts will be updated (`design.md`, `impl-spec.md`, `verification.md`)
3. The iteration number that will be created

Use the `ask-user` tool to ask the user whether to:
- **Proceed** — Record the iteration and update artifacts
- **Stop** — Do not write anything

If the user chooses **Stop**, end without writing.

## Step 4: Record the iteration

### 4a: Determine iteration number

If `{iterations}` exists, find the highest existing iteration number and increment by 1. If `{iterations}` does not exist, the new iteration number is **1**.

### 4b: Write iterations.md

If `{iterations}` does not exist, create it with a title and the new iteration section. If it exists, append the new iteration section.

Each iteration section should follow this format:

```markdown
## Iteration N

Brief summary of what changed and why.

### Change Title 1

Description of the material change — what the original artifact said vs. what was actually done and why.

### Change Title 2

Description of another material change.

### Artifacts Updated

- `design.md` — Updated section X to reflect Y
- `impl-spec.md` — Added new file Z, modified section W
- (list only artifacts that were actually updated)

**Git state:** `<HEAD SHA at time of iteration>`
```

## Step 5: Update affected artifacts

For each artifact that needs updating (`{design_spec}`, `{implementation_spec}`, `{verification_plan}`):

### 5a: Add or update cross-reference note

Add or update a cross-reference note at the top of the artifact (immediately after the first heading):

```markdown
> **Note:** This document was updated as part of **Iteration N** (see `iterations.md`) to reflect changes made during implementation.
```

If the artifact already has a cross-reference note from a previous iteration, update it to reference the latest iteration number.

### 5b: Surgically edit content

Surgically edit the artifact content to reflect the iteration's changes. Preserve the original document structure and content as much as possible — only modify the specific sections that are affected by the material changes.

Do **not** rewrite artifacts from scratch. Make targeted edits that bring the artifact in sync with what was actually done.

## Step 6: Confirm

Report to the user what was written and updated:

- Whether `iterations.md` was created or appended to
- Which artifacts were updated and what sections changed
- The iteration number that was recorded

Print a brief summary of the iteration.
