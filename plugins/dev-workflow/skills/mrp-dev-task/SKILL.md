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

Before writing any file, ensure the task directory exists by running `mkdir -p "$MRP_TASKS_DIR/<task_name>"` via `launch-process`.

## Variables to store

When a command loads this skill, it will instruct you which variables to resolve and store. Use the stored values as placeholders everywhere they appear in the command — substitute the actual values.

### Directory and name variables

- `{tasks_dir}` — The value of the `MRP_TASKS_DIR` environment variable. If it is not set or empty, **stop and ask the user** to set it.
- `{task_name}` — The current task name. Determine it using the first available source:
  1. Check the `MRP_TASK` environment variable. If it is set and non-empty, use it.
  2. Otherwise, **ask the user** for the task name.
- `{task_dir}` — The full path to the current task's directory: `{tasks_dir}/{task_name}`.

### Task file variables

Each task directory can contain the following files. When a command asks you to load one of these, follow the procedure for its type (**required** or **optional**).

| File | Variable | Type | Description |
|------|----------|------|-------------|
| `task.md` | `{task_description}` | required | The task description — what needs to be accomplished. |
| `research.md` | `{research}` | optional | Codebase research report with context and findings. |
| `design.md` | `{design}` | optional | High-level design document with architecture decisions. |
| `impl-spec.md` | `{impl_spec}` | optional | Detailed implementation spec with file-by-file changes. |

**Loading a required file:** Check that the task directory and the file both exist and are non-empty:
```
test -d {task_dir} && test -s {task_dir}/<file> && echo OK || echo MISSING
```
If the result is `MISSING`, tell the user:
> The task directory or task description does not exist (or is empty). Please run the `dev-workflow:new-task` command first.

Then **stop** — do not continue. If `OK`, read the file with `cat` and store its contents in the corresponding variable.

**Loading an optional file:** Check whether the file exists:
```
test -f {task_dir}/<file> && echo EXISTS || echo NONE
```
If `EXISTS`, read it with `cat` and store its contents in the corresponding variable. If `NONE`, note that the file is not available and continue.

