---
name: "[MRP] New Dev Task"
description: "Create a new dev task directory and populate its task.md"
---

Load the `mrp-dev-task` skill and follow these steps:

## Step 1: Determine the task name

Determine the task name using the first available source:
1. Check the `MRP_TASK` environment variable. If it is set and non-empty, use it.
2. Otherwise, ask the user for the task name.

Store the resolved name as `{task_name}`. Use `{task_name}` everywhere below as a placeholder — substitute the actual value.

## Step 2: Resolve the tasks directory

Read the `MRP_TASKS_DIR` environment variable. If it is not set or empty, **stop and ask the user** to set it. Store it as `{tasks_dir}`.

## Step 3: Create the task directory

Run:
```
mkdir -p {tasks_dir}/{task_name}
```

## Step 4: Populate task.md

The target file is `{tasks_dir}/{task_name}/task.md`.

First, check whether the file already exists:
```
test -f {tasks_dir}/{task_name}/task.md && echo EXISTS || echo NEW
```

### If the file already exists

Ask the user whether they want to:
- **Overwrite** the existing task.md
- **Append** to the existing task.md

Then proceed based on their choice using the write methods below.

### Writing the file contents

Use the `ask-user` tool to ask the user how they'd like to provide the description, with these two suggested responses:
- "Edit the file myself."
- "Supply the description here."

If the user chooses **"Edit the file myself"**: print the following command so they can copy/paste it into another terminal:
```
vim {tasks_dir}/{task_name}/task.md
```
Then wait for the user to confirm they are done editing before proceeding to Step 5.

If the user chooses **"Supply the description here"**: wait for the user's next message and treat its contents as the description. Write it to the task.md file. Do not use the `ask-user` tool, just wait for the user's next input and treat that as the description.

## Step 5: Confirm

After the file is written, print the contents of the task.md file and confirm the task was created successfully.

