---
name: "[MRP] Research Task"
description: "Investigate the codebase for the current dev task and write a research report"
---

Load the `mrp-dev-task` skill and follow these steps:

## Step 1: Determine the task name and read the description

Determine the task name using the first available source:
1. Check the `MRP_TASK` environment variable. If it is set and non-empty, use it.
2. Otherwise, ask the user for the task name.

Store the resolved name as `{task_name}`.

Read the `MRP_TASKS_DIR` environment variable. If it is not set or empty, **stop and ask the user** to set it.

Set `{task_dir}` to `$MRP_TASKS_DIR/{task_name}`.

Check that the task directory and description file exist and are non-empty:
```
test -d {task_dir} && test -s {task_dir}/task.md && echo OK || echo MISSING
```

If the result is `MISSING`, tell the user:
> The task directory or task description does not exist (or is empty). Please run the `mrp:new-task` command first.

Then **stop** — do not continue.

If the result is `OK`, read the task description:
```
cat {task_dir}/task.md
```

Store the contents as `{task_description}`.

## Step 2: Check for existing research

Check whether a research report already exists:
```
test -f {task_dir}/research.md && echo EXISTS || echo NEW
```

If the result is `EXISTS`, use the `ask-user` tool to ask the user whether they want to:
- **Regenerate** the research report from scratch
- **Stop** and keep the existing report

If the user chooses **Stop**, end command execution immediately — do not continue.

## Step 3: Investigate the codebase

Run a thorough investigation of the codebase to uncover information relevant to the task. The goal is to gather all the context needed to later create a design and implementation plan. Do **NOT** create a plan or make any code changes — only gather information.

Launch **multiple `sub-agent-mrp-explorer` agents in parallel**. Each agent should focus on a different angle of the investigation. Decide which angles to explore based on the task description, but consider the following:

1. **Existing patterns and conventions** — Find code that already does something similar to what the task requires. Identify the patterns, abstractions, and conventions used so the eventual implementation can follow them.

2. **Related implementations** — Locate the specific files, classes, functions, types, and modules that the task will need to touch or interact with. Read their signatures, docstrings, and surrounding context.

3. **Dependencies and impact areas** — Trace the dependency graph around the areas the task will affect. Identify callers, callees, shared interfaces, tests, configuration, and anything else that could be impacted.

4. **Infrastructure and tooling** — Identify relevant build targets, test commands, deployment configs, feature flags, proto definitions, or other infrastructure the task may depend on.

5. **Edge cases and constraints** — Look for error handling patterns, validation logic, concurrency concerns, backwards-compatibility requirements, or other constraints that the task must respect.

If the task description is broad or touches multiple subsystems, add more parallel explorer agents to cover each subsystem independently.

After all explorer agents complete, synthesize their findings into a single coherent research report. The report should be organized by topic, not by agent. Include:
- Relevant file paths and code references
- Key abstractions and interfaces involved
- Patterns the implementation should follow
- Constraints and risks discovered
- Open questions that need human input before planning can proceed

## Step 4: Write the research report

Write the synthesized report to `{task_dir}/research.md` using `cat` via `launch-process`:

```bash
cat > {task_dir}/research.md << 'AUGEOF'
<report contents here>
AUGEOF
```

After writing, confirm to the user that the research report has been saved and print a brief summary of the key findings.

