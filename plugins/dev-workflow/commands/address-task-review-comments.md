---
name: "mrp-address-task-review-comments"
description: "Address review feedback from user-review.txt, review-findings.md, and the task PR, interactively"
---

Load the `mrp-dev-task` skill. Store `{task_name}`, `{task_dir}`, and `{tasks_dir}`. Then load `{task_description}` (required), `{user_review}` (optional), and `{review_findings}` (optional) as described in the skill.

## Review sources

There are three possible sources of review feedback. Process each available source in turn, in this order: `{user_review}`, `{review_findings}`, then the task PR. Sources that are absent (no file on disk, or no PR with unresolved threads) are simply skipped — do not stop the command because one source is missing.

## File-based sources: `{user_review}` and `{review_findings}`

`{review_findings}` uses the canonical format:

```
# Round N [status]

* Finding 1: [status] ...
  <details>

* Finding 2: [status] ...
  <details>
```

- Round status is one of `NEW` or `COMPLETE`.
- Finding status is one of `NEW`, `ADDRESSED`, or `SKIPPED`.

`{user_review}` uses a relaxed format that is authored by hand. Any of the following variations are valid and must be accepted:

```
Round N:
* <finding 1>
* <finding 2>
```

- The round header may be `# Round N [status]`, `# Round N`, `Round N [status]`, or `Round N:` — with or without the leading `#` and with or without a `[status]` annotation. A round header with no status is treated as `[NEW]`.
- Findings may be plain bullets (`* <finding>`) with no numbering and no `[status]`. A finding with no status is treated as `NEW`. Indented continuation lines belong to the preceding finding.
- When updating a round or finding's status in `{user_review}`, rewrite it in place to include an explicit `[COMPLETE]`, `[ADDRESSED]`, or `[SKIPPED]` annotation. Preserve the rest of the original text (header style, finding wording, indentation) as-is.

For each file-based source that exists, identify every round that is **not** marked `COMPLETE` (treating rounds with no status annotation as `NEW`). Process those rounds in ascending order. Within each round, process every finding that is currently `NEW` (treating findings with no status annotation as `NEW`), in the order it appears. Findings already marked `ADDRESSED` or `SKIPPED` in a prior pass are left alone.

### Per-finding flow

For each `NEW` finding:

1. **Display** the finding to the user with its round, number, status, summary, and any details.
2. **Examine the code** pointed to by the finding. Line numbers in the finding may not match the current file if the code has since changed — use the file path and context to locate the right code.
3. **Propose fix options**, explaining what the finding is asking for. Present two or more distinct options for how the fix could be implemented, each with a short description of the approach and its trade-offs. Clearly highlight which option you recommend and why.
4. **Get user approval** using whichever tool is best suited for asking the user a question with selectable choices. Offer one choice per proposed fix option, plus the following:
   - **Modify** — The user provides additional guidance; incorporate it and re-propose (loop back to step 3).
   - **Skip** — Move on without making changes.
5. **Implement** the fix if approved. Make targeted, minimal edits.
6. **Update the finding's status** in the same source file it came from:
   - `ADDRESSED` if the fix was implemented.
   - `SKIPPED` if the user chose to skip.

### Round completion

After every finding in a round has been processed (i.e., no more findings remain `NEW`), mark the round as `COMPLETE` in the source file, even if one or more findings were `SKIPPED`.

## PR review threads

After file-based sources have been processed, handle unresolved review threads on the task's GitHub PR.

### Discovery

1. **Check `{task_description}`** — Look for GitHub PR URLs (e.g., `https://github.com/{owner}/{repo}/pull/{number}`) or PR number references (e.g., `#123`). Extract the PR number.
2. **Branch fallback** — If no PR is found in the task description, query the GitHub API for open PRs on the task branch (`markp/{task_name}`). If exactly one PR is found, use it.
3. **Skip if absent** — If no PR can be identified by either method, skip this source and continue with the commit/push step.

Store the PR number as `{pr_number}` and the repo owner/name as `{owner}` and `{repo}`.

### Fetching unresolved threads

Use GraphQL via the `gh` CLI to fetch review threads with their resolution status, author, body, file path, line, and timestamps. Filter to only threads where `isResolved` is `false`.

- If the `gh` CLI is unavailable or unauthenticated, inform the user with clear installation/auth guidance and skip this source.
- If there are no review threads or all threads are resolved, inform the user and skip this source.

### Per-thread flow

For each unresolved thread, in order:

1. **Display the thread** with a clear header (e.g., "Thread 1 of N"), the file path and line from the first comment, and every comment in chronological order showing author (`@login`), body, and timestamp.
2. **Examine the code** around the referenced line. Line numbers may be stale — use the file path and comment context to locate the relevant code.
3. **Propose a fix**, explaining what the reviewer is asking for, what change you propose, and your reasoning.
4. **Get user approval** via `ask-user` with three options:
   - **Approve** — Implement the proposed fix.
   - **Modify** — The user provides additional guidance; incorporate it and re-propose (loop back to step 3).
   - **Skip** — Move on without making changes.
5. **Implement** the fix if approved. Make targeted, minimal edits.

PR thread resolution state on GitHub is **not** modified by this command; the PR remains the source of truth for its own threads.

## Commit and push

Once all available review sources have been processed:

1. If **no code changes** were made across all sources (everything was skipped, or no actionable findings existed), inform the user and **stop**.
2. Otherwise, load the `mrp-commit-message` skill and use it to generate a commit message and offer to commit and push the changes.

> **CRITICAL:** Do NOT push changes without explicit user approval. Always ask the user for confirmation before pushing.
