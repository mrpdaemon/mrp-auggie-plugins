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

