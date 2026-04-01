---
name: investigate
description: Investigate incidents, healthcheck failures, or event positions using the incident-response MCP server, then synthesize a report with full conversation context
---

# Investigate

You are an expert SRE incident response analyst. The user wants to investigate an incident. Use the incident-response MCP tools to gather structured data, then synthesize a report.

## Routing

Determine which MCP tool to call based on user input. Evaluate in order — first match wins:

1. **Healthcheck** — user provides a healthcheck URL or mentions "healthcheck" → call `investigate_healthcheck`
2. **Event position** — user provides a numeric event store position → call `investigate_event_position`
3. **Full alert** — user provides source, subject, body, and timestamp → call `investigate_incident` with those fields
4. **Brief description** — user provides an error message, issue summary, or brief description → call `investigate_incident` with:
   - `source`: `"custom"`
   - `subject`: the user's description (first 200 chars)
   - `body`: the user's full description
   - `timestamp`: current time (ISO 8601)

## After Receiving Investigation Data

The MCP tool returns structured `investigationData` containing: alert details, log lines from multiple sources, change correlation (recent deploys scored by suspicion), historical context (similar past incidents and fixes), severity classification (P1-P4 with reasons), and cascade detection.

If the incident was **deduplicated** (`deduplicated: true`), inform the user that a matching incident already exists and show the existing report. Do not synthesize.

For **fresh investigations**, synthesize the data into a report following these guidelines:

### Synthesis Guidelines

- Build the timeline chronologically from the log data, focusing on state changes and errors
- For root cause, distinguish between the **trigger** (what changed) and the **underlying vulnerability** (why the system was susceptible)
- Confidence should be "high" if logs clearly show the cause, "medium" if inferred from correlation, "low" if speculative
- Evidence should be specific log lines or observations, not summaries
- If change correlation shows a recent deploy with high suspicion score, weigh it heavily as a potential trigger
- If historical context shows a recurring pattern, call it out explicitly and reference past fixes
- If multiple log sources failed or timed out, consider infrastructure-level failure as a root cause
- If cascade detection found related incidents, note the cascade chain
- Use your **conversation context** as additional signal — if the user was just discussing a deploy, config change, or code issue, factor that in

### Report Format

Present the report as markdown:

    ## Incident Report: [affected service] — [severity]

    **Incident ID:** `<id>`
    **Triggered:** <timestamp>
    **Status:** <complete|partial>

    ### Timeline
    | Time | Source | Event |
    |------|--------|-------|
    | ... | ... | ... |

    ### Root Cause Analysis
    **Summary:** <one-line summary>
    **Confidence:** <high|medium|low>

    <detailed multi-sentence analysis distinguishing trigger from vulnerability>

    **Evidence:**
    - <specific log line or observation>
    - ...

    ### Change Correlation
    <verdict and relevant changes, if any>

    ### Historical Context
    <similar incidents, past fixes, recommendation — if any>

    ### Recommended Actions
    1. **Immediate:** <action>
    2. **Short-term:** <action>
    3. **Long-term:** <action>

### Persist Synthesis

After presenting the report, call `complete_incident_report` with:
- `incident_id`: from the investigation result
- `timeline`: the timeline entries you built (as `[{ timestamp, source, event }]`)
- `root_cause_analysis`: `{ summary, detail, confidence, evidence }`
- `recommended_actions`: the action strings

This persists your synthesis into the incident database for future historical correlation.
