---
name: "mrp-address-task-pr-comments"
description: "Fetch unresolved PR review comments, walk through each interactively, propose and implement fixes"
---

Load the `mrp-dev-task` skill. Store `{task_name}`, `{task_dir}`, and `{tasks_dir}`. Then load `{task_description}` (required) as described in the skill.

## Step 1: Discover the PR

Find the GitHub PR associated with this task:

1. **Check `{task_description}`** — Look for GitHub PR URLs (e.g., `https://github.com/{owner}/{repo}/pull/{number}`) or PR number references (e.g., `#123`, `PR #123`). Extract the PR number.
2. **Branch fallback** — If no PR is found in the task description, query the GitHub API for open PRs on the task branch (`markp/{task_name}`). Extract the owner and repo from the git remote URL. If exactly one PR is found, use it.
3. **Ask the user** — If no PR is found by either method, ask the user for the PR URL or number. Stop if the user cannot provide one.

Store the PR number as `{pr_number}` and the repo owner/name as `{owner}` and `{repo}`.

## Step 2: Fetch unresolved review threads

Run a GraphQL query via the `gh` CLI to fetch all review threads and their resolution status. The query should retrieve `repository.pullRequest.reviewThreads` with `isResolved` and nested `comments` including `author.login`, `body`, `path`, `line`, and `createdAt`.

Parse the JSON response. Filter to only threads where `isResolved` is `false`. Store the list of unresolved threads.

### Edge cases

- If the `gh` CLI is not available or not authenticated, inform the user with a clear error message including installation/auth instructions and **stop**.
- If there are **no review threads at all**, inform the user and **stop**.
- If **all threads are resolved**, inform the user that all review comments have been addressed and **stop**.

## Step 3: Process each unresolved thread

For each unresolved thread, in order:

### 3a: Display the thread

Show the thread to the user with clear formatting:

- **Thread number and total** (e.g., "Thread 1 of 5")
- **File path and line number** from the first comment in the thread
- **Each comment** in chronological order, showing:
  - Author (`@login`)
  - Comment body
  - Timestamp

### 3b: Examine the code

Read the relevant source file and the surrounding context around the line referenced by the comment. Note that line numbers from the review may not match the current file if the code has changed since the review — use the file path and comment context to locate the relevant code.

### 3c: Propose a fix

Based on the review comment(s) and the current code, propose a fix. Explain:

- What the reviewer is asking for
- What change you propose
- Your reasoning

### 3d: Get user approval

Present three options to the user:

- **Approve** — Implement the proposed fix as-is
- **Modify** — The user provides additional guidance; incorporate their feedback and re-propose (loop back to 3c)
- **Skip** — Move on to the next thread without making changes

### 3e: Implement the fix

If the user approves, implement the fix. Make targeted, minimal edits.

## Step 4: Commit and push

After all unresolved threads have been processed:

1. If **no changes were made** (all threads were skipped), inform the user and **stop**.
2. If changes were made, load the `mrp-commit-message` skill and use it to generate a commit message and offer to commit and push the changes.

> **CRITICAL:** Do NOT push changes without explicit user approval. Always ask the user for confirmation before pushing.
