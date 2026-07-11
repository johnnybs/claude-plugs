---
name: orchestrator
description: Use when a task involves delegating to subagents — deciding whether to spawn one at all, splitting work across several, running agents in parallel, writing the delegation prompt, picking the right agent type, or making sense of what agents return. Teaches the main agent to orchestrate subagents deliberately instead of spawning reflexively. Triggers on "use a subagent", "spawn agents", "delegate this", "run these in parallel", "fan out", "have an agent look into…", or any moment you're about to call the Agent/Task tool.
---

# Orchestrating subagents

You are the **orchestrator**. Subagents are workers you dispatch to do bounded pieces of a job. This skill is about doing that *well* — most bad orchestration is either spawning when you shouldn't, or spawning without giving the worker what it needs to succeed.

Two facts drive everything below:

1. **A subagent starts cold.** It does not see your conversation, your context, the user's earlier messages, or what other agents are doing. It sees only the prompt you hand it. It cannot ask you a follow-up mid-task.
2. **You get back only its final message.** Its tool calls, intermediate reasoning, and the files it read are gone. Whatever you didn't get it to say in that final message, you don't have.

Everything that follows is a consequence of those two facts.

## First decide: delegate, or do it inline?

Spawning is the expensive path — a cold start, a full round-trip, and a lossy hand-back. Default to doing the work yourself. Delegate only when at least one of these clearly holds:

- **Fan-out.** There are several *independent* pieces that can run at once and you'd otherwise do them one after another (search five subsystems, review four files, investigate three hypotheses).
- **Context hygiene.** The work involves reading a large volume of material you don't need to keep — a wide code search, a sprawling investigation. Let an agent burn its own context and hand you back the conclusion.
- **Specialized expertise.** A purpose-built agent exists for exactly this (a reviewer, a debugger, a test-writer). Its instructions and tool scope make it better at the job than a generic pass.
- **Isolation.** The subtask is self-contained with a crisp interface, and you can describe "done" without needing to watch the intermediate steps.

Do **not** delegate when:

- The task is small enough that writing the prompt costs more than doing it.
- You need tight, iterative back-and-forth — an agent can't converse mid-task, so a job that needs five course-corrections becomes five cold starts.
- You need to see the raw material yourself (you'll have to read the files anyway to act on the result).
- It's the *only* thing to do and there's nothing to parallelize — a lone agent doing your one task just adds a round-trip and loses fidelity.

> A task described as "thorough," "multi-angle," or having several parts is **not** automatically a delegation. Handle multi-part work inline unless the parts are genuinely independent and you gain from running them at once. Never spawn an agent to *look* busy or to appear rigorous.

## Write a delegation prompt that can't fail quietly

Because the agent is cold and can't ask questions, the prompt must be self-contained. Include, every time:

- **Objective** — the one outcome, stated first, in a sentence.
- **Grounding** — the specific files, paths, symbols, commands, or facts it needs. Don't make it rediscover what you already know; paste the `path:line` references, the repro command, the relevant constraint.
- **Boundaries** — what's in scope and, explicitly, what's out. For read-only work say "do not modify anything." For write work name the exact files it may touch.
- **Return contract** — the shape of the answer you want back: a ranked list, a root cause with evidence, a diff summary, specific fields. You only get the final message, so specify what that message must contain. Ask for `path:line` citations when you'll need to act on them.
- **Done criteria** — how it knows it's finished (tests pass, the cause is proven, every file reviewed).

A weak prompt ("look into the auth bug") returns a weak, unactionable summary. A strong prompt ("Find the root cause of the 401 on `/login` after token refresh; start at `auth/refresh.go:88`; return the exact file:line where the token is dropped, the mechanism, and the minimal fix location; do not change code") returns something you can immediately use.

## Parallelize the independent, serialize the dependent

- **Independent subtasks → one batch.** Issue the spawns together so they run concurrently. Read-only fan-out (searching, reviewing, investigating) is the safe, high-value case — parallelize freely.
- **Dependent subtasks → serialize.** If B needs A's output, you can't spawn them together; run A, read its result, then shape B's prompt from it.
- **Concurrent writes are the trap.** Two agents editing the same files will clobber each other. If you must parallelize write work, give each a **disjoint** set of files and integrate the results yourself. When in doubt, keep writes sequential — or keep the writing yourself and let agents only investigate.
- **Don't over-decompose.** Five agents whose outputs you have to stitch back together can cost more than doing the whole thing linearly. Split along seams that are actually independent, not arbitrarily.

## Match the task to the agent

Pick the most specific agent whose description fits the task; fall back to a general-purpose or exploration agent for open-ended search. Respect tool scope: read-only agents (reviewers, investigators, researchers) **advise** — they don't write, so plan to apply their findings yourself or route the change to a write-capable agent. Don't hand a write task to a read-only agent and expect a change; you'll get a report.

**Continue vs. re-spawn.** A fresh `Agent`/`Task` call always cold-starts. To iterate on a worker's own output — "now also handle the edge case you found" — continue the *same* agent (e.g. via `SendMessage`) so it keeps its context, rather than spawning a new one that has to rediscover everything. Reuse the context you've already paid for.

## Own the results

The hand-back is lossy and unverified — treat it that way.

- **Verify, don't trust.** An agent can report success it didn't achieve, or miss the real issue. Spot-check claims against the actual code or by running the thing before you build on them.
- **Reconcile parallel outputs.** When several agents return, resolve overlaps and contradictions yourself — they didn't see each other's work. You are the integration point.
- **Relay what matters.** The user never saw the agent's output; only you did. Surface the findings and decisions, not "the agent finished."
- **Don't thrash.** If an agent comes back empty or wrong, a *better prompt* usually beats re-spawning the same one. Re-running an under-specified task just reproduces the under-specified result.

## Background agents

If agents run in the background, the harness re-invokes you when they finish — don't burn turns polling. Keep making progress on work that doesn't depend on their results, and integrate each one as it lands. Only block and wait when you genuinely can't proceed without an outstanding result.

## The short version

Delegate for fan-out, context hygiene, expertise, or clean isolation — not to look thorough. Give each agent everything it needs up front, because it's cold and mute. Parallelize independent reads; serialize dependencies; never let two agents write the same files. Match the task to the right agent, and continue an agent to iterate instead of re-spawning. Then verify, reconcile, and relay — you're the orchestrator, so the integration and the truth-checking are yours.
