# incident-response

Incident investigation, lifecycle management, escalation tracking, and change logging.

## Prerequisites

Requires the `incident-response` MCP server to be configured. This plugin provides the skills that interact with the MCP tools — the server itself is not bundled.

## Skills

| Skill | Description |
|-------|-------------|
| `investigate` | Route to appropriate MCP tool, synthesize structured incident reports |
| `manage-incident` | Handle incident lifecycle: acknowledge, snooze, investigate, resolve |
| `check-escalations` | Query open incidents past acknowledgement deadline |
| `record-change` | Log infrastructure changes for future incident correlation |

## Configuration

No configuration needed. All skills work with zero setup (beyond the MCP server).

## Installation

```bash
claude plugin add incident-response@jbalatero
```
