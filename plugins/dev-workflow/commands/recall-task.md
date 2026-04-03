---
name: "mrp-recall-task"
description: "Read all files from the current dev task directory into context to prepare for working on the task"
---

Load the `mrp-dev-task` skill. Store `{task_name}`, `{task_dir}`, and `{tasks_dir}`. Then load `{task_description}` (required) as described in the skill.

## Step 1: Read all task files into context

List all files in the task directory:
```
find {task_dir} -type f | sort
```

Read **every** file found into context using the `view` tool. Read all files in parallel.

## Step 2: Summarize and wait for instructions

After reading all files, print a brief summary of what was loaded:
- The task name
- The list of files that were read
- A short summary of the task based on the task description (`task.md`)

Then tell the user you are ready and **wait for their instructions**. Do not take any further action until the user tells you what to do.

