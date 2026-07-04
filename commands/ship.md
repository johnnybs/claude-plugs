---
description: Drive a task all the way to a merged PR — clarify, plan, worktree, implement, test, open PR, wait on CI, fix, merge.
argument-hint: <task description, or a ticket/issue reference>
allowed-tools: Bash, Read, Edit, Write, Glob, Grep, Task, AskUserQuestion, TodoWrite, WebFetch
---

# /ship — drive a task to a merged PR

You are running the **pr-pilot** workflow. Your job is to take the task below and carry it end-to-end until the pull request is **merged** (or until you hit a blocker only the user can resolve).

## The task

$ARGUMENTS

If the task above is empty, ask the user what they want shipped before doing anything else.

## Operating principles

- **Track progress with TodoWrite.** Create one todo per phase below at the start, and keep it updated so the user can see where you are.
- **Never leave the repo in a broken state.** All work happens in a dedicated worktree + branch, never on `main`/`master` directly.
- **Prefer complete work over deferred work.** Don't open a PR with `TODO`s you could have finished. If you must defer something, call it out explicitly in the PR description.
- **Surface blockers early.** If you're stuck on a decision that's genuinely the user's to make, ask with `AskUserQuestion` rather than guessing on something material.
- **Report faithfully.** If tests or CI fail, say so with the real output. Don't claim done until CI is green and the PR is merged.

Work through the phases in order. Do not skip phases.

---

## Phase 0 — Preflight

1. Confirm you're in a git repo: `git rev-parse --show-toplevel`.
2. Confirm the GitHub CLI is available and authenticated: `gh auth status`. If `gh` is missing or unauthenticated, tell the user how to fix it (`gh auth login`) and stop — you can't open/merge a PR without it. Suggest they run it via `! gh auth login`.
3. Check working tree state: `git status --porcelain`. If there are uncommitted changes, ask the user whether to include them, stash them, or abort. Don't silently discard work.
4. Identify the base branch (usually `main` or `master`): `git symbolic-ref refs/remotes/origin/HEAD --short 2>/dev/null` or fall back to the current default. Make sure it's up to date: `git fetch origin`.

## Phase 1 — Understand & clarify (the "find and answer ambiguities" phase)

1. If the task references a ticket/issue (e.g. `#123`, a Linear ID, a URL), fetch its contents (`gh issue view`, WebFetch, or the relevant tool) and read it fully.
2. **Explore the codebase** to ground the task in reality: locate the relevant files, existing patterns, tests, and conventions. For anything beyond a trivial change, dispatch the **`ambiguity-hunter`** agent via Task — it returns task grounding, pre-resolved ambiguities (with evidence), and a short list of genuine open questions. For small changes, do this inline with Grep/Glob/Read.
3. **Hunt for ambiguities.** From that exploration, enumerate what's underspecified: unclear scope, edge cases, competing approaches, naming, backward-compat, migration, config, where new code should live, what "done" means, how it should be tested.
4. **Resolve them in this order:**
   - Answer from the codebase, docs, and conventions wherever you can — don't ask the user things you can determine yourself.
   - For the ones that genuinely need a human decision (material trade-offs, product choices, anything hard to reverse), batch them into `AskUserQuestion` (offer sensible defaults, mark a recommended option first). Ask these **before** writing the plan.
5. Write a concrete **plan**: the approach, the files you'll touch, the test strategy, and how you'll verify. Present it to the user and get a thumbs-up before implementing. Keep it tight — this is a plan to act on, not an essay.

## Phase 2 — Isolated worktree + branch

1. Choose a descriptive branch name: `pr-pilot/<short-kebab-summary>` (or match the repo's existing branch naming convention if there is one).
2. Create a worktree so the main checkout is untouched. Put it beside the repo, not inside it:
   ```bash
   REPO_ROOT="$(git rev-parse --show-toplevel)"
   WT_DIR="$(dirname "$REPO_ROOT")/$(basename "$REPO_ROOT")-<short-kebab-summary>"
   git worktree add -b <branch-name> "$WT_DIR" origin/<base-branch>
   ```
3. **Do all remaining work from inside `$WT_DIR`.** Use absolute paths, or pass `-C "$WT_DIR"` to git. Note the worktree path in a todo so you don't lose track of it.
4. If the repo needs setup (install deps, etc.), do it in the worktree.

## Phase 3 — Implement

1. Make the change following the plan and the repo's existing conventions (match surrounding style, naming, comment density).
2. Keep commits logical. Write clear commit messages. End each commit message with:
   ```
   Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>
   ```
3. Add or update tests to cover the change.

## Phase 4 — Test locally (before spending CI minutes)

1. Discover how this repo tests/lints/builds (check `package.json` scripts, `Makefile`, `pyproject.toml`, CI config in `.github/workflows`, etc.) and run the relevant ones: tests, linter, type-checker, build.
2. If anything fails, **fix it and re-run** until the local checks pass. Don't push known-red code.
3. If there's a `/verify`-style skill or the change has a runtime surface, exercise the actual behavior, not just the tests.

## Phase 5 — Open the PR

1. Commit everything, then push: `git -C "$WT_DIR" push -u origin <branch-name>`.
2. Open the PR with `gh pr create`, targeting the base branch. Write a real description:
   - **What & why** (link the issue with `Closes #123` if applicable).
   - **How it was tested.**
   - Anything deferred or worth a reviewer's attention.
   - End the body with:
     ```
     🤖 Generated with [Claude Code](https://claude.com/claude-code)
     ```
3. Print the PR URL for the user.

## Phase 6 — Wait on CI, fix if red

1. Poll the checks: `gh pr checks <pr> --watch` (or loop `gh pr checks <pr>` with a wait between polls). Give the user a heads-up that you're watching CI.
2. **If a check fails:**
   - Pull the failing job's logs: `gh run view <run-id> --log-failed` (find the run via `gh run list --branch <branch-name>`).
   - Diagnose the real cause, fix it in the worktree, re-run local checks, commit, and push. CI re-runs automatically on push.
   - Repeat until all required checks are green.
   - If a failure is clearly infrastructure/flake (not your code), retry the job (`gh run rerun <run-id> --failed`) once before assuming it's your bug.
3. If you get stuck fixing the same failure repeatedly (≈3 attempts) or a failure needs a human decision, stop and report to the user with the logs — don't thrash.

## Phase 7 — Merge

1. Only merge once **all required checks are green** and any required reviews/approvals are satisfied. Check `gh pr view <pr> --json mergeStateStatus,reviewDecision,statusCheckRollup`.
2. **Merging is outward-facing and hard to reverse — confirm with the user before merging** unless they told you up front to auto-merge. Ask which merge strategy if the repo allows more than one (squash / merge / rebase); default to squash unless the repo convention says otherwise.
3. Merge: `gh pr merge <pr> --squash` (or the chosen strategy). Add `--delete-branch` to clean up the remote branch.

## Phase 8 — Cleanup

1. Remove the worktree: `git worktree remove "$WT_DIR"` (from the main repo dir). Delete the local branch if desired.
2. Update the base branch locally: `git -C "$REPO_ROOT" pull`.
3. Report the final result: PR URL, merge status, and anything the user should know (deferred items, follow-ups).

---

## If something blocks you

Stop and tell the user plainly what's blocking, what you tried, and the concrete options. Leave the worktree and branch intact so work isn't lost. Never fake a green result.
