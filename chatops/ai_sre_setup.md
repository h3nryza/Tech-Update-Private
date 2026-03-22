# Setting Up an AI SRE Across All Platforms

## Overview

This guide walks through setting up a complete AI SRE that integrates AWS, GitHub, Datadog, Atlassian, Slack, and Cloudflare into a unified operations platform powered by Claude Code.

## Architecture

```mermaid
graph TB
    subgraph "Claude Code AI SRE"
        BRAIN[AI SRE Brain<br/>Claude Code + Agent SDK]
        SKILLS[Skills Library]
        AGENTS[Agent Fleet]
    end

    subgraph "MCP Servers"
        MCP_AWS[AWS MCP]
        MCP_DD[Datadog MCP]
        MCP_GH[GitHub MCP]
        MCP_JIRA[Jira MCP]
        MCP_SLACK[Slack MCP]
        MCP_CF[Cloudflare MCP]
    end

    subgraph "Platforms"
        AWS[AWS]
        DD[Datadog]
        GH[GitHub]
        JIRA[Jira]
        SLACK[Slack]
        CF[Cloudflare]
    end

    BRAIN --> SKILLS
    BRAIN --> AGENTS
    SKILLS --> MCP_AWS
    SKILLS --> MCP_DD
    SKILLS --> MCP_GH
    SKILLS --> MCP_JIRA
    SKILLS --> MCP_SLACK
    SKILLS --> MCP_CF
    MCP_AWS --> AWS
    MCP_DD --> DD
    MCP_GH --> GH
    MCP_JIRA --> JIRA
    MCP_SLACK --> SLACK
    MCP_CF --> CF
```

## Prerequisites

- Claude Code installed and configured
- Accounts on all platforms with appropriate API access
- Node.js 18+, Python 3.10+

---

## Step 1: Install All MCP Servers

### Create the unified MCP configuration

```json
// .claude/mcp.json
{
  "mcpServers": {
    "aws-core": {
      "command": "npx",
      "args": ["-y", "@awslabs/core-mcp-server"],
      "env": {
        "AWS_PROFILE": "production",
        "AWS_REGION": "us-east-1",
        "MCP_SERVERS": "aws-api,aws-docs,aws-iac,cost-analysis"
      }
    },
    "datadog": {
      "command": "npx",
      "args": ["-y", "@datadog/mcp-server"],
      "env": {
        "DD_API_KEY": "${DD_API_KEY}",
        "DD_APP_KEY": "${DD_APP_KEY}",
        "DD_SITE": "datadoghq.com"
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/github-mcp-server"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "atlassian": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/atlassian-mcp-server"],
      "env": {
        "ATLASSIAN_SITE": "${ATLASSIAN_SITE}",
        "ATLASSIAN_EMAIL": "${ATLASSIAN_EMAIL}",
        "ATLASSIAN_API_TOKEN": "${ATLASSIAN_API_TOKEN}"
      }
    },
    "slack": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/slack-mcp-server"],
      "env": {
        "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}",
        "SLACK_APP_TOKEN": "${SLACK_APP_TOKEN}"
      }
    },
    "cloudflare": {
      "command": "npx",
      "args": ["-y", "@cloudflare/workers-mcp"],
      "env": {
        "CLOUDFLARE_API_TOKEN": "${CF_API_TOKEN}",
        "CLOUDFLARE_ACCOUNT_ID": "${CF_ACCOUNT_ID}"
      }
    }
  }
}
```

### Install via CLI

```bash
# AWS
claude mcp add aws-core --transport stdio -- npx -y @awslabs/core-mcp-server

# Datadog
claude mcp add datadog --transport stdio -- npx -y @datadog/mcp-server

# GitHub (or use gh CLI directly)
claude mcp add github --transport stdio -- npx -y @anthropic-ai/github-mcp-server

# Atlassian
claude mcp add atlassian --transport stdio -- npx -y @anthropic-ai/atlassian-mcp-server

# Slack
claude mcp add slack --transport stdio -- npx -y @anthropic-ai/slack-mcp-server

# Cloudflare
claude mcp add cloudflare --transport stdio -- npx -y @cloudflare/workers-mcp
```

---

## Step 2: Configure Authentication

### Environment Variables

Create a `.env` file (add to `.gitignore`):

```bash
# AWS (uses aws configure / SSO - no keys needed here)
AWS_PROFILE=production
AWS_REGION=us-east-1

# Datadog
DD_API_KEY=your-datadog-api-key
DD_APP_KEY=your-datadog-app-key

# GitHub
GITHUB_TOKEN=ghp_your-github-token

# Atlassian
ATLASSIAN_SITE=your-org.atlassian.net
ATLASSIAN_EMAIL=you@company.com
ATLASSIAN_API_TOKEN=your-atlassian-api-token

# Slack
SLACK_BOT_TOKEN=xoxb-your-slack-bot-token
SLACK_APP_TOKEN=xapp-your-slack-app-token

# Cloudflare
CF_API_TOKEN=your-cloudflare-api-token
CF_ACCOUNT_ID=your-cloudflare-account-id

# Anthropic
ANTHROPIC_API_KEY=sk-ant-your-api-key
```

### Verify All Connections

```bash
# Verify each platform
claude "List all configured MCP servers and test connectivity to each"
```

Expected output: All 6 MCP servers connected and responding.

---

## Step 3: Install Skills

### Create the skills directory structure

```bash
mkdir -p .claude/skills/{sre-incident,sre-monitor,sre-deploy,sre-investigate,sre-runbook,sre-postmortem}
```

### Core SRE Skill: Incident Response

```yaml
# .claude/skills/sre-incident/SKILL.md
---
name: sre-incident
description: Full incident response workflow across all platforms
allowed-tools:
  - Bash
  - Read
  - Write
  - mcp__slack__*
  - mcp__datadog__*
  - mcp__aws-core__*
  - mcp__atlassian__*
  - mcp__cloudflare__*
---
```

```markdown
# SRE Incident Response

When an incident occurs:

1. **Assess** severity using Datadog metrics and logs
2. **Declare** incident in Slack (create channel, page responders)
3. **Track** in Jira (create incident ticket)
4. **Investigate** using AWS/Datadog/Cloudflare data
5. **Mitigate** with human approval (rollback, scale, config change)
6. **Resolve** and update all channels
7. **Document** post-mortem in Confluence
8. **Follow up** on action items in Jira
```

### Core SRE Skill: Cross-Platform Monitor

```yaml
# .claude/skills/sre-monitor/SKILL.md
---
name: sre-monitor
description: Unified monitoring across AWS, Datadog, and Cloudflare
allowed-tools:
  - Bash
  - Read
  - mcp__datadog__*
  - mcp__aws-core__*
  - mcp__cloudflare__*
---
```

```markdown
# Cross-Platform Monitoring

Provide a unified health view across all platforms:

1. Query Datadog for application metrics (error rate, latency, throughput)
2. Query AWS for infrastructure metrics (CPU, memory, connections)
3. Query Cloudflare for edge metrics (cache hit rate, security events)
4. Correlate and present as unified health dashboard
```

---

## Step 4: Set Up GitHub Actions

### Claude Code for PR Review and Issue Implementation

```yaml
# .github/workflows/claude-sre.yml
name: Claude AI SRE
on:
  issue_comment:
    types: [created]
  pull_request:
    types: [opened, synchronize]
  issues:
    types: [labeled]

jobs:
  claude-sre:
    if: |
      (github.event_name == 'issue_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'pull_request') ||
      (github.event_name == 'issues' && contains(github.event.issue.labels.*.name, 'ai-sre'))
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
      issues: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          model: claude-opus-4-6
          timeout_minutes: 30
          mcp_config: |
            {
              "mcpServers": {
                "aws-api": {
                  "command": "npx",
                  "args": ["-y", "@awslabs/aws-api-mcp-server"],
                  "env": {
                    "AWS_ACCESS_KEY_ID": "${{ secrets.AWS_ACCESS_KEY_ID }}",
                    "AWS_SECRET_ACCESS_KEY": "${{ secrets.AWS_SECRET_ACCESS_KEY }}",
                    "AWS_REGION": "us-east-1"
                  }
                },
                "datadog": {
                  "command": "npx",
                  "args": ["-y", "@datadog/mcp-server"],
                  "env": {
                    "DD_API_KEY": "${{ secrets.DD_API_KEY }}",
                    "DD_APP_KEY": "${{ secrets.DD_APP_KEY }}"
                  }
                }
              }
            }
```

---

## Step 5: Set Up Slack Integration

### Slack App Configuration

1. Create a Slack App at https://api.slack.com/apps
2. Required scopes:
   - `channels:history`, `channels:read`, `channels:write`, `channels:manage`
   - `chat:write`, `chat:write.customize`
   - `groups:read`, `groups:write`, `groups:history`
   - `im:read`, `im:write`
   - `users:read`
   - `app_mentions:read`
   - `commands`
   - `pins:write`
   - `reactions:read`, `reactions:write`
3. Enable Socket Mode
4. Subscribe to events: `app_mention`, `message.channels`
5. Create slash commands: `/incident`, `/sre-status`, `/runbook`

### Alert Routing Configuration

```yaml
# alert_routing.yml
routing_rules:
  - name: critical_production
    conditions:
      severity: [critical, high]
      environment: production
    actions:
      - channel: "#incidents"
      - page: oncall-sre
      - page: oncall-backend
      - create_incident_channel: true

  - name: warning_production
    conditions:
      severity: [warning]
      environment: production
    actions:
      - channel: "#alerts"

  - name: staging_failures
    conditions:
      environment: staging
    actions:
      - channel: "#staging-alerts"

  - name: security_events
    conditions:
      type: security
    actions:
      - channel: "#security"
      - page: security-oncall
```

---

## Step 6: Configure Datadog Monitors

### Standard Monitor Set

Create these monitors for AI SRE integration:

```bash
# Using the Datadog MCP server
claude "Create the following Datadog monitors:
1. High error rate (>2%) for each production service
2. High p99 latency (>2s) for each production service
3. ECS task failures
4. RDS connection count approaching limit
5. Deployment event correlation (error spike within 30 min of deploy)
6. SLO burn rate alerts for all defined SLOs

Configure all monitors to notify #incidents on Slack and include
the tag 'managed-by:ai-sre' for identification."
```

---

## Step 7: Verify the Setup

### Verification Checklist

```bash
# 1. Verify MCP servers
claude mcp list

# 2. Test each integration
claude "Test connectivity:
- AWS: Describe EC2 instances in us-east-1
- Datadog: List monitors in ALERT state
- GitHub: List open PRs in this repo
- Jira: Search for recent issues in PROJ
- Slack: Send a test message to #test-channel
- Cloudflare: List DNS records for example.com"

# 3. Test incident flow (in a test channel)
claude "/sre-incident Declare a test incident SEV-4 in #test-incidents"

# 4. Verify alert routing
claude "Verify that Datadog monitors are configured to notify the correct Slack channels"
```

### Smoke Test: End-to-End Incident Flow

```
1. Trigger a test alert in Datadog (or simulate one)
2. Verify AI SRE correlates the alert
3. Verify incident channel is created in Slack
4. Verify Jira ticket is created
5. Verify context message is posted with correct information
6. Verify runbook link is included
7. Post a resolution update
8. Verify post-mortem is generated
```

---

## Ongoing Maintenance

### Weekly
- Review alert noise (false positives, missing alerts)
- Update routing rules based on team changes
- Check MCP server health and versions

### Monthly
- Review and update runbooks
- Analyze incident trends
- Update SLO targets based on performance
- Rotate API tokens/keys

### Quarterly
- Full security audit of all integrations
- Review and archive old incident channels
- Update escalation matrices
- Evaluate new MCP servers and tools
