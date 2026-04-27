---
name: "mrp-address-task-review-comments"
description: "Address review feedback from user-review.txt, review-findings.md, and the task PR, with selectable per-source processing modes"
---

Load the `mrp-dev-task` skill. Store `{task_name}`, `{task_dir}`, and `{tasks_dir}`. Then load `{task_description}` (required), `{user_review}` (optional), and `{review_findings}` (optional) as described in the skill.

## Review sources

There are three possible sources of review feedback. Process each available source in turn, in this order: `{user_review}`, `{review_findings}`, then the task PR. Sources that are absent (no file on disk, or no PR with unresolved threads or unaddressed comments) are simply skipped — do not stop the command because one source is missing.

## Processing modes

For every source that has at least one unaddressed item, **select a processing mode before walking through any items**. First enumerate the unaddressed items in the source with stable identifiers — round + finding number for `{review_findings}`; round + bullet position for `{user_review}`; `Comment N` / `Thread N` for the PR — and present a one-line summary per item. Then ask the user to pick one of:

- **Interactive** — Walk through every unaddressed item one by one with the user, using the source's interactive per-item flow below.
- **Autonomous** — Process every unaddressed item without per-item user interaction, using the source's autonomous per-item flow. Only pause for user approval when an item meets the **significant impact** criteria below.
- **Specific items** — Ask the user which item(s) to address, accepting any unambiguous reference (e.g., a list of identifiers, a range, "all unaddressed in round 2"). Then, in a follow-up question, ask whether to process the selected subset **interactively** or **autonomously**, and proceed accordingly. Items not in the selected subset are left untouched and their status is not modified.

The selected mode applies to every item processed for that source in this invocation. For the PR, a single mode selection covers both top-level comments and unresolved review threads.

### Significant impact criteria

When processing autonomously — whether **Autonomous** mode or **Specific items → autonomous** — pause and ask the user for approval (offering **Approve** / **Modify** / **Skip**, and surfacing the proposed fix together with the matching criterion) before implementing any fix that meets one or more of the following:

- The fix would change a documented design decision recorded in `task.md`, `design-spec.md`, or `implementation-spec.md`, or otherwise contradicts an existing task artifact.
- The fix changes a public API, a function/method signature with multiple call sites, a data schema, or any other contract that ripples beyond the immediate file.
- The fix has a large change footprint — many files touched, large sections rewritten or deleted, or substantial new modules introduced.
- The fix adds, removes, or upgrades a dependency.
- Multiple distinct fix approaches with materially different trade-offs exist and none is clearly preferred.
- The fix would alter behavior beyond what the item explicitly asks for.

When in doubt, prefer to pause and ask.

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

For each file-based source that exists, identify every round that is **not** marked `COMPLETE` (treating rounds with no status annotation as `NEW`). Process those rounds in ascending order. Within each round, the candidate items are every finding that is currently `NEW` (treating findings with no status annotation as `NEW`), in the order it appears. Findings already marked `ADDRESSED` or `SKIPPED` in a prior pass are left alone.

### Mode selection

Before processing any findings in this source, perform the **mode selection** described in [Processing modes](#processing-modes), enumerating the unaddressed findings across all `NEW` rounds. The selected mode (and, for **Specific items**, the chosen subset and sub-mode) determines which per-finding flow below to use.

### Per-finding flow (interactive)

For each `NEW` finding in the chosen subset:

1. **Display** the finding to the user with its round, identifier, status, summary, and any details.
2. **Examine the code** pointed to by the finding. Line numbers in the finding may not match the current file if the code has since changed — use the file path and context to locate the right code.
3. **Propose fix options**, explaining what the finding is asking for. Present two or more distinct options for how the fix could be implemented, each with a short description of the approach and its trade-offs. Clearly highlight which option you recommend and why.
4. **Get user approval**. Offer one choice per proposed fix option, plus:
   - **Modify** — The user provides additional guidance; incorporate it and re-propose (loop back to step 3).
   - **Skip** — Move on without making changes.
5. **Implement** the fix if approved. Make targeted, minimal edits.
6. **Update the finding's status** in the same source file it came from:
   - `ADDRESSED` if the fix was implemented.
   - `SKIPPED` if the user chose to skip.

### Per-finding flow (autonomous)

For each `NEW` finding in the chosen subset, in order:

1. **Examine the code** pointed to by the finding (locate the right code even when line numbers are stale).
2. **Decide on the best fix**, briefly noting the reasoning. If any **significant impact** criterion applies, fall back to the interactive flow's "Propose fix options" + "Get user approval" steps for this finding only; otherwise implement the fix directly with targeted, minimal edits.
3. **Update the finding's status** in the source file: `ADDRESSED` if implemented, `SKIPPED` if the user explicitly chose to skip during a significant-impact pause.
4. After all findings in the subset have been processed, **summarize** what was done — one short line per finding indicating addressed / paused-approved / paused-modified / paused-skipped.

### Round completion

A round is marked `COMPLETE` in the source file only when no findings in it remain `NEW` (every finding is `ADDRESSED` or `SKIPPED`). In **Specific items** mode the user may intentionally leave some findings unaddressed; in that case the round stays in its prior state and may be revisited in a future invocation.

## PR review comments and threads

After file-based sources have been processed, handle feedback on the task's GitHub PR. There are two kinds of PR feedback to consider:

- **Review threads** — inline comments tied to specific lines of code. Threads carry a resolution status (`isResolved`), which is used as a proxy for whether the feedback still needs attention. Only unresolved threads are in scope.
- **PR comments** — top-level comments on the PR conversation (not part of a review thread). These do **not** carry a resolution status, so each comment must be individually assessed against the current code to determine whether its feedback has already been addressed.

### Discovery

1. **Check `{task_description}`** — Look for GitHub PR URLs (e.g., `https://github.com/{owner}/{repo}/pull/{number}`) or PR number references (e.g., `#123`). Extract the PR number.
2. **Branch fallback** — If no PR is found in the task description, query the GitHub API for open PRs on the task branch (`markp/{task_name}`). If exactly one PR is found, use it.
3. **Skip if absent** — If no PR can be identified by either method, skip this source and continue with the commit/push step.

Store the PR number as `{pr_number}` and the repo owner/name as `{owner}` and `{repo}`.

### Fetching feedback

Use the `gh` CLI to fetch both streams for the PR:

- Every review thread with its resolution status, file path, line, and the chronological list of comments it contains (author, body, timestamp).
- Every top-level PR conversation comment with author, body, and timestamp.

Filter review threads to those where `isResolved` is `false`. PR comments are **not** pre-filtered by any resolution proxy; each is assessed individually in the per-comment flow below.

- If the `gh` CLI is unavailable or unauthenticated, inform the user with clear installation/auth guidance and skip this source.
- If there are no unresolved threads **and** no PR comments, inform the user and skip this source.

Treat the PR as a **single source** for mode selection: one mode applies to both top-level comments and unresolved review threads. Within the chosen subset, process all PR comments first, then all unresolved threads.

### Mode selection

Before processing any PR feedback, perform the **mode selection** described in [Processing modes](#processing-modes), enumerating both PR comments (as `Comment N`) and unresolved review threads (as `Thread N`). The selected mode (and, for **Specific items**, the chosen subset and sub-mode) determines which per-item flows below to use.

### Per-comment flow (interactive)

For each PR comment in the chosen subset, in chronological order:

1. **Display the comment** with a clear header (e.g., "Comment 1 of N"), showing author (`@login`), body, and timestamp.
2. **Assess whether the comment is already addressed.** Because PR comments have no resolution status, examine the current state of the code (and any other relevant task artifacts) in light of what the comment is asking for, and decide whether the requested change — or its spirit — is already reflected in the branch. Present the assessment and the evidence used.
3. **Branch on the assessment:**
   - If the comment is **confidently already addressed**, ask the user to confirm with options **Confirm (no change)** / **Fix anyway** / **Skip**. Confirming or skipping moves on without changes; choosing "Fix anyway" falls through to step 4.
   - Otherwise, continue to step 4.
4. **Propose a fix**, explaining what the commenter is asking for, what change you propose, and your reasoning.
5. **Get user approval** with three options:
   - **Approve** — Implement the proposed fix.
   - **Modify** — The user provides additional guidance; incorporate it and re-propose (loop back to step 4).
   - **Skip** — Move on without making changes.
6. **Implement** the fix if approved. Make targeted, minimal edits.

### Per-thread flow (interactive)

For each unresolved thread in the chosen subset, in order:

1. **Display the thread** with a clear header (e.g., "Thread 1 of N"), the file path and line from the first comment, and every comment in chronological order showing author (`@login`), body, and timestamp.
2. **Examine the code** around the referenced line. Line numbers may be stale — use the file path and comment context to locate the relevant code.
3. **Propose a fix**, explaining what the reviewer is asking for, what change you propose, and your reasoning.
4. **Get user approval** with three options:
   - **Approve** — Implement the proposed fix.
   - **Modify** — The user provides additional guidance; incorporate it and re-propose (loop back to step 3).
   - **Skip** — Move on without making changes.
5. **Implement** the fix if approved. Make targeted, minimal edits.

### Per-item flow (autonomous)

For each PR item (comment or unresolved thread) in the chosen subset, in order — comments first, then threads:

1. **Examine the relevant code.** For comments, examine the current state of the code in light of what the comment is asking for; for threads, examine the code around the referenced line, accounting for stale line numbers.
2. For comments only, **assess whether the comment is already addressed** in the current code; if it confidently is, record that and move on to the next item without making changes.
3. **Decide on the best fix.** If any **significant impact** criterion applies, fall back to the matching interactive flow's "Propose a fix" + "Get user approval" steps for this item only; otherwise implement the fix directly with targeted, minimal edits.
4. After all items in the subset have been processed, **summarize** what was done — one short line per item indicating addressed / already-addressed / paused-approved / paused-modified / paused-skipped.

PR review thread resolution state and PR comment state on GitHub are **not** modified by this command; the PR remains the source of truth for its own state.

## Commit and push

Once all available review sources have been processed:

1. If **no code changes** were made across all sources (everything was skipped, or no actionable findings existed), inform the user and **stop**.
2. Otherwise, load the `mrp-commit-message` skill and use it to generate a commit message and offer to commit and push the changes.

> **CRITICAL:** Do NOT push changes without explicit user approval. Always ask the user for confirmation before pushing.
