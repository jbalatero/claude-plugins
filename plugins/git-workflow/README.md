# git-workflow

Staging-based Git workflow with conventional commits, PR creation, squash merging, rebasing, and deployment.

## Commands

| Command | Description |
|---------|-------------|
| `/commit` | Create well-formatted conventional commits with emoji |
| `/push-to-target` | Push commits to target branch via cherry-pick (worktree-safe) |
| `/create-pr-merge` | Create PR, squash merge, and optionally update plan file |
| `/rebase-target` | Rebase onto target branch with smart conflict resolution |
| `/deploy` | Full target-to-production deployment pipeline |

## Skills

| Skill | Description |
|-------|-------------|
| `squash-merge-pr` | Squash merge an open PR into the target branch |

## Configuration

Add a `## Git Workflow Config` section to your project's CLAUDE.md:

```markdown
## Git Workflow Config
- Target branch: `staging`
- Plan docs path: `docs/plans/`
- Changelog path: `docs/changelogs/`
- Pre-deploy script: `./pre-deploy.sh`
```

All values are optional with sensible defaults:

| Key | Default | Effect if missing |
|-----|---------|-------------------|
| Target branch | `staging` | Uses `staging` |
| Plan docs path | none | Skips plan file linking |
| Changelog path | none | Skips changelog generation in deploy |
| Pre-deploy script | none | Skips pre-deploy step |

## Installation

```bash
claude plugin add git-workflow@jbalatero
```
