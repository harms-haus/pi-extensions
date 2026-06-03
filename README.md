# pi-extensions

A curated collection of extensions and tools for building a powerful [pi coding agent](https://github.com/earendil-works/pi-coding-agent) for software development. Each extension is an independent package that drops into pi via `pi install` and adds new capabilities — from language-server intelligence to process management, workflow orchestration, and web search.

## Extensions

### [pi-cwd](https://github.com/harms-haus/pi-cwd)

Change the effective working directory without restarting the agent. Provides a `/cwd` command with tab-completion, a footer indicator, and session persistence across reloads.

```bash
pi install npm:@harms-haus/pi-cwd
```

### [pi-git](https://github.com/harms-haus/pi-git)

Rich git status tracking with a live, color-coded footer label and an agent-end summary of changed files. Integrates with [pi-powerline](#pi-powerline) for display.

```bash
pi install npm:@harms-haus/pi-git
```

### [pi-lens](https://github.com/harms-haus/pi-lens)

Unified code quality checks after every file change. Hooks after `write`, `edit`, and `bash` tools to automatically run prettier (report-only), 11 linters (ESLint, Biome, Ruff, Flake8, Pylint, Mypy, Clippy, staticcheck, RuboCop, ShellCheck, Stylelint), LSP diagnostics across 33 languages, and TypeScript `tsc --noEmit` — all concurrently via `Promise.all`. Configurable via `.pi-lens.json`.

```bash
pi install npm:@harms-haus/pi-lens
```

### [pi-processes](https://github.com/harms-haus/pi-processes)

Process management tools. Spawn, monitor, kill, and restart long-running processes (dev servers, watchers, API backends) with debounce-based startup detection, log querying, and SIGTERM → SIGKILL escalation.

```bash
pi install npm:@harms-haus/pi-processes
```

### [pi-subagents](https://github.com/harms-haus/pi-subagents)

Spawn parallel sub-agents with live TUI output windows. Each sub-agent runs in its own isolated process with optional named profiles for provider/model, system prompts, and thinking levels.

```bash
pi install npm:@harms-haus/pi-subagents
```

### [pi-kanban](https://github.com/harms-haus/pi-kanban)

Phased task board with strict status gating, dependency tracking, auto-continue, and session persistence. Tasks progress through an enforced lifecycle: `draft` → `ready` → `testing` → `implementing` → `review` → `done`.

```bash
pi install npm:@harms-haus/pi-kanban
```

### [pi-til-done](https://github.com/harms-haus/pi-til-done)

Iterative todo list that auto-loops the agent until every task is complete. Provides `write_todos`, `list_todos`, and `edit_todos` tools with a 3-second auto-continue countdown, 20-iteration circuit breaker, and event-sourced state that survives session restarts.

```bash
pi install npm:@harms-haus/pi-til-done
```

### [pi-update](https://github.com/harms-haus/pi-update)

Update pi core and all extensions from within the agent. Runs updates in parallel with a `/update` command and auto-reloads when changes are detected.

```bash
pi install npm:@harms-haus/pi-update
```

### [pi-web-content](https://github.com/harms-haus/pi-web-content)

Unified `fetch_content` tool for fetching web pages (converted to clean markdown via Mozilla Readability) and cloning git repositories. Auto-detects URL type and handles both transparently.

```bash
pi install npm:@harms-haus/pi-web-content
```

### [pi-workflows](https://github.com/harms-haus/pi-workflows)

Define and run named multi-phase workflows with per-phase tool control, subworkflow nesting, state persistence, and auto-continue enforcement. Compose complex agent behaviors from simple phase definitions.

```bash
pi install npm:@harms-haus/pi-workflows
```

### [pi-worktrees](https://github.com/harms-haus/pi-worktrees)

Manage git worktrees with slash commands for creating, switching, merging, and cleaning up worktrees. Each worktree gets its own directory so you can work on multiple branches simultaneously without stashing or committing. Requires [pi-cwd](#pi-cwd).

```bash
pi install npm:@harms-haus/pi-worktrees
```

### [pi-zai-usage](https://github.com/harms-haus/pi-zai-usage)

Monitors Z.ai token quota in real time and displays a color-coded progress bar in the pi-powerline footer. Caches responses to minimize API calls and only activates when a Z.ai provider is selected.

```bash
pi install npm:@harms-haus/pi-zai-usage
```

## Quick Start

Install individual extensions with pi:

```bash
pi install npm:@harms-haus/pi-lens
pi install npm:@harms-haus/pi-workflows
```

Then restart pi or run `/reload`.

## Recommended Setup

For a fully-featured software development agent, install these extensions together:

| Extension | Purpose |
|-----------|---------|
| **pi-lens** | Unified code quality checks (linters, LSP diagnostics, prettier, tsc) after every edit |
| **pi-git** | Live git status in the footer |
| **pi-til-done** | Iterative task tracking with auto-continue |
| **pi-kanban** | Phased task board with dependency tracking |
| **pi-subagents** | Parallel sub-agent execution |
| **pi-workflows** | Multi-phase workflow orchestration |
| **pi-processes** | Dev server and process management |
| **pi-web-content** | Web fetching and repo cloning |
| **pi-worktrees** | Git worktree management |
| **pi-update** | One-command self-updates |
| **pi-cwd** | Working directory switching |
| **pi-zai-usage** | Z.ai API quota monitor |

## Contributing

Each extension lives in its own repository. Issues, feature requests, and pull requests should be directed to the individual extension repos linked above.

## License

Each extension is individually licensed. See the respective repository for license information.
