# claude-plugs

A [Claude Code](https://claude.com/claude-code) plugin marketplace. Each plugin lives in its own directory under [`plugins/`](plugins/); the marketplace descriptor is [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json).

## Plugins

| Plugin | Description |
| --- | --- |
| [`pr-pilot`](plugins/pr-pilot) | Drives a task from idea to a merged PR: clarify ambiguities, plan, worktree, implement, test, open the PR, wait on CI (fixing failures), and merge. |
| [`development-agents`](plugins/development-agents) | Coding-focused subagents — `architecture`, `clean-code`, `performance`, `debugger`, and `websearch`, plus `game-design` and `game-feel` advisors for game development. |

## Install

Add the marketplace once, then install any plugin from it.

### From the CLI (outside Claude Code)

```bash
# From the GitHub repo
claude plugin marketplace add johnnybs/claude-plugs
claude plugin install pr-pilot@claude-plugs

# ...or from a local clone
claude plugin marketplace add /path/to/claude-plugs
claude plugin install pr-pilot@claude-plugs
```

### From within Claude Code

```
/plugin marketplace add johnnybs/claude-plugs
/plugin install pr-pilot
```

Then run `/reload-plugins` (or restart) so the plugin's commands and agents register.

Useful management commands:

```bash
claude plugin list                                  # show installed plugins
claude plugin update pr-pilot                        # pull the latest version
claude plugin uninstall pr-pilot                     # remove it
claude plugin marketplace remove claude-plugs
```

## Adding a new plugin to this marketplace

1. Create a directory under `plugins/<your-plugin>/` containing a `.claude-plugin/plugin.json` manifest and the plugin's `commands/`, `agents/`, `skills/`, or `hooks/`.
2. Add an entry to the `plugins` array in `.claude-plugin/marketplace.json` with `"source": "./plugins/<your-plugin>"`.
3. Commit and push.

See [`plugins/pr-pilot`](plugins/pr-pilot) for a working example.
