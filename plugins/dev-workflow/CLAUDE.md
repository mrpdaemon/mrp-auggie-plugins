# Dev Workflow Plugin - Agent Guidelines

## Version Bumping

Every commit MUST bump the plugin version for the Claude plugin manifest.

## Documentation

If adding new skills/commands or updating existing skills/commands, make sure to update `plugins/dev-workflow/README.md` to reflect the changes.

## Command and Skill Authoring Style

Commands and skills must be written in a **declarative** style: describe **what** the agent should accomplish and **what outcomes** are expected, not **how** to do it step by step. Do not include exact shell commands, API call syntax, or code snippets — the agent is capable of figuring out the mechanics on its own. Focus on intent, constraints, and success criteria.

Do **not** refer to specific tools by name (e.g., `ask-user`, `web-search`). Tool names vary across runtimes, so naming them couples commands to a particular host. Instead, instruct the agent in terms of the action or outcome — "ask the user", "search the codebase", "look up information online" — and let the agent pick the appropriate tool from whatever it has available. Sub-agent names defined by this plugin (e.g., `mrp-explorer`, `mrp-builder`, `mrp-reviewer`) and skill names defined by this plugin (e.g., `mrp-dev-task`, `mrp-commit-message`) are part of the plugin's own surface area and may be referenced directly.
