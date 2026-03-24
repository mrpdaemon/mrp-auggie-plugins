---
name: "[MRP] Recall Task"
description: "Read all files from the current dev task directory into context to prepare for working on the task"
---

Load the `mrp-dev-task` skill and follow these steps:

## Step 1: Determine the task name

Determine the task name using the first available source:
1. Check the `MRP_TASK` environment variable. If it is set and non-empty, use it.
2. Otherwise, ask the user for the task name.

Store the resolved name as `{task_name}`.

Read the `MRP_TASKS_DIR` environment variable. If it is not set or empty, **stop and ask the user** to set it.

Set `{task_dir}` to `$MRP_TASKS_DIR/{task_name}`.

## Step 2: Verify the task directory exists

Check that the task directory and description file exist and are non-empty:
```
test -d {task_dir} && test -s {task_dir}/task.md && echo OK || echo MISSING
```

If the result is `MISSING`, tell the user:
> The task directory or task description does not exist (or is empty). Please run the `dev-workflow:new-task` command first.

Then **stop** — do not continue.

## Step 3: Read all task files into context

List all files in the task directory:
```
find {task_dir} -type f | sort
```

Read **every** file found into context using the `view` tool. Read all files in parallel.

## Step 4: Summarize and wait for instructions

After reading all files, print a brief summary of what was loaded:
- The task name
- The list of files that were read
- A short summary of the task based on the task description (`task.md`)

Then tell the user you are ready and **wait for their instructions**. Do not take any further action until the user tells you what to do.

