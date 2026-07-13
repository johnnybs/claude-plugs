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

## Phase 1 — Plan (delegated to code-planner)

Planning is owned by the **code-planner** plugin, not by this workflow. `/ship` either reuses an existing plan or produces one via `/plan`, then implements against it. Plans live at `.plans/<slug>.md`.

1. **Look for an existing plan.**
   - If the user pointed at a specific plan file (e.g. `/ship .plans/rate-limit-login.md`), use that.
   - Otherwise derive the same kebab-case **slug** the planner would from the task, and check for `.plans/<slug>.md`. If the task is empty and one or more plans exist under `.plans/`, list them and ask the user which to ship.
2. **If a plan exists:** read it fully. Confirm with the user in one line that you're shipping this plan (show its path and goal), then **skip straight to Phase 2** — do not re-plan. If it's clearly stale relative to the current code, say so and offer to re-run planning.
3. **If no plan exists:** run the planning workflow now by invoking the **`code-planner` `/plan` skill** with this task (via the Skill tool). It will ground the task, resolve ambiguities, ask you any genuine open questions, and write `.plans/<slug>.md`. Once it reports a saved plan, read that file and continue.
   - If the `code-planner` plugin/`/plan` skill isn't available, tell the user it's a dependency (`claude plugin install code-planner@claude-plugs`) and, with their okay, fall back to planning inline here: explore the codebase (dispatch the `ambiguity-hunter` agent if installed), resolve ambiguities, ask only genuine human questions via `AskUserQuestion`, and write the plan to `.plans/<slug>.md` in the same format before proceeding.
4. Get the user's thumbs-up on the plan before implementing.

## Phase 2 — Isolated worktree + branch

1. Use the plan's **Suggested branch** (or its slug) as the branch name; otherwise choose a descriptive `pr-pilot/<short-kebab-summary>`. Either way, match the repo's existing branch naming convention if there is one.
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
