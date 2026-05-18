# pi-extensions

A curated collection of extensions and tools for building a powerful [pi coding agent](https://github.com/earendil-works/pi-coding-agent) for software development. Each extension is an independent package that drops into pi via `pi install` and adds new capabilities — from language-server intelligence to process management, workflow orchestration, and web search.

## Extensions

### [pi-acp](https://github.com/harms-haus/pi-acp)

ACP (Agent Client Protocol) agent transport for pi. Exposes a fully ACP-compliant agent over JSON-RPC 2.0 on stdin/stdout, enabling any ACP-compatible client (Zed, VS Code, etc.) to drive the pi coding agent.

### [pi-alibaba-dashscope](https://github.com/harms-haus/pi-alibaba-dashscope)

Alibaba Cloud (DashScope) provider extension. Adds DashScope as a model provider so you can use Qwen and other DashScope-hosted models within pi. Set `DASHSCOPE_API_KEY` and go.

### [pi-cwd](https://github.com/harms-haus/pi-cwd)

Change the effective working directory without restarting the agent. Provides a `/cwd` command with tab-completion, a footer indicator, and session persistence across reloads.

### [pi-git](https://github.com/harms-haus/pi-git)

Rich git status tracking with a live, color-coded footer label and an agent-end summary of changed files. Integrates with [pi-powerline](#pi-powerline) for display.

### [pi-lens](https://github.com/harms-haus/pi-lens)

Unified code quality checks after every file change. Hooks after `write`, `edit`, and `bash` tools to automatically run prettier (report-only), 11 linters (ESLint, Biome, Ruff, Flake8, Pylint, Mypy, Clippy, staticcheck, RuboCop, ShellCheck, Stylelint), LSP diagnostics across 33 languages, and TypeScript `tsc --noEmit` — all concurrently via `Promise.all`. Configurable via `.pi-lens.json`.

### [pi-powerline](https://github.com/harms-haus/pi-powerline)

Centralized powerline status bar. Replaces the built-in footer and consolidates status displays from all extensions into a unified layout — todo counts, workflow phase, active tasks, and more.

### [pi-processes](https://github.com/harms-haus/pi-processes)

Process management tools. Spawn, monitor, kill, and restart long-running processes (dev servers, watchers, API backends) with debounce-based startup detection, log querying, and SIGTERM → SIGKILL escalation.

### [pi-searxng](https://github.com/harms-haus/pi-searxng)

Web search via a self-hosted SearXNG instance. Adds a `web_search` tool so the agent can look up documentation, APIs, and answers online.

### [pi-subagents](https://github.com/harms-haus/pi-subagents)

Spawn parallel sub-agents with live TUI output windows. Each sub-agent runs in its own isolated process with optional named profiles for provider/model, system prompts, and thinking levels.

### [pi-til-done](https://github.com/harms-haus/pi-til-done)

Iterative todo list that auto-loops the agent until every task is complete. Provides `write_todos`, `list_todos`, and `edit_todos` tools with a 3-second auto-continue countdown, 20-iteration circuit breaker, and event-sourced state that survives session restarts.

### [pi-update](https://github.com/harms-haus/pi-update)

Update pi core and all extensions from within the agent. Runs updates in parallel with a `/update` command and auto-reloads when changes are detected.

### [pi-web-content](https://github.com/harms-haus/pi-web-content)

Unified `fetch_content` tool for fetching web pages (converted to clean markdown via Mozilla Readability) and cloning git repositories. Auto-detects URL type and handles both transparently.

### [pi-workflows](https://github.com/harms-haus/pi-workflows)

Define and run named multi-phase workflows with per-phase tool control, subworkflow nesting, state persistence, and auto-continue enforcement. Compose complex agent behaviors from simple phase definitions.

## Quick Start

Install individual extensions with pi:

```bash
pi install git:github.com/harms-haus/<extension-name>
```

For example:

```bash
pi install git:github.com/harms-haus/pi-lens
pi install git:github.com/harms-haus/pi-workflows
```

Then restart pi or run `/reload`.

## Recommended Setup

For a fully-featured software development agent, install these extensions together:

| Extension | Purpose |
|-----------|---------|
| **pi-lens** | Unified code quality checks (linters, LSP diagnostics, prettier, tsc) after every edit |
| **pi-git** | Live git status in the footer |
| **pi-powerline** | Unified status bar |
| **pi-til-done** | Iterative task tracking with auto-continue |
| **pi-subagents** | Parallel sub-agent execution |
| **pi-workflows** | Multi-phase workflow orchestration |
| **pi-processes** | Dev server and process management |
| **pi-web-content** | Web fetching and repo cloning |
| **pi-searxng** | Web search |
| **pi-update** | One-command self-updates |

## Contributing

Each extension lives in its own repository. Issues, feature requests, and pull requests should be directed to the individual extension repos linked above.

## License

Each extension is individually licensed. See the respective repository for license information.
