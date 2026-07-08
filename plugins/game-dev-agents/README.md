# game-dev-agents

A Claude Code plugin bundling **game-development subagents**. Each one is read-only and returns a focused report — they investigate and advise; they don't edit your code. It's a companion to [`development-agents`](../development-agents) (general coding agents).

## Agents

`game-design` and `game-feel` are engine-agnostic — they reason about the design and the parameters behind it, not any one framework's API.

| Agent | Focus | Use it when |
| --- | --- | --- |
| `game-design` | Mechanics, systems, balance, progression curves, economy, and difficulty/pacing. | You're deciding how a mechanic should work or why it feels off, tuning numbers, or shaping progression. |
| `game-feel` | Game feel and "juice" — input responsiveness, control tuning, feedback (visual/audio/haptic), animation, and camera. | Something works but feels floaty, mushy, unresponsive, or flat, and you want it tight and satisfying. |

`game-design` is about whether the rules are *fun and fair*; `game-feel` is about whether the same action *feels good to touch*. They pair well.

The next three are **Ebiten/Go-specific** — they know Ebiten's rendering, audio, and update-loop APIs.

| Agent | Focus | Use it when |
| --- | --- | --- |
| `gameplay-systems` | ECS and data-oriented gameplay architecture — entity/component/system boundaries, storage, system scheduling, and the `Update` loop. | You're structuring how entities, components, and systems fit together, or moving ad-hoc game objects toward ECS. |
| `shader-graphics` | 2D rendering and Kage shaders — draw-call batching, texture atlases, the `DrawImage`/`DrawTriangles` pipeline, blend modes, and offscreen targets. | Rendering is slow or looks wrong, or you're writing/optimizing a Kage shader or effect. |
| `audio` | Ebiten audio — the `audio.Context` lifecycle, players, decoding/streaming, looping, mixing, and avoiding per-frame allocations. | You're adding sound/music, audio glitches or lags, or many concurrent sounds need managing. |

## Requirements

- [Claude Code](https://claude.com/claude-code)

## Install

### From the CLI (outside Claude Code)

```bash
# From the GitHub repo
claude plugin marketplace add johnnybs/claude-plugs
claude plugin install game-dev-agents@claude-plugs

# ...or from a local clone
claude plugin marketplace add /path/to/claude-plugs
claude plugin install game-dev-agents@claude-plugs
```

Useful management commands:

```bash
claude plugin list                           # show installed plugins
claude plugin update game-dev-agents          # pull the latest version
claude plugin uninstall game-dev-agents        # remove it
```

### From within Claude Code

```
/plugin marketplace add johnnybs/claude-plugs
/plugin install game-dev-agents
```

Then restart or reload so the agents register.

## Usage

Once installed, ask Claude Code to use an agent by name, or let it pick one automatically when a task matches. For example:

```
Ask the game-design agent whether the upgrade curve ramps too fast.
Have the game-feel agent tell me why the jump feels floaty.
Ask the gameplay-systems agent how to structure my entities and systems.
Have the shader-graphics agent find why my sprites aren't batching.
Ask the audio agent why sound effects stutter when many fire at once.
```

The agents are prefixed with the plugin name when dispatched (e.g. `game-dev-agents:game-feel`).

## What's inside

| File | Purpose |
| --- | --- |
| `agents/game-design.md` | Advises on mechanics, balance, progression, and economy. |
| `agents/game-feel.md` | Advises on input responsiveness, feedback/juice, animation, and camera feel. |
| `agents/gameplay-systems.md` | Advises on ECS and data-oriented gameplay architecture for Ebiten/Go. |
| `agents/shader-graphics.md` | Advises on Ebiten 2D rendering, batching, and Kage shaders. |
| `agents/audio.md` | Advises on Ebiten audio: context, players, decoding, looping, and mixing. |
| `.claude-plugin/plugin.json` | Plugin manifest. |
