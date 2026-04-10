---
name: mrp-dev-task
description: Manages development task artifact files and the git branch for the currently active development task. Use when asked to 'determine the current development task', 'write to task directory', 'list task files', 'determine changes done for the current development task' or any request involving reading/writing files to the task directory and determinining changes that have been done as part of the task.
---

We will be working on a specific development task. The name of the task we will be working on is specified by the `MRP_TASK` environment variable. If `MRP_TASK` is not set or is empty, **ask the user** for the name of the active development task. Store the name of the current task as {task_name}.

# Development Task File Management

Files representing artifacts related to the current development task are stored in a task-specific directory. The root path storing all task directories is specified by the `MRP_TASKS_DIR` environment variable. If this variable is not set or is empty, there is an issue with the user's configuration, ask the user to fix it by setting this variable. Store the value of the tasks directory as {tasks_dir}.

## Task Artifact Files

Artifacts related to the current development task are stored under the {tasks_dir}/{task_name} directory. Store this as {task_dir}. Each task directory may contain some of the following files. Refer to the contents of the appropriate file as needed to complete the user's requests.

| File | Variable | Type | Description |
|------|----------|------|-------------|
| `task.md` | `{task_description}` | required | The task description — what needs to be accomplished. |
| `research.md` | `{research_report}` | optional | Codebase research report with context and findings. |
| `design.md` | `{design_spec}` | optional | High-level design document with architecture decisions. |
| `impl-spec.md` | `{implementation_spec}` | optional | Detailed implementation spec with file-by-file changes. |
| `verification.md` | `{verification_plan}` | optional | Verification plan with testing methodology and test cases. |

# Development Task Git Branch

Changes related to the current development branch go in the `markp/{task_name}` branch. Store the name of this branch as {task_branch}. All work that has already been done for this task (if any) would be contained in the commits between the base branch (main or master) and {task_branch}, as well as staged changes in the git index (if any) or uncommitted changes (if any).

# Conventions

## Task name guidance

- Task names must be **kebab-case** (e.g. `foo-bar-baz`)
- Task names are **all lowercase**
- Task names are **concise** — at most 4 words, often 2–3 words
