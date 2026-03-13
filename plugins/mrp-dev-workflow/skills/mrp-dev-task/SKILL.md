---
name: mrp-dev-task
description: Manage files for Mark's personal development task workflow. Use when asked to 'write to task directory', 'save to task directory', 'add to task directory', or any request involving reading/writing files in the current dev task directory.
---

# MRP Dev Task File Management

Manage files organized by development task in Mark's personal task workflow.

## Task directory structure

All task files live under:
```
/home/augment/.augment/tasks/
```

Each task has its own subdirectory. For example, a task named `foo` stores files in:
```
/home/augment/.augment/tasks/foo/
```

## Determining the current task

1. Read the `MRP_TASK` environment variable to get the current task name.
2. If `MRP_TASK` is not set or is empty, **ask the user** for the name of the current task.

## Writing files

**Do NOT use the `save-file` tool** to write files into the task directory. The task directory is in the user's home directory (`/home/augment/.augment/tasks/`), not under the repository root, so `save-file` will not work.

Instead, use the `cat` command via `launch-process` to write file contents. For example:

```bash
mkdir -p /home/augment/.augment/tasks/<task_name>
cat > /home/augment/.augment/tasks/<task_name>/filename.ext << 'AUGEOF'
file contents here
AUGEOF
```

## Task description

The description for a task lives in the `task.md` file in the task directory.
For example, the description for the `foo` task lives in:
```
/home/augment/.augment/tasks/foo/task.md
```

## Workflow

1. Determine the current task name from `MRP_TASK` or by asking the user.
2. Ensure the task directory exists (`mkdir -p /home/augment/.augment/tasks/<task_name>/`).
3. Read or write files in that directory as requested.

