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

## Step 2: Produce a single well-messaged commit

The goal of this step is to end up with exactly one commit on the task branch whose message conforms to the format defined by the `mrp-commit-message` skill. Choose one of the following based on the current branch state:

- **Multiple commits above the base** — Load the `mrp-commit-message` skill to generate a commit message using the **"first commit" pattern** (Problem/Solution/Testing), providing the skill with the full diff of changes on the branch relative to the base. Soft-reset to the merge base and create a single new commit with the generated message.
- **Single commit above the base, message already conforms** — Skip the step entirely. The existing commit and message are preserved as-is.
- **Single commit above the base, message does not conform** — Load the `mrp-commit-message` skill to generate a conforming message (same "first commit" pattern as above) and amend the existing commit to use it. Do not change the commit's contents.

Record whether this step rewrote history (either a squash or an amend) so later steps can branch on it.

## Step 3: Clean stale SHA references in iterations.md

Only applicable when Step 2 rewrote history. If `{iterations}` exists and contains `**Git state:** \`<hex_sha>\`` lines, remove those lines from the file — the SHAs are stale after the rewrite. If any lines were removed, amend the commit from Step 2 to include the updated file. If Step 2 was skipped, leave `iterations.md` untouched.

## Step 4: Push and create draft PR

1. **Push** the branch to the remote and set up upstream tracking. Use a force push only if Step 2 rewrote history; otherwise a regular push is sufficient.
2. **Create a draft PR** via the GitHub API. Use the first line of the final commit message as the PR title and the remainder as the PR body. Before submitting, adapt the body for a public PR-reviewer audience:
   - **Reflow paragraphs** — Join hard-wrapped lines within each prose paragraph (which the `mrp-commit-message` skill produces at a ~72–80 character column) into a single long line so paragraphs render as flowing paragraphs rather than fixed-width column breaks. Preserve blank lines between paragraphs/sections, section headers, list items, code blocks, and other intentional line breaks.
   - **Strip references to local task artifacts** — Remove any mentions of files that live only under `{task_dir}` and are not visible to PR reviewers (e.g. `task.md`, `research-report.md`, `design-spec.md`, `implementation-spec.md`, `verification-plan.md`, `verification-report.md`, `iterations.md`, `review-findings.md`, `user-review.txt`). Where such a mention provided context, replace it with a self-contained description of the underlying information rather than a pointer to the file.
   - **Inline verification details** — Replace references to test case identifiers from the verification plan/report (e.g. `HP-1`, `EC-3`) with a concise description of what was actually tested. Reviewers should be able to understand the testing performed without access to the verification artifacts.

   Aside from these adaptations, the PR body's content should remain faithful to the commit message body.
3. **Record the PR** — Append `PR: #<number>` to `{task_dir}/task.md`.

## Step 5: Confirm

Report to the user which path Step 2 took (squashed N commits, reworded the sole commit's message, or left the sole conforming commit untouched), that the branch was pushed, the draft PR URL, and that the PR number was recorded in task.md.
