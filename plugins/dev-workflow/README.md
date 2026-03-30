# dev-workflow

Augment plugin for Mark's personal development task workflow.

## Prerequisites

Set the `MRP_TASKS_DIR` environment variable to the directory where task files are stored:

```bash
export MRP_TASKS_DIR="$HOME/.augment/tasks"
```

All commands and skills in this plugin read `MRP_TASKS_DIR` at runtime to locate task directories. If the variable is not set, the agent will stop and ask you to set it.

## Environment variables

| Variable | Required | Description |
|---|---|---|
| `MRP_TASKS_DIR` | Yes | Root directory for task files |
| `MRP_TASK` | No | Current task name (if unset, the agent will prompt you) |

## Task directory

Each task has its own directory under `$MRP_TASKS_DIR/{task_name}/`. All files belonging to a task are stored in this directory. The workflow commands read from and write to the following files:

| File | Created by | Description |
|---|---|---|
| `task.md` | `/new-task` or `/linear-task` | Task description (required — all other commands depend on it) |
| `research.md` | `/research-task` | Codebase research report |
| `design.md` | `/design-task` | High-level design document |
| `impl-spec.md` | `/spec-task` | Detailed implementation spec |

## Workflow

Either `/new-task` or `/linear-task` is used to create a task, then `/impl-task` implements it. The intermediate commands (`/research-task`, `/design-task`, `/spec-task`) are optional and can be skipped — for example, you can go directly from `/new-task` to `/impl-task`. When intermediate outputs exist, later commands will use them automatically; when they don't, the commands will explore the codebase on their own.

### `/new-task`

Create a new task directory and populate it with a task description.

- **Writes:** `task.md` (required) — the task description, provided by the user.

### `/linear-task`

Create a new task directory from a Linear issue. Looks up the issue, derives a task name, and synthesizes the issue contents into a task description.

- **Writes:** `task.md` (required) — the task description, synthesized from the Linear issue.

### `/research-task`

Investigate the codebase and produce a research report with all the context needed for design and implementation.

- **Reads:** `task.md` (required)
- **Writes:** `research.md` — synthesized findings including relevant files, key abstractions, patterns, constraints, and open questions.

### `/design-task`

Collaborate interactively with the user on a high-level design. Walks through key design questions one at a time, then synthesizes the answers into a design document.

- **Reads:** `task.md` (required), `research.md` (optional)
- **Writes:** `design.md` — design decisions, chosen approach, components to modify, and open questions/risks.

### `/spec-task`

Collaborate interactively with the user on a detailed implementation spec. Explores the codebase to identify every file and function that needs to change, resolves ambiguities with the user, and defines a testing plan.

- **Reads:** `task.md` (required), `research.md` (optional), `design.md` (optional)
- **Writes:** `impl-spec.md` — file-by-file change descriptions, new file specifications, dependency ordering, and testing plan.

### `/recall-task`

Read all files from the current task directory into context to prepare for resuming work on a task. Lists and reads every file in the task directory, then prints a summary and waits for instructions.

- **Reads:** `task.md` (required), plus all other files in the task directory.

### `/impl-task`

Implement all code changes, then build, test, format, and stage. Follows the implementation spec if available, otherwise falls back to the design or research for guidance.

- **Reads:** `task.md` (required), `research.md` (optional), `design.md` (optional), `impl-spec.md` (optional)
- **Writes:** source code changes, staged via `git add -A`.
