---
name: "mrp-parallel-impl-task"
description: "Implement all code changes for the current dev task in parallel via mrp-builder sub-agents, then build, test, format, and stage"
---

Load the `mrp-dev-task` skill. Store `{task_name}`, `{task_dir}`, and `{tasks_dir}`. Then load `{task_description}` (required), `{research_report}` (optional), `{design_spec}` (optional), and `{implementation_spec}` (optional) as described in the skill.

If none of the optional files are available, perform your own codebase exploration as needed. Do quick lookups yourself, and launch `mrp-explorer` sub-agents (check your available tools for the one ending in `mrp-explorer`) for deeper exploration.

You are the **coordinator** for this task. Your job is to decompose the work into a dependency graph and dispatch every implementation slice — whether part of a parallel wave or a standalone sequential step — to an `mrp-builder` sub-agent (check your available tools for the one ending in `mrp-builder`). You do not implement slice code yourself; your responsibilities are planning, dispatch, reconciliation, and the integration-level work in later steps.

## Step 1: Plan the implementation into a dependency graph

Decompose the required code changes into a set of **slices**. A slice is a unit of implementation work that can be independently built and unit-tested.

For each slice, identify:

- The files it creates, modifies, or deletes.
- The build targets and unit tests it owns.
- The other slices it depends on (e.g., because it imports a symbol introduced by another slice, or because its tests exercise behavior another slice implements).

From this, derive a dependency graph and determine, for each slice, whether it can run **in parallel** with others or must run **sequentially**.

A pair of slices is safe to run in parallel only if **all** of the following hold:

- They do not edit or create overlapping files.
- Neither depends on symbols, types, or behavior introduced by the other.
- Each can be built and unit-tested independently of the other's changes.

If any of these fail, the slices are sequential and must be ordered accordingly.

Use task management tools to record the slices, their dependencies, and their execution mode (parallel batch vs. sequential), and to track progress as you go.

## Step 2: Execute the graph

Walk the dependency graph in topological order, grouping slices into waves:

- **Parallel wave.** When two or more slices are mutually independent and both independently build-and-unit-testable, dispatch them concurrently as separate `mrp-builder` sub-agent invocations in a single batch of parallel tool calls. Give each builder a precise scope statement covering: the slice's goal, the exact files it may touch, the build targets and unit tests it owns, and any cross-slice assumptions it may rely on from already-completed work.
- **Sequential slice.** When a slice has no parallel peers — either because it is the only remaining independent slice in the wave, or because it sits on a critical path — still dispatch it to a single `mrp-builder` sub-agent rather than implementing it yourself. The builder gets the same kind of precise scope statement as in a parallel wave. Wait for it to complete and return a green report before moving on.

After each wave completes, collect builder reports, reconcile any blockers or conflicts they surface, and update the graph if new dependencies emerge. Do not start a wave until all slices it depends on are green (built and unit-tested).

Constraints for builder dispatch:

- Never dispatch two builders whose scopes edit the same file.
- Never dispatch a builder whose scope depends on a slice that is not yet green.
- Builders do **NOT** run end-to-end tests; keep that out of their scope.

## Step 3: Integration build and unit tests

Once every slice is green in isolation, run a combined build of all targets affected across the whole task, and a combined run of all relevant unit tests, to catch integration issues that per-slice validation can miss.

**Fix integration failures yourself.** Integration failures almost always stem from cross-slice interactions, and diagnosing them requires holding the full picture across slices — which is your responsibility as coordinator, not a builder's. You **MUST** own these fixes directly by default. Do **NOT** dispatch a builder just because a fix exists; the default is always to fix it yourself.

The **only** circumstance in which you may dispatch builders at this step is when there are **multiple, clearly independent failures** that each meet all of the following:

- Each failure is localized to a disjoint set of files with no overlap between them.
- Each fix is independently build-and-unit-testable without reference to the others.
- Fixing them in parallel produces a meaningful wall-clock speedup over fixing them yourself sequentially.

If any of these conditions is in doubt, fix it yourself. A single integration failure, or a cluster of failures that share root causes or files, is always yours to fix.

Repeat until the full affected surface is green.

## Step 4: Run end-to-end tests (only if changed)

If a **new** end-to-end test was added, or an **existing** end-to-end test was modified as part of this task, run it and fix any failures.

If there was no actual change to an end-to-end test, do **NOT** attempt to run end-to-end tests.

## Step 5: Format and stage

Once all builds and tests pass:

### 5a: Format the code

Run the project's code formatter on all changed files. Determine the appropriate format command by examining the project's configuration files (e.g., Makefile, package.json scripts, formatting config files, BUILD files, etc.).

### 5b: Stage all changes

Stage all changed and newly created files in git:
```
git add -A
```

Confirm to the user that all changes have been formatted and staged, and print a brief summary of what was implemented, including which slices ran in parallel vs. sequentially.
