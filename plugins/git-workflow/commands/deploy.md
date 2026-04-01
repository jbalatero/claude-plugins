---
description: "Automates the full target-to-production deployment: optional changelog generation, optional pre-deploy build, and rebase to main"
allowed-tools:
  [
    "Bash(git add:*)",
    "Bash(git status:*)",
    "Bash(git commit:*)",
    "Bash(git diff:*)",
    "Bash(git log:*)",
    "Bash(git checkout:*)",
    "Bash(git pull:*)",
    "Bash(git rebase:*)",
    "Bash(git push:*)",
    "Bash(git branch:*)",
    "Bash(git rev-parse:*)",
    "Bash(git merge-base:*)",
    "Bash(gh pr *)",
    "Read",
    "Write",
    "Glob",
    "Grep",
  ]
---

## Deploy to Production

Automates the full target branch → main deployment pipeline. Run this command from the target branch with a clean working tree.

### Configuration

Read the `## Git Workflow Config` section from CLAUDE.md for configuration. If not present, use defaults: target branch = `staging`. Optional config:
- `changelogPath` — if set, generates changelogs (e.g., `docs/changelogs/`)
- `preDeployScript` — if set, runs before deploy (e.g., `./pre-deploy.sh`)
- `planDocsPath` — if set, checks for plan docs to source changelog content

## Steps

### 1. Validate State

*   Confirm current branch is the target branch
*   Confirm working tree is clean (no uncommitted changes)
*   Confirm target is ahead of main (`git log main..<target> --oneline`)
*   If any check fails, stop and report what's wrong

### 2. Generate Changelog (if changelogPath configured)

*   Get PRs merged since the last deploy: extract PR numbers from `git log main..<target> --oneline`
*   For each PR, gather source material in this priority order:
    1.  If `planDocsPath` is configured, check for a design doc matching the PR number or feature topic
    2.  **If no design doc, analyze the actual code diffs** for that PR's commits to understand what was implemented
    3.  Optionally check `gh pr view <number>` for hints from the PR title and body
*   Determine the next changelog filename:
    *   Glob `<changelogPath>/YYYY-MM-DD.*.md` for today's date to find existing files
    *   Next number = count of matches + 1, zero-padded to 3 digits (e.g., `.001`, `.002`, `.003`)
    *   Filename: `<changelogPath>/YYYY-MM-DD.NNN.md`
*   Create the changelog file:
    *   `# Release YYYY-MM-DD.NNN`
    *   `> summary line` (under 200 chars, no double quotes) summarizing all PRs — plain language understandable by non-technical people
    *   Reference lines as additional `>` blockquotes after the summary, one per PR (if PR numbers available)
    *   One `## feat/fix(scope): Description (#NNN)` section per PR with overview, key changes, and notable decisions
    *   Always add a `## Rollback` section at the end
*   **Always generate a changelog** if there are code changes in the diff. Only skip if the diff contains exclusively non-functional changes.

### 3. Run Pre-Deploy (if preDeployScript configured)

*   Execute the configured pre-deploy script
*   If it fails, stop and report the error

### 4. Commit

*   Stage the changelog file (if generated) and any lockfile changes from pre-deploy
*   Commit with message: `build: pre-deploy and changelog for YYYY-MM-DD.NNN release`
*   If only lockfile changes (no changelog), use: `build: pre-deploy`
*   If only changelog (no lockfile changes), use: `docs: add changelog for YYYY-MM-DD.NNN release`

### 5. Rebase to Main

*   `git checkout main`
*   `git pull origin main`
*   `git rebase <target>` (fast-forwards main to target tip, keeping linear history)
*   `git push origin main`
*   If rebase fails due to conflicts, abort the rebase, return to target branch, and report the issue

### 6. Return to Target Branch

*   `git checkout <target>`
*   Report success with a summary of what was deployed (PR numbers, changelog file path if applicable)

## Error Handling

*   Stop at the first failure and report clearly what went wrong
*   Never force-push
*   If step 5 fails, abort the rebase (`git rebase --abort`), checkout target branch, and report
*   Always leave the repo in a clean state on the branch the user started from
