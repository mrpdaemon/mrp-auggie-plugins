---
name: "[MRP] Implement Task"
description: "Implement all code changes for the current dev task, build, test, format, and stage"
model: "sonnet4.6"
---

Load the `mrp-dev-task` skill and follow these steps:

## Step 1: Determine the task name and read the description

Determine the task name using the first available source:
1. Check the `MRP_TASK` environment variable. If it is set and non-empty, use it.
2. Otherwise, ask the user for the task name.

Store the resolved name as `{task_name}`.

Set `{task_dir}` to `/home/augment/.augment/tasks/{task_name}`.

Check that the task directory and description file exist and are non-empty:
```
test -d {task_dir} && test -s {task_dir}/task.md && echo OK || echo MISSING
```

If the result is `MISSING`, tell the user:
> The task directory or task description does not exist (or is empty). Please run the `mrp:new-task` command first.

Then **stop** — do not continue.

If the result is `OK`, read the task description:
```
cat {task_dir}/task.md
```

Store the contents as `{task_description}`.

## Step 2: Read existing context files (if available)

Check for and read each of the following files if they exist:

```
test -f {task_dir}/research.md && echo EXISTS || echo NONE
test -f {task_dir}/design.md && echo EXISTS || echo NONE
test -f {task_dir}/impl-spec.md && echo EXISTS || echo NONE
```

For each file that exists, read it with `cat` and store the contents:
- `{task_dir}/research.md` → `{research}`
- `{task_dir}/design.md` → `{design}`
- `{task_dir}/impl-spec.md` → `{impl_spec}`

If none of the optional files are available, you will perform your own codebase exploration as needed during implementation. Use `codebase-retrieval` and `sub-agent-mrp-explorer` agents to explore the codebase.

## Step 3: Implement the code changes

Implement all the code changes required to complete the task described in `{task_description}`.

- If `{impl_spec}` is available, follow it closely — it contains detailed file-by-file change descriptions, new file specifications, ordering constraints, and a testing plan.
- If `{impl_spec}` is not available but `{design}` is, use the design document to guide your implementation approach.
- If only `{research}` is available, use it for codebase context.
- If none of the optional files are available, explore the codebase yourself using `codebase-retrieval` and `sub-agent-mrp-explorer` agents before implementing.

Use task management tools to break the implementation into trackable sub-tasks and mark them complete as you go.

## Step 4: Build and test

Once the implementation is complete, iterate until all of the following pass:

### 4a: Build affected targets

Identify all Bazel targets affected by the code changes and build them:
```
bazel build //path/to:target
```

Fix any build errors before proceeding.

### 4b: Run unit tests

Run all unit tests relevant to the changed code, including any newly added tests:
```
bazel test //path/to:test --test_output=all
```

Fix any test failures before proceeding.

### 4c: Run end-to-end tests (only if changed)

If a **new** end-to-end test was added, or an **existing** end-to-end test was modified as part of this task, run it and fix any failures.

If there was no actual change to an end-to-end test, do **NOT** attempt to run end-to-end tests.

### 4d: Iterate

Repeat steps 4a–4c until all builds succeed and all tests pass. Do **NOT** attempt any manual verification of the changes.

## Step 5: Format and stage

Once all builds and tests pass:

### 5a: Format the code

Run the formatter on all changed files:
```
bazel run //:format
```

### 5b: Stage all changes

Stage all changed and newly created files in git:
```
git add -A
```

Confirm to the user that all changes have been formatted and staged, and print a brief summary of what was implemented.

