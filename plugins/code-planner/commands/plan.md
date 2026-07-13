---
description: Turn a coding task into a concrete, saved implementation plan — ground it in the codebase, resolve ambiguities, ask only what needs a human, and write a reviewable plan file.
argument-hint: <task description, or a ticket/issue reference>
allowed-tools: Bash, Read, Edit, Write, Glob, Grep, Task, AskUserQuestion, TodoWrite, WebFetch
---

# /plan — turn a task into a saved implementation plan

You are running the **code-planner** workflow. Your job is to take the task below, understand it deeply against the actual codebase, resolve every ambiguity you can, ask the human only what genuinely needs their decision, and **write a concrete plan to a file** that a later implementation pass (your own, or a workflow like pr-pilot's `/ship`) can execute without re-planning.

**You plan; you do not implement.** Do not create a worktree, write production code, or open a PR. The single deliverable is a saved plan file plus a short summary to the user.

## The task

$ARGUMENTS

If the task above is empty, ask the user what they want planned before doing anything else.

## Operating principles

- **The output is a file, not just chat.** The plan must be written to `.plans/<slug>.md` so it survives the session and can be handed off. Chat-only output does not count as done.
- **Answer from the code before asking the human.** Resolve everything you can from the codebase, docs, and conventions. Only escalate genuine product/trade-off/hard-to-reverse decisions.
- **Ground everything in evidence.** Cite real files as `path:line`. Don't plan against an imagined codebase.
- **Keep it tight and actionable.** The plan is something to execute, not an essay.

---

## Phase 1 — Understand the task

1. If the task references a ticket/issue (e.g. `#123`, a Linear ID, a URL), fetch its contents (`gh issue view`, WebFetch, or the relevant tool) and read it fully.
2. Confirm you're in a code repository (`git rev-parse --show-toplevel`, or note the working directory if it's not a git repo — planning still works, it just can't cite a base branch).

## Phase 2 — Ground the task in the codebase

1. **Explore** to locate the relevant files, existing patterns, tests, and conventions the change must match.
2. For anything beyond a trivial change, dispatch the **`ambiguity-hunter`** agent via Task — it returns task grounding, pre-resolved ambiguities (with evidence), and a short list of genuine open questions. For small changes, do this inline with Grep/Glob/Read.

## Phase 3 — Hunt and resolve ambiguities

1. **Enumerate what's underspecified:** scope boundaries, edge cases, error/empty/null handling, competing approaches, naming, backward-compat, migration, config/flags, where new code should live, performance/security constraints, and what "done" and "tested" mean here.
2. **Resolve in this order:**
   - Answer from the codebase, docs, and conventions wherever you can — don't ask the user things you can determine yourself.
   - For the ones that genuinely need a human decision, batch them into `AskUserQuestion` (offer sensible defaults, mark a recommended option first).

## Phase 4 — Write the plan file

1. Derive a short kebab-case **slug** from the task (e.g. `rate-limit-login`). Keep it stable — a later `/ship <same task>` will look for this exact file.
2. Ensure the plan directory is ignored by git so plans don't get committed by accident:
   - If a `.gitignore` exists and doesn't already ignore `.plans/`, append a `.plans/` line to it.
   - Create `.plans/` if needed.
3. Write the plan to **`.plans/<slug>.md`** using the structure below. Overwrite an existing file for the same slug only after telling the user you're replacing it.

```markdown
---
task: <one-line restatement of the task>
slug: <slug>
source: <ticket/issue/URL if any, else "ad-hoc">
status: ready
created: <YYYY-MM-DD>
---

# Plan: <short title>

## Goal
<what we're building and why — 2-4 sentences>

## Approach
<the chosen approach, and briefly why over alternatives>

## Files to touch
- `path/to/file.ext` — <what changes here>
- ...

## Steps
1. <ordered, concrete implementation steps>
2. ...

## Test strategy
<what tests to add/update, which framework, how to run them>

## Verification
<how to confirm it actually works end-to-end beyond tests>

## Decisions
<resolved ambiguities and the answers — including anything the human chose in Phase 3, with evidence/reasoning>

## Open / deferred
<anything intentionally out of scope or left for follow-up; "none" if none>

## Suggested branch
`<suggested branch name, e.g. pr-pilot/rate-limit-login>`
```

## Phase 5 — Present and confirm

1. Show the user the plan path (`.plans/<slug>.md`) and a tight summary of the approach and any decisions they made.
2. Invite edits — they can tweak the file directly or tell you to adjust it.
3. Make clear the next step: they can implement from this plan now, or hand it to a shipping workflow (e.g. `/ship`) which will detect the plan and skip re-planning.

Do not proceed to implementation. Planning ends here.
