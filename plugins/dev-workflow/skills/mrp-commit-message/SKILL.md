---
name: mrp-commit-message
description: Generate well-structured git commit messages for Mark's development workflow. Use when asked to 'commit changes', 'commit to git', 'create a commit', 'make a commit', or any request involving committing code changes.
---

# MRP Commit Message

Generate structured git commit messages following Mark's preferred format.

## Task context

Load the `mrp-dev-task` skill to determine the current task name and task directory location using the `MRP_TASK` environment variable. Read available task files from the task directory to inform the commit message content:

- `task.md` — task description (problem context, may reference a Linear issue)
- `research.md` — investigation report (additional problem insights)
- `design.md` — high-level design (solution context)
- `impl-spec.md` — implementation spec (detailed solution information)
- `verification.md` — verification plan (testing methodology and test cases)

## Commit message structure

### Summary line

Construct the summary line as follows:

1. **Linear issue prefix** — If the task description mentions a Linear issue (e.g. `AU-17889`), start the line with it: `AU-17889: <rest>`.

2. **Change type prefix** — After the optional issue prefix, add a conventional commit type:
   - `fix():` for bug fixes
   - `feat():` for features
   - `chore():` for maintenance tasks

3. **Component scope** — Inside the parentheses, name the major component being changed (e.g. `fix(cli):`, `feat(agent_personas):`). If no single major component is affected or multiple are, omit the parentheses entirely (e.g. `fix: <rest>`).

4. **Summary text** — A VERY concise description of the change. The entire summary line must be under 100 characters, and should aim for under 80 characters.

**Examples:**
- `AU-17889: fix(cli): handle missing config file gracefully`
- `feat(agent_personas): add support for custom persona templates`
- `chore: update dependency versions`

### Body

The commit body has up to three sections, separated by blank lines. Scale the detail to the size of the change:

- **For changes under 10 lines of code:** Collapse problem and solution into a single short paragraph of a few sentences. Use a single `Problem/Solution:` label.
- **For changes under 100 lines of code:** Keep problem and solution descriptions short (a few sentences each).
- **For large changes (hundreds or thousands of lines):** Provide more detailed problem and solution descriptions proportional to the scope of the change.

Each section in the body must be explicitly labeled with a header line:

#### Problem:
Infer from `task.md` and optionally `research.md` in the task directory. If these files are not available, infer the problem from the changes being committed. Explain WHY this commit is being created in a concise paragraph.

#### Solution:
Infer from `design.md` and optionally `impl-spec.md` in the task directory. If these files are not available, infer the solution from the changes being committed. Explain HOW this commit addresses the problem at a high level. Do NOT include code snippets or overly detailed information — summarize the key aspects of the implemented solution.

#### Testing done:
Always include standard testing that is apparent from the code changes (e.g. builds/compiles verified, unit tests added and executed, end-to-end verification steps carried out). Additionally, if `verification.md` exists in the task directory, assume that all testing described in it has been carried out successfully and include a summary of the types of testing from the verification plan. Omit this section entirely only if no testing is evident and no `verification.md` exists.

**Example commit body:**
```
Problem:
The CLI crashes with a stack trace when the user's config file is missing,
rather than falling back to default settings.

Solution:
Add a fallback path in the config loader that initializes default settings
when no config file is found, and log a warning to inform the user.

Testing done:
Added unit tests for the config loader fallback path. Verified manually that
the CLI starts cleanly with no config file present.
```

## Workflow

1. Determine the current task using the `mrp-dev-task` skill.
2. Read available task files (`task.md`, `research.md`, `design.md`, `impl-spec.md`, `verification.md`) from the task directory. Not all files may exist — use whatever is available.
3. Examine the staged or changed files to understand the scope of the change.
4. Compose the commit message following the structure above.

