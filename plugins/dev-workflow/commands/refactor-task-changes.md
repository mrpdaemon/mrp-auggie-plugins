---
name: "mrp-refactor-task-changes"
description: "Identify refactoring/simplification opportunities in the current task's changes, get user approval, and apply them via mrp-builder sub-agents"
---

Load the `mrp-dev-task` skill. Store `{task_name}`, `{task_dir}`, and `{tasks_dir}`. Then load `{task_description}` (required), `{design_spec}` (optional), and `{implementation_spec}` (optional) as described in the skill.

You are the **coordinator** for this refactor pass. Your job is to assess the task's changes for refactoring and simplification opportunities, get user approval on which to undertake, sequence them, and dispatch each one to an `mrp-builder` sub-agent. You do not implement the refactors yourself; your responsibilities are analysis, planning, dispatch, and the integration-level work in later steps.

## Step 1: Survey the changes done so far

Examine the full set of changes for the current task: commits on the task branch (`markp/{task_name}`) against the base branch (main or master), plus any staged or unstaged changes in the working tree. Build a working understanding of:

- The files touched and their roles in the codebase.
- The shape of the change — new abstractions introduced, existing code modified, control-flow added, tests added.
- Any structural patterns that recur across multiple changed files.

Use `{task_description}` (and `{design_spec}` / `{implementation_spec}` if present) to calibrate what the task **required**. Note the size of the diff (lines added/removed, files touched) and form a rough sense of its complexity.

## Step 2: Reason about complexity vs. what the task required

Compare the actual change footprint against the complexity the task genuinely calls for. Indicators that refactoring or simplification may be warranted include:

- The change is materially larger or more elaborate than the task requires.
- Repeated or near-duplicated code across changed files that could be unified.
- New abstractions, indirection, or configuration knobs that are not earning their keep.
- Naming, structure, or layering that obscures intent or breaks from surrounding conventions.
- Dead code, unused parameters, or speculative generality not justified by the task.
- Test code that mirrors structural problems in the production code.

Apply judgment. A change that is appropriately sized for a complex task is **not** an opportunity. Only flag refactors that would leave the code clearly simpler, smaller, or more aligned with the task's actual scope without changing behavior.

## Step 3: Present opportunities and get user selection

If no meaningful opportunities are found, inform the user and **stop** without making any changes.

Otherwise, present the candidate opportunities to the user as a numbered list. For each opportunity include:

- A short title.
- A one-paragraph description of what would change and why.
- The files (or file groups) it would touch.
- A rough complexity/risk indicator (small / medium / large).

Ask the user to select which opportunities to undertake. Accept any unambiguous selection — a list of identifiers, a range, "all", or "none". If the user picks **none**, stop without making changes.

For any selected opportunity that meets the **significant impact** bar — changes a public API or shared contract, materially alters behavior beyond simplification, touches many files, or has multiple plausible approaches with different trade-offs — confirm the chosen approach with the user before proceeding.

## Step 4: Sequence the selected refactors

Treat each selected opportunity as a **slice**. For each slice, identify the files it will touch, the build targets and unit tests it owns, and any ordering dependencies on other slices (e.g., one slice extracts a helper that another slice then consumes, or two slices edit overlapping files).

From this, derive a logical order of execution and decide, for each slice, whether it can run **in parallel** with others or must run **sequentially**. A pair of slices is safe to run in parallel only if **all** of the following hold:

- They do not edit or create overlapping files.
- Neither depends on changes introduced by the other.
- Each can be built and unit-tested independently of the other's changes.

If any condition fails, the slices are sequential. When in doubt, prefer sequential execution — refactors interact in subtle ways, and a clean order is cheaper than an integration mess.

Use task management tools to record the slices, their order, and their execution mode (parallel batch vs. sequential), and to track progress as you go.

## Step 5: Execute the plan via mrp-builder sub-agents

Walk the planned order, grouping slices into waves:

- **Parallel wave.** When two or more slices are mutually independent and each is independently build-and-unit-testable, dispatch them concurrently as separate `mrp-builder` sub-agent invocations in a single batch of parallel tool calls (check your available tools for the one ending in `mrp-builder`). Give each builder a precise scope statement covering: the refactor's goal, the exact files it may touch, the build targets and unit tests it owns, the behavioral invariant it must preserve, and any cross-slice assumptions it may rely on from already-completed work.
- **Sequential slice.** When a slice has no parallel peers, still dispatch it to a single `mrp-builder` sub-agent rather than implementing it yourself, with the same kind of precise scope statement. Wait for it to return a green report before moving on.

After each wave, collect builder reports, reconcile any blockers or conflicts they surface, and revise the plan if new dependencies emerge. Do not start a wave until all slices it depends on are green.

Constraints for builder dispatch:

- Never dispatch two builders whose scopes edit the same file.
- Never dispatch a builder whose scope depends on a slice that is not yet green.
- Every builder's scope must explicitly state that the refactor is **behavior-preserving** — no functional changes, only structural or stylistic ones — unless the user explicitly approved a behavior change in Step 3.
- Builders do **NOT** run end-to-end tests; keep that out of their scope.

## Step 6: Integration build and unit tests

Once every slice is green in isolation, run a combined build of all targets affected by the refactor pass and a combined run of all relevant unit tests, to catch integration issues that per-slice validation can miss.

**Fix integration failures yourself** by default — they almost always stem from cross-slice interactions and require holding the full picture, which is your responsibility as coordinator. Only dispatch builders for integration fixes when there are multiple, clearly independent failures in disjoint files that are each independently build-and-unit-testable; otherwise, fix them yourself. Repeat until the full affected surface is green.

If a **new** end-to-end test was added or an **existing** end-to-end test was modified by this refactor pass, run it and fix any failures. Otherwise, do **NOT** attempt to run end-to-end tests.

## Step 7: Format, stage, and commit

Once all builds and tests pass:

1. Run the project's code formatter on all changed files. Determine the appropriate format command by examining the project's configuration files (e.g., Makefile, package.json scripts, formatting config files, BUILD files, etc.).
2. Stage all changed and newly created files in git (`git add -A`).
3. Load the `mrp-commit-message` skill and use it to generate a commit message describing the refactor pass, then create the commit.

> **CRITICAL:** Do NOT push the commit. Pushing is out of scope for this command.

Report to the user:

- Which opportunities were selected and applied (and which were skipped or deferred).
- A brief per-slice summary of what changed.
- The commit SHA and one-line summary of the commit that was created.
