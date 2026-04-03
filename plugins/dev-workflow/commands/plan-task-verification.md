---
name: "mrp-plan-task-verification"
description: "Create a verification plan with testing methodology and test cases for the current dev task"
---

Load the `mrp-dev-task` skill. Store `{task_name}`, `{task_dir}`, and `{tasks_dir}`. Then load `{task_description}` (required), `{research}` (optional), and `{design}` (optional) as described in the skill.

## Step 1: Check for existing verification plan

Check whether a verification plan already exists:
```
test -f {task_dir}/verification.md && echo EXISTS || echo NEW
```

If the result is `EXISTS`, use the `ask-user` tool to ask the user whether they want to:
- **Regenerate** the verification plan from scratch
- **Stop** and keep the existing plan

If the user chooses **Stop**, end command execution immediately — do not continue.

## Step 2: Determine testing methodology

Based on `{task_description}`, `{research}` (if available), and `{design}` (if available), investigate how the changes introduced by this task can best be tested. Use `codebase-retrieval` and `sub-agent-mrp-explorer` agents to:

- Examine existing test infrastructure in the codebase (test frameworks, test runners, test helpers, fixtures).
- Look at existing end-to-end tests, integration tests, and CLI-based testing patterns already in use.
- Identify relevant agent skills or CLI tools that could be used to exercise the changes.
- Assess how the changes can be verified end-to-end — via end-to-end tests, CLI invocations, or a combination.

Unit tests and integration tests are **out of scope** — focus exclusively on **end-to-end testing** that exercises the changes from the user's perspective. Focus on **simplicity and speed** — prefer lightweight end-to-end approaches that give fast feedback over heavyweight test harnesses.

If there are **multiple viable testing methodologies**, present your **recommendation** along with the alternatives to the user using the `ask-user` tool. Explain the trade-offs (speed, coverage, complexity, reliability) and let the user pick the most suitable option.

If there is a single clear methodology, present it to the user for confirmation before proceeding.

### 2a: Obtain test data and configuration

Identify any non-trivial configuration or test data required to execute the end-to-end tests. This may include API keys, authentication tokens, config files, service URLs, environment variables, seed data, or external account credentials. If any such requirements exist, use the `ask-user` tool to ask the user to provide them.

## Step 3: Identify test cases

Once the testing methodology is agreed upon, identify a thorough set of test cases to verify that the changes accomplish the goal laid out by the task. Organize test cases into logical groups and for each test case specify:

1. **Name** — A short, descriptive name.
2. **Purpose** — What aspect of the task this test verifies.
3. **Steps** — Concrete steps to execute the test using the chosen methodology.
4. **Expected result** — What success looks like.

Cover the following categories:

- **Happy path** — The primary expected functionality works correctly.
- **Edge cases** — Boundary conditions, empty inputs, unusual but valid inputs.
- **Error conditions** — Invalid inputs, missing dependencies, failure modes, and graceful error handling.
- **Contract validation** — Verify that the changes honor the design contract and interfaces rather than testing implementation assumptions. If `{design}` is available, derive test cases from the design decisions and approach described there.
- **Regression** — Ensure existing functionality that could be affected by the changes still works correctly.

Be thorough but practical — every test case should be executable using the chosen methodology.

## Step 4: Write the verification plan

Write the complete verification plan to `{task_dir}/verification.md`.

The verification plan should include:

1. **Summary** — A brief overview of what is being verified and the scope of testing.
2. **Testing methodology** — The chosen approach for executing tests, including:
   - Tools, frameworks, or commands used
   - How to set up and run tests
   - Why this methodology was chosen
3. **Test cases** — The full set of test cases organized by category, each with name, purpose, steps, and expected result.
4. **Coverage notes** — Any areas that are intentionally not covered and why (e.g., out of scope, covered by existing tests, requires infrastructure not available).

After writing, confirm to the user that the verification plan has been saved and print a brief summary of the number of test cases by category.
