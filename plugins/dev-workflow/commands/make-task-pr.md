---
name: "mrp-make-task-pr"
description: "Squash task branch commits, push, and create a draft GitHub PR"
---

Load the `mrp-dev-task` skill. Store `{task_name}`, `{task_dir}`, and `{tasks_dir}`. Then load `{task_description}` (required) and `{iterations}` (optional) as described in the skill.

## Step 1: Pre-flight checks

Before doing anything, verify all of the following. If any check fails, **stop** with a clear error message explaining what's wrong and how to fix it.

- **Clean working tree** — There must be no uncommitted or unstaged changes. The user should commit or stash before running this command.
- **Commits exist** — There must be at least one commit on `{task_branch}` above the base branch (main or master). Determine which base branch the repository uses.
- **No existing PR** — Parse the repository owner and name from the git remote URL, then check the GitHub API for an open PR targeting this branch. If one already exists, report its number and stop.

## Step 2: Squash commits

Squash all commits on the task branch into a single commit:

1. Load the `mrp-commit-message` skill to generate the commit message. This is a squash of all branch commits, so use the **"first commit" pattern** with Problem/Solution/Testing sections. Provide the skill with the full diff of changes on the branch relative to the base.
2. Soft-reset to the merge base and create a single new commit with the generated message.

## Step 3: Clean stale SHA references in iterations.md

If `{iterations}` exists and contains `**Git state:** \`<hex_sha>\`` lines, remove those lines from the file — the SHAs are stale after the squash. If any lines were removed, amend the squash commit to include the updated file.

## Step 4: Push and create draft PR

1. **Push** the branch to the remote with force (history was rewritten by the squash) and set up upstream tracking.
2. **Create a draft PR** via the GitHub API. Use the first line of the squash commit message as the PR title and the remainder as the PR body.
3. **Record the PR** — Append `PR: #<number>` to `{task_dir}/task.md`.

## Step 5: Confirm

Report to the user: how many commits were squashed, that the branch was pushed, the draft PR URL, and that the PR number was recorded in task.md.
