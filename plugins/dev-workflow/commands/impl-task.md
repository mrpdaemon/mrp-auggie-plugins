---
name: "mrp-impl-task"
description: "Implement all code changes for the current dev task, build, test, format, and stage"
---

Load the `mrp-dev-task` skill. Store `{task_name}`, `{task_dir}`, and `{tasks_dir}`. Then load `{task_description}` (required), `{research_report}` (optional), `{design_spec}` (optional), and `{implementation_spec}` (optional) as described in the skill.

If none of the optional files are available, you will perform your own codebase exploration as needed during implementation. Use `codebase-retrieval` for quick lookups, and launch `mrp-explorer` sub-agents (check your available tools for the one ending in `mrp-explorer`) for deeper exploration.

## Step 1: Implement the code changes

Implement all the code changes required to complete the task described in `{task_description}`.

- If `{implementation_spec}` is available, follow it closely — it contains detailed file-by-file change descriptions, new file specifications, ordering constraints, and a testing plan.
- If `{implementation_spec}` is not available but `{design_spec}` is, use the design document to guide your implementation approach.
- If only `{research_report}` is available, use it for codebase context.
- If none of the optional files are available, explore the codebase yourself using `codebase-retrieval` for quick lookups, and launch `mrp-explorer` sub-agents (check your available tools for the one ending in `mrp-explorer`) for deeper exploration before implementing.

Use task management tools to break the implementation into trackable sub-tasks and mark them complete as you go.

## Step 2: Build and test

Once the implementation is complete, iterate until all of the following pass:

### 2a: Build affected targets

Identify all targets affected by the code changes and build them using the project's build system. Determine the appropriate build command by examining the project's configuration files (e.g., Makefile, package.json scripts, build.gradle, Cargo.toml, BUILD files, etc.).

Fix any build errors before proceeding.

### 2b: Run unit tests

Run all unit tests relevant to the changed code, including any newly added tests. Use the project's established test runner and ensure test output is visible so failures can be diagnosed.

Fix any test failures before proceeding.

### 2c: Run end-to-end tests (only if changed)

If a **new** end-to-end test was added, or an **existing** end-to-end test was modified as part of this task, run it and fix any failures.

If there was no actual change to an end-to-end test, do **NOT** attempt to run end-to-end tests.

### 2d: Iterate

Repeat steps 2a–2c until all builds succeed and all tests pass. Do **NOT** attempt any manual verification of the changes.

## Step 3: Format and stage

Once all builds and tests pass:

### 3a: Format the code

Run the project's code formatter on all changed files. Determine the appropriate format command by examining the project's configuration files (e.g., Makefile, package.json scripts, formatting config files, BUILD files, etc.).

### 3b: Stage all changes

Stage all changed and newly created files in git:
```
git add -A
```

Confirm to the user that all changes have been formatted and staged, and print a brief summary of what was implemented.

