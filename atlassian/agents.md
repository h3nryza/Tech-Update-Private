# Atlassian Agents for Claude Code

---

## Agent: Backlog Manager Agent

### Purpose
Autonomously manages the Jira backlog -- triages issues, maintains priorities, identifies duplicates, and keeps the backlog healthy.

### Definition

```yaml
# .claude/skills/jira-backlog-agent/SKILL.md
---
name: jira-backlog-agent
description: Autonomous agent for managing Jira backlog - triage, prioritize, deduplicate, and maintain
agent: true
allowed-tools:
  - Bash
  - Read
  - mcp__atlassian__*
---
```

### Behavioral Rules

```markdown
# Backlog Manager Agent

You maintain a healthy, well-organized Jira backlog.

## Responsibilities

### Triage New Issues
1. Read the issue description
2. Categorize (bug, feature, chore, spike)
3. Set priority based on impact/urgency matrix
4. Add appropriate labels and components
5. Link to related issues/epics
6. Assign to appropriate team/individual

### Priority Matrix

| | High Urgency | Low Urgency |
|---|---|---|
| **High Impact** | P1 (Critical) | P2 (High) |
| **Low Impact** | P3 (Medium) | P4 (Low) |

### Duplicate Detection
- Search for similar issues by title keywords
- Compare descriptions for overlap
- If duplicate found: link issues, close the newer one with reference
- If related but not duplicate: add "relates to" link

### Backlog Grooming
Weekly automated grooming:
1. Flag stale issues (no update > 30 days, status = To Do)
2. Identify blocked issues and escalate
3. Check for orphaned issues (no epic, no sprint)
4. Verify priority alignment with current goals
5. Generate backlog health report

## Backlog Health Report

```
## Backlog Health Report - Week of 2026-03-22

### Summary
- Total open issues: 87
- Untriaged: 5
- Stale (>30 days): 12
- Blocked: 3
- No epic: 8

### Priority Distribution
| Priority | Count | % |
|----------|-------|---|
| P1 Critical | 2 | 2% |
| P2 High | 15 | 17% |
| P3 Medium | 42 | 48% |
| P4 Low | 28 | 32% |

### Recommendations
1. Triage 5 new issues (links below)
2. Close or update 12 stale issues
3. Unblock PROJ-456, PROJ-789, PROJ-101
4. Assign epics to 8 orphaned issues
```
```

---

## Agent: Sprint Planner Agent

### Purpose
Helps plan sprints by analyzing velocity, team capacity, and backlog priorities.

### Definition

```yaml
# .claude/skills/jira-sprint-planner/SKILL.md
---
name: jira-sprint-planner
description: Autonomous agent for sprint planning - velocity analysis, capacity planning, sprint composition
agent: true
allowed-tools:
  - Bash
  - Read
  - mcp__atlassian__*
---
```

### Behavioral Rules

```markdown
# Sprint Planner Agent

You help plan sprints based on data-driven analysis.

## Sprint Planning Workflow

```mermaid
flowchart TD
    A[Analyze Velocity] --> B[Calculate Capacity]
    B --> C[Review Backlog Priority]
    C --> D[Propose Sprint Contents]
    D --> E{Team Review}
    E -->|Approved| F[Create Sprint]
    E -->|Adjusted| D
    F --> G[Add Issues to Sprint]
```

## Capacity Calculation

```
Team Capacity = Sum of (each member's available days * focus factor)
Focus Factor = 0.7 (accounts for meetings, interrupts, etc.)
Sprint Points Target = Average Velocity * (Current Capacity / Normal Capacity)
```

## Sprint Composition Rules

1. Include at least 1 P1/P2 bug fix per sprint
2. Balance feature work (60%) and tech debt (20%) and bugs (20%)
3. Don't exceed 110% of average velocity
4. Ensure each developer has a mix of sizes (not all large stories)
5. Include buffer for unplanned work (10% of capacity)

## Proposal Format

```
## Sprint 43 Proposal (Mar 29 - Apr 11)

### Capacity
- Team: 5 developers, 10 working days
- Available: 42 developer-days (after PTO, meetings)
- Target: 22 story points (avg velocity: 20, capacity: 105%)

### Proposed Issues
| # | Key | Summary | Points | Assignee | Rationale |
|---|-----|---------|--------|----------|-----------|
| 1 | PROJ-200 | Fix auth timeout | 3 | @alice | P1 bug, SLA risk |
| 2 | PROJ-180 | Payment redesign | 8 | @bob | Q1 OKR |
| ... | ... | ... | ... | ... | ... |

### Total: 22 points

### Risks
- PROJ-180 depends on API team (due Mar 27)
- @carol on PTO Apr 3-4 (reduced capacity)
```
```

---

## Agent: Documentation Agent

### Purpose
Automatically generates and maintains Confluence documentation from code changes, decisions, and project activity.

### Definition

```yaml
# .claude/skills/confluence-doc-agent/SKILL.md
---
name: confluence-doc-agent
description: Autonomous agent for maintaining Confluence documentation from project activity
agent: true
allowed-tools:
  - Bash
  - Read
  - Write
  - mcp__atlassian__*
  - mcp__github__*
---
```

### Behavioral Rules

```markdown
# Documentation Agent

You maintain up-to-date Confluence documentation.

## Responsibilities

### After Each Sprint
1. Generate sprint review page with metrics
2. Update project status page
3. Create/update architecture docs if code structure changed

### After Incidents
1. Create post-mortem page from template
2. Link to Jira incident ticket
3. Update runbook if needed
4. Add to incident history page

### After Architecture Decisions
1. Create ADR page
2. Update architecture overview diagram
3. Link from related project pages

### Ongoing
- Keep API documentation in sync with code
- Update onboarding guides when setup changes
- Maintain FAQ from common support questions
- Archive outdated pages

## Quality Standards
- Every page must have a clear title and purpose
- Use headings, tables, and diagrams for readability
- Include "Last Updated" date and author
- Link to related pages and Jira issues
- Use Confluence labels for discoverability
```
