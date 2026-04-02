# Worklog Skills

Skills for tracking daily work activity and generating status reports.

## Skills

### `/update-worklog`

Queries GitHub for all PR activity (authored, reviewed, commented) on a given date and generates worklog.yaml task entries with contextual descriptions, auto-detected JIRA tickets, and inferred status. Supports incremental updates via `last_updated` timestamps.

### `/status-update`

Generates an HTML work report and posts status comments to JIRA tickets for the current biweekly reporting period. Automatically calculates the start date based on a Tuesday/Thursday cycle.

## Prerequisites

- GitHub CLI (`gh`) must be installed and authenticated
- A worklog.yaml file (default: `~/worklog/worklog.yaml`)
- [TaskLedger](https://github.com/bryan-cox/taskledger) binary for HTML report generation
- Atlassian JIRA MCP server for posting status comments
