---
description: "Rebase current branch onto target, resolve simple conflicts, ask for complex ones, then force push"
allowed-tools:
  [
    "Bash(git status:*)",
    "Bash(git diff:*)",
    "Bash(git log:*)",
    "Bash(git fetch:*)",
    "Bash(git rebase:*)",
    "Bash(git checkout:*)",
    "Bash(git add:*)",
    "Bash(git commit:*)",
    "Bash(git push:*)",
    "Bash(git rev-parse:*)",
    "Bash(git branch:*)",
    "Bash(git merge-base:*)",
    "Bash(pnpm:*)",
    "Read",
    "Edit",
    "Write",
    "Grep",
    "Glob",
  ]
---

## Rebase onto Target Branch and Force Push

Rebases the current feature branch onto the target branch, resolves straightforward conflicts automatically, escalates complex conflicts to the user, and force pushes.

### Configuration

Read the `## Git Workflow Config` section from CLAUDE.md for configuration. If not present, use defaults: target branch = `staging`.

**Precondition:** Working directory is clean and all changes are committed.

### 1. Validate State

*   Confirm current branch is NOT the target branch or `main`
*   Confirm working tree is clean (`git status --porcelain` is empty)
*   Check for an in-progress rebase: if `.git/rebase-merge/` or `.git/rebase-apply/` exists, stop and report — a previous rebase was interrupted and must be resolved or aborted manually before proceeding
*   If any check fails, stop and report

### 2. Fetch and Rebase

*   `git fetch origin <target>`
*   `git rebase origin/<target>`
*   If rebase succeeds with no conflicts, skip to step 4

### 3. Resolve Conflicts

For each conflict pause during rebase:

1.  Run `git diff --name-only --diff-filter=U` to list conflicted files
2.  Read each conflicted file and examine the conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`)
3.  Classify each conflict:

**Simple (resolve automatically):**

*   Import statement ordering or additions
*   Adjacent but non-overlapping line additions (e.g., both sides added different items to a list)
*   Whitespace, formatting, or comment-only differences
*   Lock file conflicts (e.g., `pnpm-lock.yaml`) — accept theirs and re-run package install after rebase completes
*   Auto-generated files — accept theirs and re-run generation after rebase completes
*   One side deleted a line the other side didn't touch meaningfully
*   Changelog or documentation additions where both sides added content (concatenate both)

## Very Important: Don't be afraid to ask for help

If you are not sure how a conflict should be resolved, stop and ask me for direction or clarification.

1. Show me the conflicting chunks
2. Explain what you believe you should do
3. Wait for confirmation

**Complex (ask the user):**

*   Overlapping edits to the same function or logic block
*   Business logic changes where both sides modified behavior
*   Type signature changes that affect downstream code
*   Database migration conflicts
*   Any conflict where the correct resolution isn't obvious

4.  For simple conflicts: edit the file to resolve, remove conflict markers, `git add <file>`
5.  For complex conflicts: show the user the conflicted hunks with surrounding context (10 lines before and after), explain what each side changed, and ask which resolution they prefer. After the user decides, apply their choice and `git add <file>`
6.  After all files in the current step are resolved, run `git rebase --continue`
    *   If git reports "No changes — did you forget to use 'git add'?" (the commit became empty), run `git rebase --skip` instead
7.  Repeat until rebase completes or user aborts

If the user chooses to abort at any point: `git rebase --abort` and stop.

### 4. Post-Rebase Checks

*   If lock files were conflicted, re-run package install. If files changed, stage and amend the last rebase commit
*   If auto-generated files were conflicted, re-run the generation command. If files changed, stage and amend the last rebase commit
*   Confirm rebase completed: run `git log --oneline origin/<target>..HEAD` and show the output to the user so they can verify the expected commits landed

### 5. Force Push

*   `git push --force-with-lease --force-if-includes`
*   If the push fails because the remote was updated since the last fetch (lease mismatch), re-fetch with `git fetch origin <target>` and retry the push once. If it fails again, stop and report — do not force-push blindly
*   Report success with the branch name and number of commits pushed
