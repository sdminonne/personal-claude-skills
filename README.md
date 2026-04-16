# personal-claude-skills

A place for me to share personal Claude Code skills I use on a daily basis that might help others as well but I don't want to merge them into a formal repo just yet.

## Installation

1. Open Claude Code and run `/plugin`
2. Select **Add marketplace**
3. Enter the repository URL: `https://github.com/bryan-cox/personal-claude-skills`
4. Select **Install plugin**
5. Choose the plugin you want to install

## Available Plugins

### behavior-driven-testing

Guides writing of behavior-driven Go unit tests that prioritize testing meaningful behaviors over chasing code coverage.

**What it does:**
- Enforces Gherkin-style test naming: `"When <precondition>, it should <expected behavior>"`
- Promotes gomega assertions with expressive matchers
- Guides table-driven test structure (idiomatic Go)
- Covers fake clients, mocking patterns, and envtest guidance for Kubernetes API testing
- Encourages testing behaviors, not implementation details

### worklog

Populate worklog from GitHub PR activity and generate biweekly status reports with JIRA updates.

**Skills included:**
- `/update-worklog` — Queries GitHub for all PR activity (authored, reviewed, commented) on a given date and generates worklog.yaml task entries with contextual descriptions, auto-detected JIRA tickets, and inferred status. Supports incremental updates via `last_updated` timestamps.
- `/status-update` — Generates an HTML work report and posts status comments to JIRA tickets for the current biweekly reporting period (Tuesday/Thursday cycle).

**Prerequisites:**
- GitHub CLI (`gh`) installed and authenticated
- `/status-update` requires the [taskledger](https://github.com/bryan-cox/taskledger) plugin (invokes `/taskledger:html-report` and `/taskledger:update-jira`)
- Atlassian JIRA MCP server for posting status comments

### quarterly-connection

Generate Red Hat quarterly connection self-evaluations by analyzing worklog data, Jira tickets, GitHub PRs, and code reviews.

**Skills included:**
- `/quarterly-connection` — Interactively gathers your quarterly goals, self-evaluation questions, reward zone awards, and work history, then uses parallel agents to analyze your worklog.yaml, enrich Jira tickets, and summarize GitHub activity. Produces a well-organized markdown self-evaluation with work mapped to themes, high-priority items highlighted, and unanswerable questions flagged for your input.

**Prerequisites:**
- Worklog.yaml file (produced by the [worklog](#worklog) plugin's `/update-worklog` skill)
- GitHub CLI (`gh`) installed and authenticated
- Atlassian JIRA MCP server (for enriching Jira ticket details)
