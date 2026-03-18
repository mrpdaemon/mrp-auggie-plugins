---
name: mrp-dev-task
description: Manage files for Mark's personal development task workflow. Use when asked to 'write to task directory', 'save to task directory', 'add to task directory', or any request involving reading/writing files in the current dev task directory.
---

# MRP Dev Task File Management

Manage files organized by development task in Mark's personal task workflow.

## Task name guidance

- Task names must be **kebab-case** (e.g. `foo-bar-baz`)
- Task names are **all lowercase**
- Task names are **concise** — at most 4 words, often 2–3 words

## Task directory structure

All task files live under the directory specified by the `MRP_TASKS_DIR` environment variable:
```
$MRP_TASKS_DIR/
```

Each task has its own subdirectory. For example, a task named `foo` stores files in:
```
$MRP_TASKS_DIR/foo/
```

## Determining the current task

1. Read the `MRP_TASKS_DIR` environment variable to get the tasks root directory. If it is not set, **stop and ask the user** to set it.
2. Read the `MRP_TASK` environment variable to get the current task name.
3. If `MRP_TASK` is not set or is empty, **ask the user** for the name of the current task.

## Writing files

**Do NOT use the `save-file` tool** to write files into the task directory. The task directory is in the user's home directory (`$MRP_TASKS_DIR/`), not under the repository root, so `save-file` will not work.

Instead, use the `cat` command via `launch-process` to write file contents. For example:

```bash
mkdir -p "$MRP_TASKS_DIR/<task_name>"
cat > "$MRP_TASKS_DIR/<task_name>/filename.ext" << 'AUGEOF'
file contents here
AUGEOF
```

## Task description

The description for a task lives in the `task.md` file in the task directory.
For example, the description for the `foo` task lives in:
```
$MRP_TASKS_DIR/foo/task.md
```

## Workflow

1. Read the `MRP_TASKS_DIR` environment variable to get the tasks root directory.
2. Determine the current task name from `MRP_TASK` or by asking the user.
3. Ensure the task directory exists (`mkdir -p "$MRP_TASKS_DIR/<task_name>/"`).
4. Read or write files in that directory as requested.

