# quarterly-connection

Generate comprehensive Red Hat quarterly connection self-evaluations for software engineers by analyzing worklog data, Jira tickets, GitHub PRs, and code reviews.

**Skills included:**
- `/quarterly-connection` — Interactively gathers your quarterly goals, self-evaluation questions, and work history, then uses parallel agents to analyze your worklog.yaml, enrich Jira tickets, and summarize GitHub activity. Produces a well-organized markdown self-evaluation with work mapped to goals, high-priority items highlighted, and unanswerable questions flagged for your input.

**Prerequisites:**
- Worklog.yaml file (produced by the [worklog](/plugins/worklog) plugin's `/update-worklog` skill)
- GitHub CLI (`gh`) installed and authenticated
- Atlassian JIRA MCP server (for enriching Jira ticket details with priority, status, etc.)
