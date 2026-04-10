# Dev Workflow Plugin - Agent Guidelines

## Version Bumping

Every commit MUST bump the plugin version for both the Claude and Augment plugin manifests.

## Documentation

If adding new skills/commands or updating existing skills/commands, make sure to update `plugins/dev-workflow/README.md` to reflect the changes.

## Command and Skill Authoring Style

Commands and skills must be written in a **declarative** style: describe **what** the agent should accomplish and **what outcomes** are expected, not **how** to do it step by step. Do not include exact shell commands, API call syntax, or code snippets — the agent is capable of figuring out the mechanics on its own. Focus on intent, constraints, and success criteria.
