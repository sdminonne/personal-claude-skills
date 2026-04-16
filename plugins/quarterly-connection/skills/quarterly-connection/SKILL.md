---
name: quarterly-connection
description: Use when writing a Red Hat quarterly connection self-evaluation, preparing a performance review, or when the user mentions "quarterly connection", "quarterly review", "self-evaluation", "quarterly self-assessment", or "performance review". Also trigger when the user wants to summarize a quarter's worth of work into a performance document, or asks to analyze their worklog for review purposes.
---

# Red Hat Quarterly Connection

Generate a comprehensive quarterly connection self-evaluation for a Red Hat software engineer. This document directly impacts the employee's performance rating and bonus, so thoroughness and accuracy are essential. The skill analyzes worklog data, Jira tickets, GitHub PRs, and code reviews to produce a well-organized self-evaluation in markdown.

## Workflow Overview

```
Gather Inputs (interactive)
    |
    v
Parallel Investigation (subagents)
    |
    v
Synthesize & Organize by Goals
    |
    v
Draft Quarterly Connection (markdown)
    |
    v
Review with Employee & Refine
```

## Phase 1: Gather Inputs

Before any analysis, collect all required information from the employee. Ask for each of these **one at a time** to avoid overwhelming the user. Use the AskUserQuestion tool where appropriate, but for open-ended items just ask directly.

### 1.1 Worklog File

Ask: "What is the path to your worklog.yaml file?"

Default to `~/worklog/worklog.yaml` if the user confirms or doesn't specify.

### 1.2 Quarter Date Range

Ask: "What date range does this quarterly connection cover? (e.g., 2026-01-01 to 2026-03-31)"

Use this to filter worklog entries to only the relevant quarter.

### 1.3 Quarterly Goals

Ask: "What were your quarterly goals? Please list them — these will be the primary organizing structure for your self-evaluation."

Store these as the top-level sections for the output. Every piece of work will be mapped to a goal where possible. Work that doesn't fit any goal gets its own "Additional Contributions" section.

### 1.4 Quarterly Connection Questions

Ask: "What questions does the quarterly connection self-evaluation ask you to answer? Please paste them or list them."

These are the specific prompts the employee needs to respond to. The skill will attempt to answer each one using the data gathered. Questions that cannot be answered from the available data will be flagged with a `<!-- NEEDS INPUT -->` marker so the employee can fill them in.

### 1.5 Previous Quarterly Connections

Ask: "Do you have any previous quarterly connections I can reference for tone, format, and style? If so, please provide the file path(s)."

If provided, read them and use them as reference for:
- Writing style and tone
- Level of detail expected
- How goals and accomplishments are typically framed
- Any recurring themes or long-running projects

### 1.6 Additional Context

Ask: "Is there anything else I should know before creating this report? This could include:
- Work not tracked in Jira or GitHub (mentoring, meetings, design discussions, oncall, etc.)
- Key accomplishments you want highlighted
- Challenges or blockers you faced
- Team or organizational impact you want called out
- Anything else relevant to your performance this quarter"

This is the final catch-all. Everything the employee says here gets incorporated into the report.

## Phase 2: Parallel Investigation

After gathering all inputs, launch parallel subagents (model: opus) to investigate different data sources simultaneously. This phase is about extracting and enriching the raw data before organizing it.

### 2.1 Subagent: Worklog Analysis

```
Analyze the worklog.yaml file at {worklog_path}.

Filter to entries between {start_date} and {end_date} inclusive.

For each task entry, extract:
- Date
- Jira ticket (if any)
- Descriptions (what was done)
- Status (completed, in progress)
- GitHub PR URL (if any)
- Blockers

Produce a structured summary:
1. Total working days logged
2. List of all unique Jira tickets worked on, with aggregated descriptions across all days
3. List of all GitHub PRs authored, with their final status
4. List of all code reviews performed (entries with jira_ticket "Reviewing Pull Requests")
5. Count of PRs authored, PRs merged, PRs reviewed
6. Any blockers mentioned

Save the structured summary to a file.
```

### 2.2 Subagent: Jira Ticket Enrichment

For every unique Jira ticket found in the worklog, fetch details from Jira. Use the Jira MCP tools if available, otherwise note that manual enrichment is needed.

```
For each Jira ticket in {ticket_list}:

Use the jira_get_issue MCP tool to fetch:
- Summary/title
- Priority (Blocker, Critical, Major, Minor, Trivial)
- Status
- Component
- Epic link (if any)
- Description summary (first 200 chars)

Flag tickets with priority Blocker, Critical, or Major for special highlighting in the report.

Group tickets by priority level.

Save results to a file.
```

### 2.3 Subagent: GitHub PR Deep Dive

```
For each GitHub PR URL found in the worklog for the quarter:

Use gh CLI to fetch:
- PR title
- PR state (merged, open, closed)
- Lines added/removed
- Number of review comments received
- Repository name
- Merge date (if merged)

Calculate aggregate stats:
- Total PRs authored
- Total PRs merged
- Total lines changed (added + removed)
- Repos contributed to

Save results to a file.
```

### 2.4 Subagent: Code Review Analysis

```
Analyze all "Reviewing Pull Requests" entries from the worklog between {start_date} and {end_date}.

Extract every PR URL reviewed or commented on.

Calculate:
- Total PRs reviewed
- Unique repositories reviewed
- Review frequency (reviews per week)

Save results to a file.
```

### 2.5 Subagent: Previous Quarterly Connection Analysis (if provided)

Only launch this if the employee provided previous quarterly connections.

```
Read the previous quarterly connection(s) at {file_paths}.

Analyze:
- Writing style and tone (formal, casual, bullet-heavy, narrative)
- How goals are structured and referenced
- Level of detail in accomplishments
- How metrics and impact are framed
- Any recurring themes or long-running projects that may continue into this quarter
- Format and section structure

Produce a style guide summary that can be used to match tone and format.
```

## Phase 3: Synthesize and Organize

Once all subagents complete, synthesize the results.

### 3.1 Map Work to Goals

For each quarterly goal the employee listed:
1. Identify all Jira tickets, PRs, and worklog entries that relate to that goal
2. Group them under the goal
3. Summarize the body of work in a narrative that demonstrates progress toward or completion of the goal

Use Jira ticket summaries, PR titles, and worklog descriptions to build the narrative. Mention specific ticket IDs and PR URLs as evidence.

### 3.2 Identify High-Impact Work

From the Jira enrichment data, identify and flag:
- **Blocker priority** tickets: These represent critical work that unblocked the team or organization
- **Critical priority** tickets: High-urgency work with significant impact
- **Major priority** tickets: Important work with broad reach

For each high-priority item, write a brief impact statement explaining why it mattered and what the outcome was.

### 3.3 Categorize Remaining Work

Work that doesn't map to any quarterly goal goes into an "Additional Contributions" section, organized by category:
- Code reviews and mentoring
- Bug fixes
- CI/infrastructure improvements
- Documentation
- Cross-team collaboration
- Other

### 3.4 Answer Quarterly Connection Questions

For each question from section 1.4:
1. Attempt to answer it using the synthesized data
2. If the question can be answered from the data (e.g., "What did you accomplish this quarter?"), draft a response
3. If the question requires subjective input the data can't provide (e.g., "How do you want to grow?"), mark it with `<!-- NEEDS INPUT: This question requires your personal reflection. -->` and provide any relevant data points that might help the employee answer it

## Phase 4: Generate the Quarterly Connection Document

Produce the final document in markdown format. Follow the style and tone of previous quarterly connections if provided.

### Document Structure

```markdown
# Quarterly Connection: {Quarter} {Year}

**Employee:** {name from gh api user or ask}
**Period:** {start_date} to {end_date}
**Date:** {today}

---

## Quarter at a Glance

- **Jira tickets worked:** {count}
- **PRs authored:** {count} ({merged_count} merged)
- **Code reviews performed:** {count}
- **Lines of code changed:** {count}
- **Repositories contributed to:** {list}

---

## Goal: {Goal 1 Title}

{Narrative summary of work toward this goal}

### Key Accomplishments

- {Accomplishment with Jira ticket and/or PR reference}
- {Accomplishment}

### High-Impact Items

> **[BLOCKER] {TICKET-ID}: {Summary}**
> {Impact statement}

> **[CRITICAL] {TICKET-ID}: {Summary}**
> {Impact statement}

---

## Goal: {Goal 2 Title}

{Same structure as above}

---

## Additional Contributions

### Code Reviews
{Summary of review activity with count and notable reviews}

### {Other categories as needed}

---

## Quarterly Connection Questions

### {Question 1}

{Answer or <!-- NEEDS INPUT --> marker}

### {Question 2}

{Answer or <!-- NEEDS INPUT --> marker}

---

## Questions Requiring Your Input

The following questions could not be fully answered from your work data and need your personal input:

- {Question X} — see section above
- {Question Y} — see section above
```

### Formatting Guidelines

- Use bold for Jira ticket IDs and priorities
- Use blockquotes for high-impact item callouts
- Link GitHub PRs using markdown links: `[PR #123](url)`
- Use concrete numbers and metrics wherever possible — reviewers respond to quantified impact
- Write accomplishments in past tense, active voice: "Implemented X that resulted in Y" not "X was implemented"
- Frame work in terms of impact, not just activity: "Fixed critical DNS pagination bug (OCPBUGS-74495) that was blocking cluster upgrades for Azure customers" not "Fixed a bug"

### Tone

- Professional but not stiff
- Confident without being boastful — let the work speak for itself
- Specific and evidence-based
- Match the tone of previous quarterly connections if provided

## Phase 5: Review and Refine

After generating the document:

1. Present the full markdown document to the employee
2. Call out the sections marked `<!-- NEEDS INPUT -->` and ask the employee to provide their input for those
3. Ask: "Would you like me to adjust anything? I can expand on specific accomplishments, reframe how something is presented, add context I might have missed, or adjust the tone."
4. Iterate on feedback until the employee is satisfied
5. Save the final document to a file path the employee specifies (default: `~/quarterly-connection-{quarter}-{year}.md`)
