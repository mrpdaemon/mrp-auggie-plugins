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
| `research-report.md` | `/research-task` | Codebase research report |
| `design-spec.md` | `/design-task` | High-level design document |
| `implementation-spec.md` | `/spec-task` | Detailed implementation spec |
| `verification-plan.md` | `/plan-task-verification` | Verification plan with test cases |
| `iterations.md` | `/iterate-task` | Task iteration history tracking changes across iterations |
| `user-review.txt` | user (manual) | Review comments authored by the user, organized into rounds |
| `review-findings.md` | `/review-task-changes` | Review findings produced by reviewer sub-agents, organized into rounds |

## Workflow

Either `/new-task` or `/linear-task` is used to create a task, then `/impl-task` implements it. The intermediate commands (`/research-task`, `/design-task`, `/spec-task`) are optional and can be skipped — for example, you can go directly from `/new-task` to `/impl-task`. When intermediate outputs exist, later commands will use them automatically; when they don't, the commands will explore the codebase on their own.

### Example workflows by task complexity

Pick a workflow that matches the task's complexity. Each step assumes the task has already been created via `/new-task` or `/linear-task`.

- **Trivial** — `/design-task` → `/impl-task` → `/verify-task` → `/make-task-pr`
  - No dedicated research, impl spec, or verification plan. Those steps are lightweight enough to be folded in: `/design-task` does any needed research, `/impl-task` plans its own implementation, and `/verify-task` plans its own verification.
- **Low complexity** — `/research-task` → `/design-task` → `/impl-task` → `/verify-task` → `/make-task-pr`
  - A dedicated research step improves design fidelity.
- **Medium complexity** — `/research-task` → `/design-task` → `/plan-task-verification` → `/spec-task` → `/impl-task` → `/verify-task` → `/make-task-pr`
  - Dedicated verification and implementation planning improves robustness by allowing more user involvement.
- **High complexity** — `/research-task` → `/design-task` → `/plan-task-verification` → `/spec-task` → `/parallel-impl-task` → `/verify-task` → `/make-task-pr`
  - For large tasks, use coordinator/builder orchestration during implementation.

Some steps can be run in parallel to shorten the critical path. `/plan-task-verification` only depends on the design (and optionally research), so once `/design-task` has completed it can run concurrently with `/spec-task`, or even with `/impl-task` / `/parallel-impl-task` when the spec is skipped. Running them in separate agent sessions works well since their outputs (`verification-plan.md` vs. `implementation-spec.md` / source changes) don't overlap.

### `/new-task`

Create a new task directory and populate it with a task description.

- **Writes:** `task.md` (required) — the task description, provided by the user.

### `/linear-task`

Create a new task directory from a Linear issue. Looks up the issue, derives a task name, and synthesizes the issue contents into a task description.

- **Writes:** `task.md` (required) — the task description, synthesized from the Linear issue.

### `/research-task`

Investigate the codebase and produce a research report with all the context needed for design and implementation.

- **Reads:** `task.md` (required)
- **Writes:** `research-report.md` — synthesized findings including relevant files, key abstractions, patterns, constraints, and open questions.

### `/design-task`

Collaborate interactively with the user on a high-level design. Walks through key design questions one at a time, then synthesizes the answers into a design document.

- **Reads:** `task.md` (required), `research-report.md` (optional)
- **Writes:** `design-spec.md` — design decisions, chosen approach, components to modify, and open questions/risks.

### `/spec-task`

Collaborate interactively with the user on a detailed implementation spec. Explores the codebase to identify every file and function that needs to change, resolves ambiguities with the user, and defines a testing plan.

- **Reads:** `task.md` (required), `research-report.md` (optional), `design-spec.md` (optional)
- **Writes:** `implementation-spec.md` — file-by-file change descriptions, new file specifications, dependency ordering, and testing plan.

### `/plan-task-verification`

Create a verification plan with testing methodology and test cases for the current task. Investigates testing frameworks and tools available in the codebase, proposes a methodology, and identifies thorough test cases covering happy paths, edge cases, error conditions, and contract validation. When the codebase already has an established e2e test framework and that framework is the chosen methodology, the plan also includes a concise, CI-compatible set of permanent additions to the existing e2e suite.

- **Reads:** `task.md` (required), `research-report.md` (optional), `design-spec.md` (optional)
- **Writes:** `verification-plan.md` — testing methodology, test cases by category, and coverage notes.

### `/impl-task`

Implement all code changes, then build, test, format, and stage. Follows the implementation spec if available, otherwise falls back to the design or research for guidance.

- **Reads:** `task.md` (required), `research-report.md` (optional), `design-spec.md` (optional), `implementation-spec.md` (optional)
- **Writes:** source code changes, staged via `git add -A`.

### `/parallel-impl-task`

Same outcome as `/impl-task`, but the coordinating agent decomposes the work into a dependency graph of implementation slices and dispatches independent slices concurrently to `mrp-builder` sub-agents, handling sequential slices itself. After all slices are green, it runs an integration build and unit tests, then end-to-end tests (only if changed), formats, and stages.

- **Reads:** `task.md` (required), `research-report.md` (optional), `design-spec.md` (optional), `implementation-spec.md` (optional)
- **Writes:** source code changes, staged via `git add -A`.

### `/verify-task`

Execute the verification plan for the current task. Performs any required setup, runs every test case from the verification plan, and reports results. If there are failures, offers to diagnose and fix them. If no verification plan exists yet, one is generated before execution begins. The full verification plan is always re-executed from scratch — any existing verification report is consulted only for context and never used to skip or infer test case outcomes.

- **Reads:** `task.md` (required), `research-report.md` (optional), `design-spec.md` (optional), `verification-plan.md` (optional — generated if missing), `verification-report.md` (optional — context only)
- **Writes:** `verification-plan.md` (if not already present), `verification-report.md`, code fixes (if the user opts to diagnose and fix failures).

### `/iterate-task`

Detect material changes since the last documented state, record them as a numbered iteration in `iterations.md`, and surgically update affected task artifacts to stay in sync with what was actually implemented.

- **Reads:** `task.md` (required), `design-spec.md` (optional), `implementation-spec.md` (optional), `verification-plan.md` (optional), `iterations.md` (optional)
- **Writes:** `iterations.md` — created or appended with a new iteration section; `design-spec.md`, `implementation-spec.md`, `verification-plan.md` — surgically updated with cross-reference notes and content edits where affected.

### `/make-task-pr`

Ensure the task branch has exactly one well-messaged commit, push the branch, and create a draft GitHub PR. Uses the `mrp-commit-message` skill to generate the commit message. If the branch has multiple commits they are squashed; if it already has a single commit whose message conforms to the skill's format, the commit is left untouched; if it has a single commit with a non-conforming message, the message is reworded via an amend. Cleans stale Git SHA references from `iterations.md` when history was rewritten, and records the PR number in `task.md`.

- **Reads:** `task.md` (required), `iterations.md` (optional), plus all task artifacts used by the `mrp-commit-message` skill for commit message generation.
- **Writes:** `iterations.md` (removes stale SHA references), `task.md` (appends PR number).

### `/review-task-changes`

Spawn five `mrp-reviewer` sub-agents in parallel to review the changes on the task branch, deduplicate and severity-rank their findings, and record them as a new numbered round in `review-findings.md`. Findings marked `SKIPPED` in a prior round are excluded from the new round; findings marked `ADDRESSED` that the reviewers still flag are re-included.

- **Reads:** `task.md` (required), `review-findings.md` (optional — prior rounds inform deduplication).
- **Writes:** `review-findings.md` — appends a new `Round N [NEW]` section listing the deduplicated findings in severity order.

### `/address-task-review-comments`

Walk through review feedback from all available sources — `user-review.txt`, `review-findings.md`, and unresolved review threads and top-level comments on the task PR — and propose and implement fixes interactively. For file-based sources, processes every round that is not marked `COMPLETE`; each addressed finding is updated to `ADDRESSED`, each skipped finding to `SKIPPED`, and the round is marked `COMPLETE` once all of its findings have been resolved. For the task PR, unresolved review threads are processed using their resolution status, and top-level PR comments are individually assessed against the current code to decide whether they still need addressing. PR thread resolution state and PR comment state on GitHub are not modified by this command.

- **Reads:** `task.md` (required), `user-review.txt` (optional), `review-findings.md` (optional) — and unresolved PR review threads and top-level PR comments when a PR exists.
- **Writes:** source code changes based on approved feedback; updates finding/round status in `user-review.txt` and `review-findings.md`; optionally commits and pushes via `mrp-commit-message`.

