---
name: mrp-new-task
description: Bootstraps a new development task — task directory and objectives-focused task.md, the task git branch, and active-task setup (environment variable and tmux window title). Use when asked to 'create a new dev task', 'start a new dev task', 'new dev task', or any equivalent request to begin a fresh development task.
---

Load the `mrp-dev-task` skill to learn the task naming conventions, the task directory layout, and the development task git branch pattern. This skill establishes the active task, so do not treat a missing or empty `MRP_TASK` as an error while running it.

# Goal

Bootstrap a new development task by:

1. Determining an objectives-only task description and a task name.
2. Creating the task directory and writing `task.md`.
3. Creating or checking out the task's git branch.
4. Marking the new task as the active task for downstream tooling.

# Step 1: Determine the task description

The task description must capture **objectives only** — what needs to be accomplished and any hard constraints provided by the user. It must NOT include:

- Codebase research notes, findings, or exploration summaries.
- Design alternatives, trade-offs, or recommended approaches.
- Implementation details or file-level plans.

Research, design, and implementation are handled by later steps in the task workflow, which do their own investigation. Including that material here pollutes the task description and biases the later steps.

Determine the description as follows:

- First, examine the current conversation context. If the user has already described an objective, problem to solve, feature to build, or bug to fix in this conversation, distill that into a concise objectives-focused description.
- If the conversation context does not contain enough information to write the description, ask the user for it.

Store the resulting description as `{task_description}`.

# Step 2: Determine the task name

Derive a candidate task name from `{task_description}` following the task name guidance in the `mrp-dev-task` skill (kebab-case, all lowercase, concise — at most 4 words, often 2–3).

Present the candidate name to the user and ask for confirmation, allowing them to alter it if they prefer something different.

Store the confirmed name as `{task_name}`. Set `{task_dir}` to `{tasks_dir}/{task_name}`.

# Step 3: Create the task directory

Ensure `{task_dir}` exists, creating it if necessary.

# Step 4: Write task.md

The target file is `{task_dir}/task.md`.

If the file already exists, ask the user whether to:

- **Overwrite** the existing file with the new description, or
- **Stop** and keep the existing file (end the skill immediately).

Otherwise, write `{task_description}` to `{task_dir}/task.md`.

# Step 5: Set up the task git branch

The task branch is `markp/{task_name}` (the value the `mrp-dev-task` skill calls `{task_branch}`). Determine the repository's main branch (typically `main` or `master`; honour the `MRP_MAIN_BRANCH_NAME` environment variable if set) and use it as the base.

- If `{task_branch}` already exists, check it out.
- Otherwise, create it from the base branch and check it out.

# Step 6: Mark the task as the active task

- Treat `{task_name}` as the active task for the remainder of this agent session — do not re-prompt the user for the task name later in the conversation.
- Instruct the user to set the `MRP_TASK` environment variable to `{task_name}` in their interactive shell so that subsequent shell tooling and other agent sessions pick up the same active task. The skill itself cannot modify the user's shell environment, so this step must be communicated to the user.
- If running under tmux (the `TMUX` environment variable is set), rename the current tmux window to `{task_name}`.

# Step 7: Confirm

Print the contents of `{task_dir}/task.md` and a short confirmation summary that includes:

- The task name.
- The task directory path.
- The git branch that was created or checked out.
- Whether the tmux window title was updated.
- The reminder to set `MRP_TASK` in the user's shell.
