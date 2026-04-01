---
description: "Create PR from feature branch to target, squash merge, and optionally update plan file with PR number"
allowed-tools:
  [
    "Bash(git add:*)",
    "Bash(git status:*)",
    "Bash(git commit:*)",
    "Bash(git diff:*)",
    "Bash(git log:*)",
    "Bash(git checkout:*)",
    "Bash(git pull:*)",
    "Bash(git push:*)",
    "Bash(git branch:*)",
    "Bash(git rev-parse:*)",
    "Bash(gh pr *)",
    "Read",
    "Edit",
    "Glob",
    "Grep",
  ]
---

## Create PR, Squash Merge, and Optionally Update Plan File

### Configuration

Read the `## Git Workflow Config` section from CLAUDE.md for configuration. If not present, use defaults: target branch = `staging`, plan docs path = none.

### 1. Validate State

*   Confirm current branch is NOT the target branch or `main` (must be a feature branch)
*   Confirm working tree is clean
*   Confirm the branch has commits ahead of target (`git log <target>..HEAD --oneline`)
*   If any check fails, stop and report what's wrong

### 2. Push and Create PR

*   Push current branch to origin with `-u` flag
*   Create a PR targeting the configured target branch using `gh pr create`
*   Use the branch's commit messages to draft the PR title and body
*   Report the PR URL and number

### 3. Squash Merge

*   Squash merge the PR using `gh pr merge <number> --squash`
*   Confirm the merge succeeded

### 4. Update Plan File with PR Number (if plan docs path configured)

*   Only perform this step if `planDocsPath` is configured
*   Get the PR number from step 2
*   Search `<planDocsPath>/*.md` for a design doc matching the feature topic:
    1.  Extract the feature topic from the branch name (e.g., `feature/abandoned-cart` → `abandoned-cart`)
    2.  Match against plan filenames (e.g., `*-abandoned-cart-*.md`)
*   If a matching plan file is found:
    *   Read the file and find the first `#` heading line
    *   Append ` (#NNN)` to that heading
    *   Stage and commit the change with message: `docs: link PR #NNN to plan file`
    *   Push the commit to the target branch
*   If no matching plan file found, skip this step and inform the user

### 5. Clean Up

*   Do NOT checkout the target branch locally — another worktree may have it checked out, which causes `fatal: '<target>' is already used by worktree`
*   The squash merge already updated the target branch on the remote, so no local checkout is needed
*   Report summary: PR number, merge status, plan file updated (if applicable)
