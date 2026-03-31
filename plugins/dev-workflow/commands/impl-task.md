---
name: "[MRP] Implement Task"
description: "Implement all code changes for the current dev task, build, test, format, and stage"
model: "sonnet4.6"
---

Load the `mrp-dev-task` skill. Store `{task_name}`, `{task_dir}`, and `{tasks_dir}`. Then load `{task_description}` (required), `{research}` (optional), `{design}` (optional), and `{impl_spec}` (optional) as described in the skill.

If none of the optional files are available, you will perform your own codebase exploration as needed during implementation. Use `codebase-retrieval` and `sub-agent-mrp-explorer` agents to explore the codebase.

## Step 1: Implement the code changes

Implement all the code changes required to complete the task described in `{task_description}`.

- If `{impl_spec}` is available, follow it closely — it contains detailed file-by-file change descriptions, new file specifications, ordering constraints, and a testing plan.
- If `{impl_spec}` is not available but `{design}` is, use the design document to guide your implementation approach.
- If only `{research}` is available, use it for codebase context.
- If none of the optional files are available, explore the codebase yourself using `codebase-retrieval` and `sub-agent-mrp-explorer` agents before implementing.

Use task management tools to break the implementation into trackable sub-tasks and mark them complete as you go.

## Step 2: Build and test

Once the implementation is complete, iterate until all of the following pass:

### 2a: Build affected targets

Identify all Bazel targets affected by the code changes and build them:
```
bazel build //path/to:target
```

Fix any build errors before proceeding.

### 2b: Run unit tests

Run all unit tests relevant to the changed code, including any newly added tests:
```
bazel test //path/to:test --test_output=all
```

Fix any test failures before proceeding.

### 2c: Run end-to-end tests (only if changed)

If a **new** end-to-end test was added, or an **existing** end-to-end test was modified as part of this task, run it and fix any failures.

If there was no actual change to an end-to-end test, do **NOT** attempt to run end-to-end tests.

### 2d: Iterate

Repeat steps 2a–2c until all builds succeed and all tests pass. Do **NOT** attempt any manual verification of the changes.

## Step 3: Format and stage

Once all builds and tests pass:

### 3a: Format the code

Run the formatter on all changed files:
```
bazel run //:format
```

### 3b: Stage all changes

Stage all changed and newly created files in git:
```
git add -A
```

Confirm to the user that all changes have been formatted and staged, and print a brief summary of what was implemented.

