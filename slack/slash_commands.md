# Slack Slash Commands

## Command Reference

| Command | Description | Example |
|---------|-------------|---------|
| `/slack-notify` | Send a notification to Slack | `/slack-notify Deploy notification to #deploys` |
| `/slack-incident` | Manage incidents | `/slack-incident Declare SEV-1 for checkout errors` |
| `/slack-summarize` | Summarize a thread or channel | `/slack-summarize Summarize #architecture from today` |
| `/slack-standup` | Run async standup | `/slack-standup Post standup prompt to #team` |
| `/slack-search` | Search Slack messages | `/slack-search Find discussion about Redis migration` |
| `/slack-report` | Post a status report | `/slack-report Post daily engineering report` |

---

## `/slack-notify`

```yaml
---
name: slack-notify
description: Send structured notifications to Slack channels
user-invocable: true
allowed-tools:
  - Bash
  - mcp__slack__*
---
```

### Usage

```
# Deploy notification
/slack-notify Send a deployment notification to #deployments for api-service v2.3.1

# Custom message
/slack-notify Post to #engineering: Sprint 42 review meeting moved to 3 PM

# Alert summary
/slack-notify Summarize current alerts and post to #incidents
```

---

## `/slack-incident`

```yaml
---
name: slack-incident
description: Declare and manage incidents through Slack
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Write
  - mcp__slack__*
  - mcp__datadog__*
  - mcp__atlassian__*
---
```

### Usage

```
# Declare incident
/slack-incident Declare SEV-1: Checkout service returning 500 errors

# Update incident
/slack-incident Update: We identified the root cause as a null pointer in OrderService

# Resolve
/slack-incident Resolve the checkout incident - rollback to v2.3.0 successful

# Generate post-mortem
/slack-incident Generate post-mortem for the checkout incident
```

### What Happens on Declaration

1. Creates `#inc-2026-03-22-checkout-errors` channel
2. Posts pinned context message with alert details
3. Pages on-call responders
4. Creates Jira incident ticket
5. Posts to #incidents with link to incident channel
6. Starts timeline tracking

---

## `/slack-summarize`

### Usage

```
# Summarize a thread
/slack-summarize Summarize the thread in #architecture about database migration

# Channel digest
/slack-summarize Give me a digest of #engineering from this week

# Meeting summary
/slack-summarize Summarize the standup thread from today
```

### Output Example

```
## Summary: Database Migration Discussion (#architecture)

### Context
Thread started by @alice about migrating from MySQL to PostgreSQL.

### Key Points
1. PostgreSQL offers better JSON support for our document storage needs
2. Migration can be done with minimal downtime using logical replication
3. Estimated effort: 2-3 sprints including data migration and testing

### Decision
Team agreed to proceed with PostgreSQL migration starting Sprint 43.

### Action Items
- [ ] @alice: Create migration plan document by March 25
- [ ] @bob: Set up PostgreSQL staging environment
- [ ] @carol: Audit all MySQL-specific queries in the codebase

### Open Questions
- How to handle the reporting database that depends on MySQL replication?
```

---

## `/slack-standup`

### Usage

```
# Start standup
/slack-standup Post standup prompt to #engineering-standup

# Generate summary
/slack-standup Summarize today's standup responses

# Weekly report
/slack-standup Generate weekly standup summary for #engineering-standup
```

---

## `/slack-search`

```yaml
---
name: slack-search
description: Search Slack messages and conversations
user-invocable: true
allowed-tools:
  - Bash
  - mcp__slack__*
---
```

### Usage

```
# Search for a topic
/slack-search Find discussions about the authentication refactor

# Search by person
/slack-search What did @alice say about the API redesign?

# Search in a channel
/slack-search Find deployment-related messages in #incidents this week
```

---

## `/slack-report`

### Usage

```
# Daily report
/slack-report Post daily engineering report to #engineering

# Sprint report
/slack-report Post sprint progress update to #product

# Incident report
/slack-report Post weekly incident summary to #sre
```

### Report Template

```
## Engineering Daily Report - March 22, 2026

### Highlights
- Shipped checkout redesign (v2.3.1)
- Resolved 3-day-old cache invalidation bug

### Metrics
| Metric | Today | Yesterday | Trend |
|--------|-------|-----------|-------|
| Deploys | 3 | 2 | :arrow_up: |
| PRs Merged | 7 | 5 | :arrow_up: |
| Open Incidents | 0 | 1 | :arrow_down: |
| Error Rate | 0.2% | 0.3% | :arrow_down: |

### Blockers
- Staging environment Redis cluster needs upgrade (affects 2 teams)

### Tomorrow
- Payment provider API integration testing
- Sprint 42 retrospective
```
