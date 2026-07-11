# orchestration

A Claude Code plugin that teaches the **main agent** to orchestrate its subagents deliberately instead of spawning them reflexively.

Delegating to subagents looks easy and goes wrong quietly: an agent spawned for the wrong reason, handed an under-specified prompt, or run in parallel with another that edits the same files. This plugin ships a skill that fires whenever you're about to delegate and walks the orchestrator through the decisions that actually matter.

## What it covers

- **Delegate vs. do it inline** — spawning is the expensive, lossy path; the concrete signals that justify it (fan-out, context hygiene, specialized agents, clean isolation) and the ones that don't ("looks thorough").
- **Self-contained delegation prompts** — a subagent starts cold and can't ask follow-ups, so objective, grounding, boundaries, return contract, and done-criteria all go in up front.
- **Safe parallelism** — batch independent reads, serialize dependencies, and never let two agents write the same files.
- **Picking the right agent** — match the task to the most specific agent, respect read-only vs. write scope, and continue an agent to iterate rather than cold-starting a new one.
- **Owning the results** — verify claims, reconcile parallel outputs, and relay what matters, because the hand-back is lossy and unverified.

## Install

### From the CLI (outside Claude Code)

```bash
# From the GitHub repo
claude plugin marketplace add johnnybs/claude-plugs
claude plugin install orchestration@claude-plugs

# ...or from a local clone
claude plugin marketplace add /path/to/claude-plugs
claude plugin install orchestration@claude-plugs
```

### From within Claude Code

```
/plugin marketplace add johnnybs/claude-plugs
/plugin install orchestration
```

Then restart or reload so the skill registers.

## Usage

The `orchestrator` skill is a **model-invoked** skill — you don't call it with a slash command. Claude reads its description and loads it on its own when a task involves delegating to subagents (deciding whether to spawn, splitting work, fanning out in parallel, writing a delegation prompt, or reconciling what agents return). You can also nudge it explicitly: "use the orchestrator skill before you spawn those agents."

## What's inside

| File | Purpose |
| --- | --- |
| `skills/orchestrator/SKILL.md` | The orchestrator skill — guidance the main agent loads when delegating to subagents. |
| `.claude-plugin/plugin.json` | Plugin manifest. |
