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

## Workflow

Only `/new-task` and `/impl-task` are required. The intermediate commands (`/research-task`, `/design-task`, `/spec-task`) are optional and can be skipped — for example, you can go directly from `/new-task` to `/impl-task`. When intermediate outputs exist, later commands will use them automatically; when they don't, the commands will explore the codebase on their own.

### `/new-task`

Create a new task directory and populate it with a task description.

- **Writes:** `task.md` (required) — the task description, provided by the user.

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

### `/impl-task`

Implement all code changes, then build, test, format, and stage. Follows the implementation spec if available, otherwise falls back to the design or research for guidance.

- **Reads:** `task.md` (required), `research.md` (optional), `design.md` (optional), `impl-spec.md` (optional)
- **Writes:** source code changes, staged via `git add -A`.
