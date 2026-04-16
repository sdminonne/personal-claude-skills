---
name: verify-qc
description: Use when verifying, fact-checking, or auditing a quarterly connection self-evaluation for accuracy. Trigger when the user says "verify my QC", "check my quarterly connection", "fact-check my self-evaluation", "audit my QC", or wants to validate claims, PR counts, or Jira ticket ownership in a quarterly connection document. Also use after running /quarterly-connection to validate the output before submitting.
---

# Verify Quarterly Connection

Fact-check a quarterly connection self-evaluation by verifying every claim against real data sources. This matters because inaccurate claims in a QC erode trust with your manager — even honest mistakes can make your entire report look unreliable. Better to catch issues before your manager does.

## Input

The skill takes one input: the path to a markdown file produced by `/quarterly-connection` (or any quarterly connection document in markdown format).

Ask: "What is the path to your quarterly connection markdown file?"

Also determine the employee's GitHub username and Jira username for ownership checks. Run `gh api user --jq '.login'` to get the GitHub username. For Jira, use the `jira_get_user_profile` MCP tool or ask the employee.

## Verification Strategy

Parse the markdown document and extract every verifiable claim. Then launch parallel subagents (model: opus) to check different categories simultaneously. Each subagent should report back with a list of findings in a structured format.

### What to Extract from the Document

Scan the markdown for:
1. **Jira ticket references** — any pattern matching `[A-Z][A-Z0-9]+-\d+` (e.g., CNTRLPLANE-1331, OCPBUGS-60517)
2. **GitHub PR references** — full URLs (`https://github.com/...`) or short-form (`#1234` with a repo context)
3. **Numerical claims** — "Merged 33 PRs", "Completed 47 Jira tickets", "60+ merged PRs", "3 upstream PRs"
4. **GitHub contribution links** — search URLs in the GitHub Contributions section
5. **Date claims** — "December 5, 2025", "October 27-31", "August 11-21"
6. **Role/status claims** — "officially approved as reviewer", "served as interrupt catcher"

## Parallel Verification Subagents

Launch these subagents in parallel (all opus model):

### Subagent 1: Jira Ticket Ownership

```
For each Jira ticket ID extracted from the document:

Use the jira_get_issue MCP tool to fetch:
- Assignee
- Summary/title
- Status
- Priority

Check:
1. Is the employee the assignee (current or was at any point)?
2. Does the ticket summary roughly match how it's described in the QC?

Report for each ticket:
- TICKET-ID: assignee match (yes/no/unassigned), summary match (yes/no), actual assignee if mismatch
```

### Subagent 2: GitHub PR Verification

```
For each GitHub PR URL or reference in the document:

Use gh CLI to fetch:
- PR state (merged, open, closed)
- PR author
- Merge date (if merged)
- Repository

Check:
1. Was the PR actually authored by the employee?
2. Is the stated status correct (e.g., "merged" — was it really merged, or just closed)?
3. If a merge date is claimed, does it fall within the quarter?

Report for each PR:
- PR URL: author match (yes/no), status correct (yes/no), in-quarter (yes/no), actual details if mismatch
```

### Subagent 3: Numerical Claims

```
For each numerical claim in the document (e.g., "Merged 33 PRs in openshift/hypershift"):

Verify by running the actual query. For merged PR counts per repo, use:
gh search prs --author={username} --merged="{start_date}..{end_date}" --repo={org}/{repo} --limit 1000 --json url | python3 -c "import sys,json; print(len(json.load(sys.stdin)))"

For total Jira ticket counts, count the unique tickets verified in Subagent 1.

For "X+ merged PRs across repositories", sum the per-repo counts.

Check:
1. Does the claimed number match the actual count?
2. If approximate (e.g., "60+"), is the actual number at least that high?

Report for each claim:
- Claim: "{exact text from document}"
- Actual: {verified number}
- Verdict: correct / overcounted / undercounted / cannot verify
```

### Subagent 4: GitHub Contribution Links

```
For each GitHub search URL in the GitHub Contributions section:

Open the URL via gh CLI or construct the equivalent API query to count results.

Check:
1. Does the claimed count next to the link match the actual search result count?
2. Does the link use the correct date range for the quarter?
3. Does the link filter to the correct author?
4. Does the link use is:merged (not just closed)?

Report for each link:
- Link text: "{Repo} Merged PRs: {claimed_count}"
- Actual count from search: {actual_count}
- Date range correct: yes/no
- Author correct: yes/no
- Filters merged only: yes/no
```

## Compile Verification Report

After all subagents complete, compile the results into a clear report for the employee.

### Report Format

```markdown
# QC Verification Report

**Document:** {file path}
**Verified:** {date}
**Employee:** {username}

## Summary

- **Claims checked:** {total count}
- **Verified correct:** {count}
- **Issues found:** {count}
- **Could not verify:** {count}

## Issues Found

List every problem, grouped by severity:

### Errors (incorrect claims that should be fixed)

| Claim | Issue | Actual Value |
|-------|-------|--------------|
| "Merged 33 PRs in hypershift" | Count is wrong | Actual: 29 merged PRs |
| OCPBUGS-60517 | Assigned to jdoe, not you | Assignee: jdoe |

### Warnings (minor discrepancies or things to double-check)

| Claim | Issue | Details |
|-------|-------|---------|
| CNTRLPLANE-1331 | Ticket summary doesn't match description | Ticket: "Azure SM bringup", QC says: "self-managed Azure HostedCluster" |
| GitHub link uses closed: instead of merged: | May include non-merged PRs | {link} |

### Unverifiable Claims

Claims that couldn't be checked against available data sources:

- "Served as interrupt catcher for October 27-31" — no data source to verify
- "Conducted 3+ technical interviews" — not tracked in Jira or GitHub

## Verified Correct

All claims that passed verification:

- CNTRLPLANE-1857: assignee confirmed, summary matches
- "Merged 11 PRs in openshift/release": actual count is 11
- {etc.}
```

### Severity Guidelines

**Error** — the claim is factually wrong and would be embarrassing if a manager checked:
- PR count is wrong (overcounted or undercounted)
- Jira ticket is assigned to someone else
- PR was not actually merged
- PR was not authored by the employee

**Warning** — the claim is technically correct but could be misleading or imprecise:
- Ticket summary doesn't closely match the QC description (but it's the right ticket)
- GitHub search link uses `closed:` instead of `merged:` (inflates count)
- Date range in search link doesn't exactly match the quarter
- Approximate claim ("60+ PRs") is technically correct but actual number is much higher (underselling)

**Unverifiable** — no available data source to check:
- Mentoring claims, meeting attendance, interrupt catcher duties
- Interview counts
- Anything that isn't tracked in Jira or GitHub

## Present Results

1. Show the full verification report to the employee
2. Highlight the most critical issues first
3. For each error, suggest the correction (e.g., "Change '33 PRs' to '29 PRs'")
4. Ask: "Would you like me to fix the errors in your QC document directly?"
5. If yes, apply the corrections to the markdown file
