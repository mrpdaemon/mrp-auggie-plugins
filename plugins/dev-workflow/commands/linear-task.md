---
name: "[MRP] Linear Task"
description: "Create a new dev task from a Linear issue"
---

Load the `mrp-dev-task` skill and follow these steps:

## Step 1: Determine the Linear issue

If the user provided a Linear issue ID as an argument (e.g. `AU-1234`), use it. Otherwise, use the `ask-user` tool to ask the user for the Linear issue ID.

Store the resolved issue ID as `{issue_id}`.

## Step 2: Read the Linear issue

Look up the Linear issue `{issue_id}` and read its contents — title, description, comments, labels, and any other relevant metadata.

Store the issue title as `{issue_title}` and the full issue contents as `{issue_contents}`.

## Step 3: Determine the task name

Based on the Linear issue title and description, come up with a task name following the task name guidance in the `mrp-dev-task` skill.

Use the `ask-user` tool to present the proposed task name to the user and ask for confirmation. Allow the user to alter the name if they prefer something different.

Store the confirmed name as `{task_name}`.

## Step 4: Resolve the tasks directory

Read the `MRP_TASKS_DIR` environment variable. If it is not set or empty, **stop and ask the user** to set it. Store it as `{tasks_dir}`.

## Step 5: Create the task directory

Run:
```
mkdir -p {tasks_dir}/{task_name}
```

## Step 6: Populate task.md

The target file is `{tasks_dir}/{task_name}/task.md`.

First, check whether the file already exists:
```
test -f {tasks_dir}/{task_name}/task.md && echo EXISTS || echo NEW
```

If the file already exists, use the `ask-user` tool to ask the user whether they want to:
- **Overwrite** the existing task.md
- **Stop** and keep the existing file

If the user chooses **Stop**, end command execution immediately.

### Writing the file contents

Synthesize the Linear issue contents (`{issue_contents}`) into a task description. The task description should:

1. **Summarize the goal** — What needs to be accomplished, distilled from the issue title and description.
2. **Include relevant context** — Any important details, constraints, or requirements from the issue description and comments.
3. **Reference the Linear issue** — Include the issue ID (e.g. `AU-1234`) so the commit message skill can pick it up.

Do **not** simply copy-paste the issue contents. Synthesize them into a clear, actionable task description written in the style of the other task.md files in the workflow.

Write the synthesized description to the task.md file using `cat >` via `launch-process`:

```bash
cat > {tasks_dir}/{task_name}/task.md << 'AUGEOF'
<synthesized task description>
AUGEOF
```

## Step 7: Confirm

After the file is written, print the contents of the task.md file and confirm the task was created successfully.

