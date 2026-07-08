# development-agents

A Claude Code plugin bundling a set of coding-focused **subagents** you can dispatch during development. Each one is read-only and returns a focused report — they investigate and advise; they don't edit your code.

## Agents

### General

| Agent | Focus | Use it when |
| --- | --- | --- |
| `architecture` | Systems and high-level design — module boundaries, layering, dependency direction, code organization, and hierarchies. | You want to evaluate how a codebase or a proposed change is *structured*, or decide where new code belongs. |
| `clean-code` | Micro-level quality — function size and responsibility, function/variable naming, readability, and small local refactors. | You want a close-up review of a function or file's craftsmanship, not its design. |
| `performance` | Hot paths, per-frame allocations, GC pressure, frame-budget offenders, and algorithmic cost — measured, then prioritized. | Something is slow, stutters, or drops frames, or you're about to optimize and want the real bottleneck first. |
| `debugger` | Root-cause investigation for a specific bug, crash, stack trace, or failing test, traced through the code with evidence. | Something is broken and you need the real "why" and fix location before changing code. |
| `websearch` | Coding-focused web research — library/API docs, error messages, version differences, and current best practices, with sources. | Answering the task correctly needs up-to-date external information the codebase can't provide. |

The `architecture` and `clean-code` agents are deliberately complementary: architecture works top-down on structure, clean-code works bottom-up on readability. Reach for both on a substantial change.

### Game development

Engine-agnostic — they reason about the design and the parameters behind it, not any one framework's API.

| Agent | Focus | Use it when |
| --- | --- | --- |
| `game-design` | Mechanics, systems, balance, progression curves, economy, and difficulty/pacing. | You're deciding how a mechanic should work or why it feels off, tuning numbers, or shaping progression. |
| `game-feel` | Game feel and "juice" — input responsiveness, control tuning, feedback (visual/audio/haptic), animation, and camera. | Something works but feels floaty, mushy, unresponsive, or flat, and you want it tight and satisfying. |

`game-design` is about whether the rules are *fun and fair*; `game-feel` is about whether the same action *feels good to touch*. They pair well.

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
Ask the performance agent why the render loop drops frames on big levels.
Have the debugger agent root-cause the crash in this stack trace.
Ask the game-design agent whether the upgrade curve ramps too fast.
Have the game-feel agent tell me why the jump feels floaty.
```

The agents are prefixed with the plugin name when dispatched (e.g. `development-agents:architecture`).

## What's inside

| File | Purpose |
| --- | --- |
| `agents/architecture.md` | Reviews systems-level and high-level design, boundaries, and code organization. |
| `agents/clean-code.md` | Reviews function size, naming, readability, and small local refactors. |
| `agents/performance.md` | Finds hot paths, allocations, GC pressure, and frame-budget offenders — measured and prioritized. |
| `agents/debugger.md` | Root-causes a bug/crash/failing test and points to the fix, with evidence. |
| `agents/websearch.md` | Researches coding questions on the web and returns sourced answers. |
| `agents/game-design.md` | Advises on mechanics, balance, progression, and economy. |
| `agents/game-feel.md` | Advises on input responsiveness, feedback/juice, animation, and camera feel. |
| `.claude-plugin/plugin.json` | Plugin manifest. |
