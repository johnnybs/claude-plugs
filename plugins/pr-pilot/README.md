# pr-pilot

A Claude Code plugin that drives a task all the way from an idea to a **merged pull request** — without you having to babysit each step.

Given a task, it will:

1. **Plan** — delegate to the [code-planner](../code-planner) `/plan` skill: read any linked issue/ticket, explore the codebase, hunt for ambiguities, answer what it can from the code, ask you only the genuine human questions, and save a plan to `.plans/<slug>.md`. If a plan already exists for the task, `/ship` reuses it and **skips planning**.
2. **Isolate** — create a dedicated git **worktree + branch** so your main checkout is never touched.
3. **Implement** — make the change following the repo's conventions, with tests.
4. **Test locally** — run the repo's tests/lint/build and fix failures before spending CI minutes.
5. **Open the PR** — push and open a PR with a real description via `gh`.
6. **Watch CI** — poll the checks, pull failing logs, fix, and push again until green.
7. **Merge** — merge (with your confirmation) and clean up the worktree.

## Requirements

- [Claude Code](https://claude.com/claude-code)
- The [**code-planner**](../code-planner) plugin — pr-pilot delegates its planning phase to code-planner's `/plan` skill (and its `ambiguity-hunter` agent lives there). Install it alongside pr-pilot: `claude plugin install code-planner@claude-plugs`.
- `git` with worktree support
- The GitHub CLI (`gh`), authenticated: `gh auth login`

## Install

### From the CLI (outside Claude Code)

Add the marketplace and install the plugin without starting an interactive session:

```bash
# From the GitHub repo
claude plugin marketplace add johnnybs/claude-plugs
claude plugin install pr-pilot@claude-plugs

# ...or from a local clone
claude plugin marketplace add /path/to/claude-plugs
claude plugin install pr-pilot@claude-plugs
```

Useful management commands:

```bash
claude plugin list                      # show installed plugins
claude plugin update pr-pilot           # pull the latest version
claude plugin uninstall pr-pilot        # remove it
claude plugin marketplace remove claude-plugs
```

The plugin name is `pr-pilot`; the marketplace it comes from is `claude-plugs` (defined in `.claude-plugin/marketplace.json`).

### From within Claude Code

Run the same steps as slash commands in a session:

```
/plugin marketplace add johnnybs/claude-plugs
/plugin install pr-pilot
```

Or, to try it from a local clone:

```
/plugin marketplace add /path/to/claude-plugs
/plugin install pr-pilot
```

Then restart or reload so the `/ship` command and `ambiguity-hunter` agent register.

## Usage

```
/ship <what you want shipped>
```

Examples:

```
/ship add rate limiting to the /login endpoint, 5 attempts per minute
/ship fix #482 — the CSV export drops the last row
/ship https://linear.app/acme/issue/ENG-123
```

### Plan first, ship later

Because planning is delegated to [code-planner](../code-planner), you can split the two:

```
/plan add rate limiting to the /login endpoint, 5 attempts per minute   # writes .plans/rate-limit-login.md
# ...review/edit the plan...
/ship add rate limiting to the /login endpoint, 5 attempts per minute   # finds the plan, skips planning, builds & ships
```

Or point `/ship` straight at a plan file: `/ship .plans/rate-limit-login.md`. Run `/ship <task>` cold and it plans for you first.

The plugin will pause for your input at two points by design: when planning has genuine open questions, and before it merges (unless you tell it up front to auto-merge, e.g. `/ship ... and merge automatically once CI is green`).

## What's inside

| File | Purpose |
| --- | --- |
| `commands/ship.md` | The `/ship` slash command — the end-to-end shipping workflow (delegates planning to code-planner). |
| `.claude-plugin/plugin.json` | Plugin manifest. |
| `.claude-plugin/marketplace.json` | Marketplace descriptor so the repo is installable directly. |

Planning lives in the separate [code-planner](../code-planner) plugin (the `/plan` skill and the `ambiguity-hunter` agent).

## Notes & safety

- All work happens in an isolated worktree/branch — `main` is never edited directly.
- It won't merge without confirmation unless you explicitly ask it to auto-merge.
- If it gets stuck (e.g. the same CI failure ~3 times, or a decision only you can make), it stops and reports with the real logs rather than thrashing or faking a green result.
