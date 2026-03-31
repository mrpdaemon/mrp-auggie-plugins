---
name: "[MRP] New Dev Task"
description: "Create a new dev task directory and populate its task.md"
---

Load the `mrp-dev-task` skill and store `{task_name}`, `{task_dir}`, and `{tasks_dir}` as described in the skill. Then follow these steps:

## Step 1: Create the task directory

Run:
```
mkdir -p {task_dir}
```

## Step 2: Populate task.md

The target file is `{task_dir}/task.md`.

First, check whether the file already exists:
```
test -f {task_dir}/task.md && echo EXISTS || echo NEW
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
vim {task_dir}/task.md
```
Then wait for the user to confirm they are done editing before proceeding to Step 3.

If the user chooses **"Supply the description here"**: wait for the user's next message and treat its contents as the description. Write it to the task.md file. Do not use the `ask-user` tool, just wait for the user's next input and treat that as the description.

## Step 3: Confirm

After the file is written, print the contents of the task.md file and confirm the task was created successfully.

