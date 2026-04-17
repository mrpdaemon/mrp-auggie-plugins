---
name: "mrp-plan-task-verification"
description: "Create a verification plan with testing methodology and test cases for the current dev task"
---

Load the `mrp-dev-task` skill. Store `{task_name}`, `{task_dir}`, and `{tasks_dir}`. Then load `{task_description}` (required), `{research_report}` (optional), and `{design_spec}` (optional) as described in the skill.

## Step 1: Check for existing verification plan

Check whether the verification plan file `{verification_plan}` already exists.

If the file exists, use the `ask-user` tool to ask the user whether they want to:
- **Regenerate** the verification plan from scratch
- **Stop** and keep the existing plan

## Step 2: Determine testing methodology

Based on `{task_description}`, `{research_report}` (if available), and `{design_spec}` (if available), investigate how the changes introduced by this task can best be tested. Use `codebase-retrieval` for quick lookups, and launch `mrp-explorer` sub-agents (check your available tools for the one ending in `mrp-explorer`) to:

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
- **Contract validation** — Verify that the changes honor the design contract and interfaces rather than testing implementation assumptions. If `{design_spec}` is available, derive test cases from the design decisions and approach described there.
- **Regression** — Ensure existing functionality that could be affected by the changes still works correctly.

Be thorough but practical — every test case should be executable using the chosen methodology.

## Step 4: Plan permanent additions to the existing e2e test suite

This step applies **only** when the codebase already has an established end-to-end testing framework **and** the chosen testing methodology is to use that framework. Otherwise, skip this step.

When it applies, the verification plan must include a dedicated section describing permanent additions to the existing e2e test suite that will land alongside the task's code changes. These additions are committed to the repository and run on CI — they are distinct from test cases that are only executed once during `/verify-task`.

When planning the permanent additions:

- **Prioritize coverage of the main task deliverable** — focus the permanent tests on the core behavior introduced or changed by this task, not on incidental or tangential scenarios.
- **Keep the additions concise** — the top priority is avoiding a meaningful increase in e2e test runtime. Either select a small subset of the full verification test cases for permanent inclusion, or write targeted, focused test cases specifically for the permanent suite. Do not port the entire verification plan into the permanent suite.
- **Ensure CI-compatibility** — permanent tests must run on CI machines, so they cannot depend on the user's local machine state, local dev deploys, personal credentials, or other non-reproducible environment assumptions. Call out the required test data, fixtures, environment variables, service dependencies, and any setup the permanent tests rely on, and confirm each is available (or can be made available) in CI.

## Step 5: Write the verification plan

Write the complete verification plan to `{task_dir}/{verification_plan}`.

The verification plan should include:

1. **Summary** — A brief overview of what is being verified and the scope of testing.
2. **Testing methodology** — The chosen approach for executing tests, including:
   - Tools, frameworks, or commands used
   - How to set up and run tests
   - Why this methodology was chosen
3. **Test cases** — The full set of test cases organized by category, each with name, purpose, steps, and expected result.
4. **Permanent e2e suite additions** — Only when Step 4 applies. List the specific test cases to be added permanently to the existing e2e test suite, where they will live, and the CI-compatible test data, fixtures, and setup they require.
5. **Coverage notes** — Any areas that are intentionally not covered and why (e.g., out of scope, covered by existing tests, requires infrastructure not available).

After writing, confirm to the user that the verification plan has been saved and print a brief summary of the number of test cases by category.
