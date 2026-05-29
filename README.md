# claude-plugins

mrpdaemon's personal Claude plugins.

## Installation

```sh
claude plugin marketplace add mrpdaemon/claude-plugins
```

## Available Plugins

- **dev-workflow**: A task-based development workflow plugin. Bootstraps a task (directory, branch, objectives) — optionally from a Linear issue — then drives it through research, design, spec, implementation (serial or parallel via sub-agents), verification planning and execution, code review, refactoring, addressing review feedback, and iteration on changes. Also handles commit messages and draft PR creation. Backed by explorer, builder, and reviewer sub-agents.
