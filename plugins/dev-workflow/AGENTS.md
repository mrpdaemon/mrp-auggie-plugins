# Dev Workflow Plugin - Agent Guidelines

## Version Bumping

Every commit MUST bump the plugin version for both the Claude and Augment plugin manifests.

## Documentation

If adding new skills/commands or updating existing skills/commands, make sure to update `plugins/dev-workflow/README.md` to reflect the changes.

## Command Style

Commands and skills should use a **declarative style** — describe *what* to accomplish, not *which tools* or shell commands to use. Do not reference specific tool names (e.g., `ask-user`, `view`, `str-replace-editor`, `launch-process`, `github-api`). Let the agent choose the appropriate tools and shell commands at runtime.
