---
name: "[MRP] Spec Task"
description: "Collaborate on a detailed implementation spec for the current dev task and save it as impl-spec.md"
---

Load the `mrp-dev-task` skill and follow these steps:

## Step 1: Determine the task name and read the description

Determine the task name using the first available source:
1. Check the `MRP_TASK` environment variable. If it is set and non-empty, use it.
2. Otherwise, ask the user for the task name.

Store the resolved name as `{task_name}`.

Read the `MRP_TASKS_DIR` environment variable. If it is not set or empty, **stop and ask the user** to set it.

Set `{task_dir}` to `$MRP_TASKS_DIR/{task_name}`.

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

## Step 2: Read existing research and design (if available)

Check whether a research report exists:
```
test -f {task_dir}/research.md && echo EXISTS || echo NONE
```

If the result is `EXISTS`, read it:
```
cat {task_dir}/research.md
```

Store the contents as `{research}`.

Check whether a design document exists:
```
test -f {task_dir}/design.md && echo EXISTS || echo NONE
```

If the result is `EXISTS`, read it:
```
cat {task_dir}/design.md
```

Store the contents as `{design}`.

If neither `{research}` nor `{design}` is available, you will perform your own codebase exploration as needed during the spec process. Use `codebase-retrieval` and `sub-agent-mrp-explorer` agents to explore the codebase to inform the implementation spec.

## Step 3: Check for existing implementation spec

Check whether an implementation spec already exists:
```
test -f {task_dir}/impl-spec.md && echo EXISTS || echo NEW
```

If the result is `EXISTS`, use the `ask-user` tool to ask the user whether they want to:
- **Regenerate** the spec from scratch
- **Stop** and keep the existing spec

If the user chooses **Stop**, end command execution immediately — do not continue.

## Step 4: Collaborate on the implementation spec

Your goal is to work **interactively** with the user to produce a detailed implementation spec. Do **NOT** write any code. The spec must be detailed enough that an implementation agent can write code based on it without doing any significant investigation.

### 4a: Explore the codebase and identify changes

Based on `{task_description}`, `{research}` (if available), and `{design}` (if available), identify every component, file, and module that needs to be created or modified. Use `codebase-retrieval` and `sub-agent-mrp-explorer` agents to:

- Locate the exact files and functions that need to change.
- Understand existing patterns, interfaces, and conventions in those files.
- Identify existing tests that cover the affected code.
- Identify existing end-to-end tests that are relevant to the task.

### 4b: Present the implementation plan and resolve uncertainties

Present the proposed set of changes to the user, organized by component/file. For each change, describe:

1. **What** needs to change (file path, function/class/module).
2. **How** it needs to change (detailed description of the modification).
3. **Why** (rationale linking back to the task requirements).

If there are any implementation decisions where you are **not very confident**, use the `ask-user` tool to ask the user before proceeding. Examples of things to ask about:
- Ambiguous requirements that could be interpreted multiple ways.
- Multiple valid implementation approaches with different trade-offs.
- Naming conventions or patterns you are unsure about.
- Whether a change should be feature-flagged.

Incorporate the user's feedback and iterate until the user is satisfied with the plan.

### 4c: Define testing requirements

For each component being changed, specify:

1. **Unit tests** — Which existing test files need to be modified and what new test cases are needed. If new test files are required, specify their paths and what they should cover.
2. **End-to-end tests** (if warranted) — If there is an existing end-to-end test that covers the affected functionality, describe how it should be updated. If the task introduces a new user-facing flow that warrants e2e coverage, describe the new e2e test.

Do **NOT** include manual verification steps.

## Step 5: Write the implementation spec

Once the user approves the plan, write the complete implementation spec to `{task_dir}/impl-spec.md` using `cat` via `launch-process`:

```bash
cat > {task_dir}/impl-spec.md << 'AUGEOF'
<spec contents here>
AUGEOF
```

The implementation spec should include:

1. **Summary** — A brief overview of what is being implemented and why.
2. **Files to modify** — For each file that needs to change:
   - File path
   - Description of the current state (relevant functions, classes, interfaces)
   - Detailed description of the required changes
   - Any new functions, classes, or types to add (with signatures and behavior)
3. **New files to create** — For each new file:
   - File path
   - Purpose
   - Contents outline (exports, classes, functions with signatures and behavior)
4. **Dependencies and ordering** — If changes must be made in a specific order (e.g., proto changes before service changes), document the order.
5. **Testing plan** — For each affected area:
   - Existing test files to modify (with specific new test cases to add)
   - New test files to create (with path and coverage description)
   - End-to-end test changes (if applicable)
6. **Design decisions** — Key implementation choices that were made during the collaboration, with rationale.

After writing, confirm to the user that the implementation spec has been saved and print a brief summary.

