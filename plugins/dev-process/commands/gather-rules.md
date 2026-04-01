---
description: "Explore codebase, discover conventions and patterns, and save as path-scoped claude rules"
allowed-tools:
  [
    "Read",
    "Glob",
    "Grep",
    "Agent",
    "Bash(git diff:*)",
    "Bash(git status:*)",
    "Bash(git add:*)",
    "Bash(git commit:*)",
    "Bash(git log:*)",
  ]
---

## Gather Claude Rules

Explore the codebase to discover conventions, architectural patterns, and implicit rules that are not yet captured in `.claude/rules/`. Present findings for approval, then save as properly scoped rule files.

### Input

`$ARGUMENTS` — optional focus area. If empty, perform a broad sweep. Examples:

- `API patterns` — controllers, services, DTOs, route registration
- `testing conventions` — test framework patterns, mock factories, fixtures
- `state management` — store patterns, data flow, caching
- `database` — ORM conventions, migrations, query patterns
- `frontend` — component patterns, styling, routing
- `error handling` — error classes, middleware, logging
- `CI/CD` — build pipeline, deployment patterns
- `authentication` — auth middleware, permission patterns

### 1. Read Existing Rules

- Glob `.claude/rules/*.md` and read every file
- Build an inventory of already-covered topics so you never create duplicates
- Note the path scoping used in each existing rule

### 2. Explore the Codebase

- Use Agent (subagent_type: Explore) to analyze the codebase broadly, or narrowed by `$ARGUMENTS`
- Look for patterns that aren't obvious from reading the code once — things that would cause bugs or wasted time if not known upfront
- **What to look for** (general pattern categories):
  - Recurring code patterns (naming, structure, error handling)
  - Architectural conventions (module boundaries, data flow)
  - Configuration conventions (env vars, config files, build tools)
  - Security and access control patterns
  - API design patterns (request/response, serialization, validation)
  - Database and ORM conventions
  - Deployment and infrastructure patterns
  - Testing patterns and conventions
  - Frontend component and state management patterns
- Skip anything already covered by an existing rule (from step 1) or already in CLAUDE.md
- Aim for at least 5 rules, up to 15, depending on focus area

### 3. Present Findings

Show the user a numbered table with these columns:

| # | Proposed Filename | Paths Scope | Summary |
|---|-------------------|-------------|---------|

For each proposed rule, also show the full content that would be written to the file (including frontmatter).

**Path scoping guidelines** (per https://code.claude.com/docs/en/memory#path-specific-rules):
- Rules that apply only when working with specific directories get `paths:` frontmatter
- Rules that are universal (git workflow, infra, architectural decisions) have NO paths frontmatter
- Cross-cutting rules that span multiple areas list all relevant path patterns
- Use glob patterns matching the project's directory structure

**Rule quality criteria:**
- Each rule should capture something non-obvious that would cause a bug, wasted effort, or style violation if not known
- Avoid restating what's already in CLAUDE.md — rules should capture patterns learned from the code itself
- Include concrete code examples where they prevent misuse
- Name files with kebab-case matching the topic (e.g., `sequelize-model-conventions.md`)

**STOP HERE.** Do not write any files until the user explicitly approves. The user may:
- Approve all
- Remove items by number
- Request edits to specific items
- Ask for more exploration in a specific area

### 4. Write Approved Rules

For each approved rule, write to `.claude/rules/<filename>.md` with this format:

```markdown
---
paths:
  - "<glob pattern>"
---

# <Rule Title>

## <Section heading>

<Content — concise, actionable, pattern-focused>
```

- Omit the `paths:` frontmatter block entirely for unconditional rules
- Use clear markdown with code examples where they add clarity
- Keep each rule focused on one topic — split broad findings into separate files

### 5. Offer to Commit

After writing all files, ask the user if they want to commit. If yes, use conventional commit format:

```
docs: add claude rules for <brief summary of topics>
```

Stage only the `.claude/rules/*.md` files that were created or modified.
