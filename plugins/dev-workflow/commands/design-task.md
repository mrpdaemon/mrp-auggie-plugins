---
name: "mrp-design-task"
description: "Collaborate on a high-level design for the current dev task and save it as design.md"
---

Load the `mrp-dev-task` skill. Store `{task_name}`, `{task_dir}`, and `{tasks_dir}`. Then load `{task_description}` (required) and `{research}` (optional) as described in the skill.

If `{research}` is not available, you will perform your own codebase exploration as needed during the design process.

## Step 1: Check for existing design

Check whether a design document already exists:
```
test -f {task_dir}/design.md && echo EXISTS || echo NEW
```

If the result is `EXISTS`, use the `ask-user` tool to ask the user whether they want to:
- **Regenerate** the design from scratch
- **Stop** and keep the existing design

If the user chooses **Stop**, end command execution immediately — do not continue.

## Step 2: Collaborate on the high-level design

Your goal is to work **interactively** with the user to produce a high-level design. Do **NOT** write code or create a detailed implementation plan.

### 2a: Identify key design questions

Based on `{task_description}` and `{research}` (if available), identify the key design questions that need to be answered before an approach can be chosen. These may include:

- **Architecture**: Which components/services are involved? Should new ones be created or should existing ones be extended?
- **UX**: How should the feature surface to the user? What interactions are needed?
- **APIs / Protobufs**: What new or modified RPCs, endpoints, or message types are needed?
- **Data model**: What new or modified storage schemas, tables, or data structures are required?
- **Integration points**: How does this interact with existing systems, feature flags, or configuration?
- **Trade-offs**: Are there competing approaches? What are the pros and cons?

If no research report was available, use `codebase-retrieval` and `sub-agent-mrp-explorer` agents to explore the codebase as needed to inform the design questions.

### 2b: Walk through questions one by one

Present each design question to the user **one at a time** using the `ask-user` tool. For each question:

1. Explain the question and why it matters.
2. Provide your **recommendation** with reasoning.
3. Offer concrete options for the user to choose from.

Incorporate the user's answer before moving to the next question. If a user's answer changes the nature of subsequent questions, adapt accordingly.

### 2c: Propose an approach

Once all key questions are answered, synthesize the answers into a coherent approach. Present the proposed approach to the user and ask for confirmation or adjustments.

## Step 3: Write the design document

Once the user approves the approach, write a detailed design document to `{task_dir}/design.md`.

The design document should include:

1. **Summary** — A brief overview of what is being built and why.
2. **Design decisions** — The key questions that were considered and the chosen answers with rationale.
3. **Approach** — The high-level approach that satisfies the design decisions.
4. **Components to modify** — A list of the components, services, files, or modules that need to change, with a description of the nature of each change (not implementation details).
5. **Key design elements** — Detailed treatment of any of the following that apply:
   - UX changes (user-facing behavior, interactions, UI elements)
   - API / Protobuf changes (new or modified RPCs, messages, endpoints)
   - Data model changes (new or modified schemas, tables, storage)
   - Configuration / feature flag changes
6. **Open questions / risks** — Anything that remains uncertain or risky.

Focus on **what** needs to change and **why**, not **how** to implement it line by line.

After writing, confirm to the user that the design document has been saved and print a brief summary.

