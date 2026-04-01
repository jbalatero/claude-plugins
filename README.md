# claude-plugins

A collection of Claude Code plugins for development workflows, incident response, and process guardrails.

## Plugins

| Plugin | Description |
|--------|-------------|
| [git-workflow](plugins/git-workflow/) | Staging-based Git workflow with conventional commits, PR creation, squash merging, rebasing, and deployment |
| [incident-response](plugins/incident-response/) | Incident investigation, lifecycle management, escalation tracking, and change logging |
| [dev-process](plugins/dev-process/) | Pre-implementation review, session retrospectives, codebase rule discovery, and GitHub Actions review |

## Installation

```bash
# Register marketplace (one-time)
/plugin marketplace add jbalatero/claude-plugins

# Install individual plugins
/plugin install git-workflow@jbalatero
/plugin install incident-response@jbalatero
/plugin install dev-process@jbalatero
```

## Local Development

```bash
# Test a single plugin without installing
claude --plugin-dir ~/Projects/claude-plugins/plugins/git-workflow

# Or register the local directory as a marketplace
/plugin marketplace add ~/Projects/claude-plugins
```

## Plugin Details

### git-workflow

Streamlines Git workflow with configurable target branches, conventional commits, PR creation with squash merge, smart rebasing with conflict resolution, and deployment automation.

**Configuration:** Add a `## Git Workflow Config` section to your project's CLAUDE.md. See [plugin README](plugins/git-workflow/README.md) for details.

### incident-response

Incident investigation and lifecycle management via the `incident-response` MCP server. Includes structured report synthesis, escalation tracking, and infrastructure change logging for future correlation.

**Prerequisites:** Requires the `incident-response` MCP server.

### dev-process

Development workflow guardrails that improve code quality through pre-implementation reviews, automated session retrospectives, codebase convention discovery, and GitHub Actions linting.

**Configuration:** None required — all tools work with zero setup.

## Author

**jbalatero**
