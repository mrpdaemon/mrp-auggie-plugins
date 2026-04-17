---
name: "mrp-verify-task"
description: "Execute the verification plan for the current dev task and report results"
---

Load the `mrp-dev-task` skill. Store `{task_name}`, `{task_dir}`, and `{tasks_dir}`. Then load `{task_description}` (required), `{research_report}` (optional), `{design_spec}` (optional), and `{verification_plan}` (optional) as described in the skill.

## Step 0: Ensure a verification plan exists

If `{verification_plan}` is not present in the task directory, produce one before proceeding:

1. Based on `{task_description}`, `{research_report}` (if available), and `{design_spec}` (if available), determine an appropriate **end-to-end** testing methodology. Examine existing test infrastructure (frameworks, runners, fixtures, CLI-based testing patterns) using `codebase-retrieval` or `mrp-explorer` sub-agents (check your available tools for the one ending in `mrp-explorer`). Unit and integration tests are out of scope — focus on lightweight end-to-end approaches that exercise the changes from the user's perspective. If there are multiple viable methodologies, present your recommendation with alternatives to the user via `ask-user`; otherwise, present the single chosen methodology for confirmation.
2. Identify any non-trivial configuration or test data required (API keys, credentials, service URLs, seed data, environment variables, etc.) and use `ask-user` to obtain them from the user.
3. Identify a thorough set of test cases covering **happy path**, **edge cases**, **error conditions**, **contract validation**, and **regression**. For each test case specify name, purpose, steps, and expected result.
4. Write the verification plan to `{task_dir}/{verification_plan}` with sections for summary, testing methodology, test cases organized by category, and coverage notes.

Once `{verification_plan}` is in place, continue with the remaining steps.

## Step 1: Set up the test environment

Follow the **Testing methodology** section of `{verification_plan}` to perform any required setup. This may include:

- Installing tools or dependencies
- Setting environment variables or configuration files
- Deploying services
- Preparing test data or seed data

If the verification plan references configuration values that are missing or placeholders (e.g., API keys, URLs, credentials), use the `ask-user` tool to ask the user to provide them.

Do **not** proceed to test execution until all setup steps have been completed successfully.

## Step 2: Execute test cases

Work through **every** test case listed in `{verification_plan}`, in the order they appear. For each test case:

1. Execute the steps exactly as described in the verification plan.
2. Record the **actual result**.
3. Compare the actual result against the **expected result**.
4. Mark the test case as **PASS** or **FAIL**.

Use task management tools to track each test case and mark them complete as you go.

If a test case cannot be executed (e.g., a required service is unavailable, a prerequisite failed), mark it as **BLOCKED** and note the reason. Continue with the remaining test cases.

## Step 3: Report results

Once all test cases have been executed, print a summary table with:

| Test Case | Category | Result |
|-----------|----------|--------|
| (name) | (category) | PASS / FAIL / BLOCKED |

Include totals: how many passed, failed, and blocked.

For each **FAIL**, include a brief description of the actual result versus the expected result.

## Step 4: Diagnose failures (if any)

If there are any **FAIL** results, use the `ask-user` tool to ask the user whether they want to:
- **Diagnose and fix** — Investigate the root cause of the failures and attempt to fix the underlying code.
- **Stop** — End verification and leave the failures for later.

If the user chooses **Diagnose and fix**:

1. For each failed test case, investigate the root cause using `codebase-retrieval` and the failure details.
2. Make the necessary code fixes.
3. Re-run the failed test cases to confirm they now pass.
4. Repeat until all previously failed test cases pass, or until the user decides to stop.

If the user chooses **Stop**, proceed to the verification report.

## Step 5: Write verification report

Write the verification report to `{verification_report}` in the task directory using the `save-file` tool. The report should contain:

1. **Summary** — Overall result (all pass, some failures, etc.) with counts of passed, failed, and blocked test cases.
2. **Results table** — The full results table from Step 3.
3. **Failure details** — For each FAIL or BLOCKED test case, include:
   - The test case name and category
   - The expected result
   - The actual result
   - Root cause analysis and fix description (if diagnosed and fixed in Step 4)
4. **Notes** — Any observations, environment issues, or caveats encountered during verification.

## Step 6: Clean up temporary files

Once verification is fully complete (all test cases pass, or the user chose to stop), clean up any temporary files that were created during the verification process. This includes but is not limited to:

- Test scripts written to disk to execute test cases
- Temporary configuration or environment files
- Test data, fixtures, or seed data files
- Any other files created solely for the purpose of verification

Do **not** remove files that were part of the original task implementation, that existed before verification began, or that were created or modified as part of diagnosing and fixing failures in Step 4. Only remove files that the verification process itself created for the purpose of executing test cases.
