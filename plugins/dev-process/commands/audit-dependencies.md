---
description: "Audit project dependencies for security vulnerabilities and outdated packages"
allowed-tools:
  [
    "Bash(npm audit:*)",
    "Bash(npm outdated:*)",
    "Bash(npx:*)",
    "Bash(pnpm audit:*)",
    "Bash(cat package.json:*)",
    "Bash(cat package-lock.json:*)",
    "Read",
    "Glob",
    "Grep",
    "Agent",
  ]
---

## Audit Dependencies

Scan project dependencies for security vulnerabilities, outdated packages, and license issues. Produces a prioritized report with actionable remediation steps.

### Input

`$ARGUMENTS` — optional focus area. If empty, perform a full audit. Examples:

- `security` — focus on CVEs and known vulnerabilities only
- `outdated` — focus on outdated packages and available updates
- `production` — audit only production dependencies (skip devDependencies)

### 1. Detect Package Manager

- Check for `pnpm-lock.yaml` or `package-lock.json` (in that order)
- Use the corresponding package manager for all subsequent commands
- If none found, stop and report that no lockfile was detected

### 2. Run Security Audit

- Run the package manager's built-in audit command (e.g., `npm audit`, `pnpm audit`, `yarn audit`)
- Capture the full output including severity levels
- If the audit command returns vulnerabilities, categorize them by severity: critical, high, moderate, low

### 3. Check for Outdated Packages

- Skip this step if `$ARGUMENTS` is `security`
- Run the outdated check (e.g., `npm outdated`, `pnpm outdated`)
- Identify packages with major version bumps (potential breaking changes)

### 4. Analyze and Prioritize

For each vulnerability found:
- Identify the dependency chain (direct vs transitive)
- Check if a fix is available (patched version exists)
- Assess impact: is the vulnerable code path reachable in this project?
- Flag any vulnerabilities in direct dependencies as highest priority

### 5. Present Report

Show the user a summary table:

| # | Package | Severity | Type | Vulnerability | Fix Available | Action |
|---|---------|----------|------|---------------|---------------|--------|

Where:
- **Type**: `direct` or `transitive`
- **Action**: `upgrade`, `replace`, `accept risk`, or `needs investigation`

Group by severity (critical first), then by direct vs transitive.

### 6. Offer Remediation

**STOP HERE.** Do not make changes until the user approves. Present options:

- **Auto-fix safe updates**: run audit fix for non-breaking patches
- **Upgrade specific packages**: list exact commands for each recommended upgrade
- **Accept risk**: document accepted vulnerabilities with justification

If the user approves fixes:
- Run the appropriate fix commands
- Re-run the audit to verify fixes were applied
- Report any remaining vulnerabilities

### Notes

- Never run `npm audit fix --force` without explicit user approval — it can introduce breaking changes
- For transitive vulnerabilities with no direct fix, suggest overrides/resolutions as a last resort
- If a vulnerability has no available fix, note the issue tracker status if available
