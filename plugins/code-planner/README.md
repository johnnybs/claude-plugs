# code-planner

A Claude Code plugin that turns a coding task into a concrete, **saved** implementation plan — before a line of production code is written.

Given a task, `/plan` will:

1. **Understand** — read any linked issue/ticket and the task itself.
2. **Ground** — explore the codebase (dispatching the `ambiguity-hunter` agent for non-trivial work) to find the relevant files, patterns, tests, and conventions.
3. **Resolve** — hunt every ambiguity, answer what it can from the code, and ask you only the questions that genuinely need a human decision.
4. **Write the plan** — save a structured plan to `.plans/<slug>.md` (goal, approach, files, steps, test strategy, verification, decisions, suggested branch).
5. **Confirm** — present the plan for review; you can edit the file directly.

It **plans only** — it never creates a worktree, writes production code, or opens a PR. The deliverable is the plan file.

## Why a saved plan?

Because the plan outlives the session and can be **handed off**. Plan now, implement later — or let a shipping workflow pick it up. [pr-pilot](../pr-pilot)'s `/ship` detects `.plans/<slug>.md` and skips its own planning phase, so you can split "decide what to do" from "do it and ship it".

## Requirements

- [Claude Code](https://claude.com/claude-code)
- A code repository (git recommended, so plans can reference a base branch — but not required)

## Install

### From the CLI (outside Claude Code)

```bash
# From the GitHub repo
claude plugin marketplace add johnnybs/claude-plugs
claude plugin install code-planner@claude-plugs

# ...or from a local clone
claude plugin marketplace add /path/to/claude-plugs
claude plugin install code-planner@claude-plugs
```

### From within Claude Code

```
/plugin marketplace add johnnybs/claude-plugs
/plugin install code-planner
```

Then restart or reload so the `/plan` command and `ambiguity-hunter` agent register.

## Usage

```
/plan <what you want planned>
```

Examples:

```
/plan add rate limiting to the /login endpoint, 5 attempts per minute
/plan fix #482 — the CSV export drops the last row
/plan https://linear.app/acme/issue/ENG-123
```

The plan lands at `.plans/<slug>.md` (the directory is added to `.gitignore` automatically). Review it, edit it, then implement it yourself or run `/ship <same task>` to build and ship it.

## What's inside

| File | Purpose |
| --- | --- |
| `commands/plan.md` | The `/plan` slash command — the planning workflow that writes the plan file. |
| `agents/ambiguity-hunter.md` | A read-only subagent that grounds the task in the codebase and surfaces the ambiguities to resolve. |
| `.claude-plugin/plugin.json` | Plugin manifest. |
