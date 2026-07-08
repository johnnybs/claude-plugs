# development-agents

A Claude Code plugin bundling a set of coding-focused **subagents** you can dispatch during development. Each one is read-only and returns a focused report — they investigate and advise; they don't edit your code.

## Agents

| Agent | Focus | Use it when |
| --- | --- | --- |
| `architecture` | Systems and high-level design — module boundaries, layering, dependency direction, code organization, and hierarchies. | You want to evaluate how a codebase or a proposed change is *structured*, or decide where new code belongs. |
| `clean-code` | Micro-level quality — function size and responsibility, function/variable naming, readability, and small local refactors. | You want a close-up review of a function or file's craftsmanship, not its design. |
| `websearch` | Coding-focused web research — library/API docs, error messages, version differences, and current best practices, with sources. | Answering the task correctly needs up-to-date external information the codebase can't provide. |

The `architecture` and `clean-code` agents are deliberately complementary: architecture works top-down on structure, clean-code works bottom-up on readability. Reach for both on a substantial change.

## Requirements

- [Claude Code](https://claude.com/claude-code)

## Install

### From the CLI (outside Claude Code)

```bash
# From the GitHub repo
claude plugin marketplace add johnnybs/claude-plugs
claude plugin install development-agents@claude-plugs

# ...or from a local clone
claude plugin marketplace add /path/to/claude-plugs
claude plugin install development-agents@claude-plugs
```

Useful management commands:

```bash
claude plugin list                              # show installed plugins
claude plugin update development-agents          # pull the latest version
claude plugin uninstall development-agents        # remove it
```

### From within Claude Code

```
/plugin marketplace add johnnybs/claude-plugs
/plugin install development-agents
```

Then restart or reload so the agents register.

## Usage

Once installed, ask Claude Code to use an agent by name, or let it pick one automatically when a task matches. For example:

```
Use the architecture agent to review how the payments module is organized.
Have the clean-code agent look over the functions in src/parser.ts.
Ask the websearch agent whether this API is deprecated in the version we're on.
```

The agents are prefixed with the plugin name when dispatched (e.g. `development-agents:architecture`).

## What's inside

| File | Purpose |
| --- | --- |
| `agents/architecture.md` | Reviews systems-level and high-level design, boundaries, and code organization. |
| `agents/clean-code.md` | Reviews function size, naming, readability, and small local refactors. |
| `agents/websearch.md` | Researches coding questions on the web and returns sourced answers. |
| `.claude-plugin/plugin.json` | Plugin manifest. |
