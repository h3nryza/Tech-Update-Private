# AI SRE Architecture

## Overview

This document describes the complete architecture of the AI SRE system, showing how AWS, GitHub, Datadog, Atlassian (Jira + Confluence), Slack, and Cloudflare connect through the AI SRE brain powered by Claude Code and the Agent SDK.

---

## High-Level Architecture

```mermaid
graph TB
    subgraph "Human Interface Layer"
        SLACK_UI[Slack<br/>Slash Commands + ChatOps]
        GH_UI[GitHub<br/>PRs + Issues + Actions]
        JIRA_UI[Jira<br/>Tickets + Boards]
        CONF_UI[Confluence<br/>Runbooks + Post-Mortems]
    end

    subgraph "AI SRE Core"
        direction TB
        ORCHESTRATOR[Agent Orchestrator]
        subgraph "Agent Fleet"
            IC[Incident Commander]
            RCA[Root Cause Analyzer]
            PMW[Post-Mortem Writer]
            AC[Alert Correlator]
            RE[Runbook Executor]
            HM[Health Monitor]
        end
        subgraph "Skills Library"
            S_INC[Incident Skills]
            S_INV[Investigation Skills]
            S_REM[Remediation Skills]
            S_DOC[Documentation Skills]
            S_MON[Monitoring Skills]
        end
        STATE[State Manager<br/>+ Audit Log]
    end

    subgraph "MCP Integration Layer"
        MCP_SLACK[Slack MCP Server]
        MCP_AWS[AWS MCP Server]
        MCP_DD[Datadog MCP Server]
        MCP_GH[GitHub MCP Server]
        MCP_JIRA[Atlassian MCP Server]
        MCP_CF[Cloudflare MCP Server]
    end

    subgraph "Platform Layer"
        direction TB
        subgraph "AWS"
            ECS[ECS / Fargate]
            RDS[RDS / Aurora]
            CW[CloudWatch]
            ECR[ECR]
            S3[S3]
            LAMBDA[Lambda]
            CF_FRONT[CloudFront]
        end
        subgraph "GitHub"
            REPOS[Repositories]
            ACTIONS[Actions / CI]
            DEPLOYS_GH[Deployments]
        end
        subgraph "Datadog"
            APM[APM / Traces]
            LOGS[Log Management]
            METRICS[Metrics]
            MONITORS[Monitors]
            SLOS[SLOs]
            RUM[Real User Monitoring]
        end
        subgraph "Atlassian"
            JIRA_API_INT[Jira Cloud]
            CONFLUENCE[Confluence]
        end
        subgraph "Slack Platform"
            CHANNELS[Channels]
            EVENTS[Events API]
            CMDS[Slash Commands]
        end
        subgraph "Cloudflare"
            DNS[DNS]
            WAF[WAF / DDoS]
            WORKERS[Workers]
            PAGES[Pages]
            ANALYTICS_CF[Analytics]
        end
    end

    %% Human interfaces connect to AI SRE
    SLACK_UI --> MCP_SLACK
    GH_UI --> MCP_GH
    JIRA_UI --> MCP_JIRA
    CONF_UI --> MCP_JIRA

    %% AI SRE Core connections
    ORCHESTRATOR --> IC
    ORCHESTRATOR --> RCA
    ORCHESTRATOR --> PMW
    ORCHESTRATOR --> AC
    ORCHESTRATOR --> RE
    ORCHESTRATOR --> HM
    IC --> S_INC
    RCA --> S_INV
    RE --> S_REM
    PMW --> S_DOC
    HM --> S_MON
    ORCHESTRATOR --> STATE

    %% MCP connections from skills
    S_INC --> MCP_SLACK
    S_INC --> MCP_AWS
    S_INC --> MCP_DD
    S_INC --> MCP_GH
    S_INC --> MCP_JIRA
    S_INC --> MCP_CF
    S_INV --> MCP_DD
    S_INV --> MCP_AWS
    S_INV --> MCP_GH
    S_INV --> MCP_CF
    S_REM --> MCP_AWS
    S_REM --> MCP_CF
    S_REM --> MCP_DD
    S_DOC --> MCP_JIRA
    S_DOC --> MCP_SLACK
    S_MON --> MCP_DD
    S_MON --> MCP_AWS
    S_MON --> MCP_CF

    %% MCP to platforms
    MCP_SLACK --> CHANNELS
    MCP_SLACK --> EVENTS
    MCP_SLACK --> CMDS
    MCP_AWS --> ECS
    MCP_AWS --> RDS
    MCP_AWS --> CW
    MCP_AWS --> ECR
    MCP_AWS --> S3
    MCP_AWS --> LAMBDA
    MCP_DD --> APM
    MCP_DD --> LOGS
    MCP_DD --> METRICS
    MCP_DD --> MONITORS
    MCP_DD --> SLOS
    MCP_GH --> REPOS
    MCP_GH --> ACTIONS
    MCP_GH --> DEPLOYS_GH
    MCP_JIRA --> JIRA_API_INT
    MCP_JIRA --> CONFLUENCE
    MCP_CF --> DNS
    MCP_CF --> WAF
    MCP_CF --> WORKERS
    MCP_CF --> ANALYTICS_CF
```

---

## Data Flow: Incident Lifecycle

This diagram shows how data flows through all 6 platforms during a complete incident from detection to post-mortem.

```mermaid
sequenceDiagram
    participant DD as Datadog
    participant CW as CloudWatch
    participant CF as Cloudflare
    participant AI as AI SRE Core
    participant Slack
    participant GH as GitHub
    participant AWS as AWS (ECS)
    participant Jira
    participant Confluence

    Note over DD,Confluence: Phase 1: Detection
    DD->>AI: Alert: High error rate (webhook)
    CW->>AI: Alarm: ECS task failures (EventBridge)
    CF->>AI: Event: Origin 5xx spike (webhook)
    AI->>AI: Correlate: 3 signals -> single incident

    Note over DD,Confluence: Phase 2: Declaration
    AI->>Slack: Create incident channel
    AI->>Slack: Post context (severity, services, dashboards)
    AI->>Slack: Page on-call responders
    AI->>Jira: Create incident ticket (INC-456)

    Note over DD,Confluence: Phase 3: Investigation
    AI->>GH: Query: Recent deployments
    GH-->>AI: Deploy v2.3.1 at 14:30 (PR #459)
    AI->>DD: Query: Error logs + APM traces
    DD-->>AI: NullPointerException at OrderService:142
    AI->>GH: Query: PR #459 diff
    GH-->>AI: Changed PaymentResponse parsing
    AI->>CF: Query: Edge health
    CF-->>AI: Origin 5xx rate 8%, no edge issues
    AI->>Slack: Post: Root cause identified

    Note over DD,Confluence: Phase 4: Mitigation
    AI->>Slack: Propose: Rollback to v2.3.0
    Slack-->>AI: Human approves (reaction)
    AI->>AWS: Update ECS task definition
    AWS-->>AI: New tasks starting
    AI->>DD: Monitor error rate
    DD-->>AI: Error rate dropping
    AI->>CF: Verify edge health
    CF-->>AI: Origin errors resolved

    Note over DD,Confluence: Phase 5: Resolution
    AI->>DD: Confirm metrics at baseline (10 min)
    AI->>Slack: Post resolution summary
    AI->>Jira: Update INC-456 -> Resolved

    Note over DD,Confluence: Phase 6: Post-Mortem
    AI->>Slack: Read incident channel history
    AI->>DD: Snapshot metrics (before/during/after)
    AI->>GH: Gather PR and deploy data
    AI->>Confluence: Create post-mortem page
    AI->>Jira: Create action item tickets
    AI->>Slack: Post post-mortem link
```

---

## Platform Integration Details

### AWS Integration

```mermaid
graph LR
    subgraph "AI SRE -> AWS"
        direction TB
        READ_AWS[Read Operations]
        WRITE_AWS[Write Operations]
    end

    subgraph "Read — no approval needed"
        R1[Describe ECS services/tasks]
        R2[Query CloudWatch metrics/logs]
        R3[Describe RDS instances]
        R4[List CloudFormation events]
        R5[Get ECR image details]
        R6[Describe Lambda functions]
        R7[Query Cost Explorer]
        R8[Describe EC2 instances]
        R9[Describe ElastiCache clusters]
    end

    subgraph "Write — requires approval"
        W1[Update ECS service<br/>task def / desired count]
        W2[Force ECS new deployment]
        W3[Modify RDS instance class]
        W4[Update Lambda concurrency]
        W5[Invalidate CloudFront cache]
    end

    READ_AWS --> R1
    READ_AWS --> R2
    READ_AWS --> R3
    READ_AWS --> R4
    READ_AWS --> R5
    READ_AWS --> R6
    READ_AWS --> R7
    READ_AWS --> R8
    READ_AWS --> R9
    WRITE_AWS --> W1
    WRITE_AWS --> W2
    WRITE_AWS --> W3
    WRITE_AWS --> W4
    WRITE_AWS --> W5
```

**IAM Policy** (least privilege for the AI SRE):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadAccess",
      "Effect": "Allow",
      "Action": [
        "ecs:Describe*",
        "ecs:List*",
        "rds:Describe*",
        "cloudwatch:GetMetricData",
        "cloudwatch:DescribeAlarms",
        "logs:FilterLogEvents",
        "logs:GetLogEvents",
        "logs:DescribeLogGroups",
        "ecr:DescribeImages",
        "ecr:ListImages",
        "cloudformation:DescribeStacks",
        "cloudformation:DescribeStackEvents",
        "lambda:GetFunction",
        "lambda:ListFunctions",
        "ec2:Describe*",
        "elasticache:Describe*",
        "s3:GetBucketLocation",
        "s3:ListBucket",
        "ce:GetCostAndUsage",
        "ce:GetCostForecast",
        "cloudfront:GetDistribution",
        "cloudfront:ListDistributions",
        "ssm:GetParameter",
        "ssm:GetParametersByPath"
      ],
      "Resource": "*"
    },
    {
      "Sid": "WriteAccessECS",
      "Effect": "Allow",
      "Action": [
        "ecs:UpdateService",
        "ecs:RegisterTaskDefinition"
      ],
      "Resource": [
        "arn:aws:ecs:*:*:service/production/*",
        "arn:aws:ecs:*:*:task-definition/*"
      ],
      "Condition": {
        "StringEquals": {
          "aws:RequestTag/managed-by": "ai-sre"
        }
      }
    },
    {
      "Sid": "WriteAccessLambda",
      "Effect": "Allow",
      "Action": [
        "lambda:PutFunctionConcurrency"
      ],
      "Resource": "arn:aws:lambda:*:*:function:*",
      "Condition": {
        "StringEquals": {
          "aws:ResourceTag/managed-by": "ai-sre"
        }
      }
    },
    {
      "Sid": "WriteAccessCloudFront",
      "Effect": "Allow",
      "Action": [
        "cloudfront:CreateInvalidation"
      ],
      "Resource": "*"
    }
  ]
}
```

---

### GitHub Integration

```mermaid
graph LR
    subgraph "AI SRE -> GitHub"
        direction TB
        GH_READ[Read Operations]
        GH_WRITE[Write Operations]
        GH_ACTIONS[GitHub Actions Triggers]
    end

    subgraph "Read"
        GR1[List deployments]
        GR2[Get PR details + diff]
        GR3[List commits / compare]
        GR4[Get workflow run status]
        GR5[Read repository files]
        GR6[List branches and tags]
    end

    subgraph "Write"
        GW1[Create issue comment]
        GW2[Create PR review]
        GW3[Add labels to issues/PRs]
        GW4[Create issue<br/>for action items]
        GW5[Create PR for fixes]
    end

    subgraph "Actions"
        GA1[Trigger rollback workflow]
        GA2[Trigger deploy workflow]
        GA3[Trigger canary workflow]
    end

    GH_READ --> GR1
    GH_READ --> GR2
    GH_READ --> GR3
    GH_READ --> GR4
    GH_READ --> GR5
    GH_READ --> GR6
    GH_WRITE --> GW1
    GH_WRITE --> GW2
    GH_WRITE --> GW3
    GH_WRITE --> GW4
    GH_WRITE --> GW5
    GH_ACTIONS --> GA1
    GH_ACTIONS --> GA2
    GH_ACTIONS --> GA3
```

**GitHub Actions Workflow for AI SRE**:

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

**GitHub Actions Workflow for AI SRE Rollback**:

```yaml
# .github/workflows/ai-sre-rollback.yml
name: AI SRE Rollback
on:
  workflow_dispatch:
    inputs:
      service:
        description: "Service to rollback"
        required: true
      target_version:
        description: "Target version (task def revision)"
        required: true
      incident_id:
        description: "Associated incident ID"
        required: true
      approved_by:
        description: "User who approved the rollback"
        required: true

jobs:
  rollback:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Audit log
        run: |
          echo "Rollback initiated by AI SRE"
          echo "Service: ${{ inputs.service }}"
          echo "Target: ${{ inputs.target_version }}"
          echo "Incident: ${{ inputs.incident_id }}"
          echo "Approved by: ${{ inputs.approved_by }}"

      - name: Execute rollback
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          service: ${{ inputs.service }}
          cluster: production
          task-definition: ${{ inputs.target_version }}
          wait-for-service-stability: true

      - name: Verify health
        run: |
          sleep 300
          ERROR_RATE=$(curl -s "https://api.datadoghq.com/api/v1/query?query=avg:trace.http.request.errors{service:${{ inputs.service }}}.as_count()" \
            -H "DD-API-KEY: ${{ secrets.DD_API_KEY }}" \
            -H "DD-APPLICATION-KEY: ${{ secrets.DD_APP_KEY }}" | jq '.series[0].pointlist[-1][1]')
          echo "Error rate after rollback: $ERROR_RATE"
```

---

### Datadog Integration

```mermaid
graph LR
    subgraph "Datadog -> AI SRE — Push"
        DW1[Monitor alerts<br/>via webhook]
        DW2[SLO burn rate<br/>alerts]
        DW3[Anomaly detection<br/>alerts]
    end

    subgraph "AI SRE -> Datadog — Pull"
        DQ1[Query metrics<br/>time series]
        DQ2[Search logs<br/>pattern matching]
        DQ3[Get APM traces<br/>distributed tracing]
        DQ4[Get service map<br/>dependency graph]
        DQ5[Query SLO status<br/>error budget]
        DQ6[List monitors<br/>in ALERT state]
        DQ7[Create/update<br/>monitors]
        DQ8[Create dashboard<br/>investigation notebook]
    end

    DW1 --> CORRELATOR[Alert Correlator]
    DW2 --> CORRELATOR
    DW3 --> CORRELATOR

    INVESTIGATOR[Root Cause Analyzer] --> DQ1
    INVESTIGATOR --> DQ2
    INVESTIGATOR --> DQ3
    INVESTIGATOR --> DQ4
    MONITOR[Health Monitor] --> DQ5
    MONITOR --> DQ6
    POSTMORTEM[Post-Mortem Writer] --> DQ1
    POSTMORTEM --> DQ2
```

**Datadog Webhook Configuration**:

```json
{
  "name": "ai-sre-webhook",
  "url": "https://your-ai-sre-endpoint/webhooks/datadog",
  "payload": {
    "alert_id": "$ALERT_ID",
    "alert_title": "$ALERT_TITLE",
    "alert_status": "$ALERT_STATUS",
    "alert_query": "$ALERT_QUERY",
    "alert_scope": "$ALERT_SCOPE",
    "alert_metric": "$ALERT_METRIC",
    "alert_transition": "$ALERT_TRANSITION",
    "hostname": "$HOSTNAME",
    "org_name": "$ORG_NAME",
    "tags": "$TAGS",
    "last_updated": "$LAST_UPDATED",
    "event_type": "$EVENT_TYPE",
    "link": "$LINK",
    "snapshot": "$SNAPSHOT"
  },
  "headers": {
    "X-AI-SRE-Token": "${AI_SRE_WEBHOOK_TOKEN}"
  }
}
```

**Standard Monitor Set**:

```bash
# Create these monitors for AI SRE integration
claude "Create the following Datadog monitors:
1. High error rate (>2%) for each production service
2. High p99 latency (>2s) for each production service
3. ECS task failures
4. RDS connection count approaching limit (>80%)
5. Deployment event correlation (error spike within 30 min of deploy)
6. SLO burn rate alerts for all defined SLOs
7. Anomaly detection on request throughput

Configure all monitors to:
- Notify the ai-sre-webhook
- Include tag 'managed-by:ai-sre'
- Route to #incidents for critical, #alerts for warning"
```

---

### Atlassian Integration (Jira + Confluence)

```mermaid
graph LR
    subgraph "AI SRE -> Jira"
        J1[Create incident ticket]
        J2[Update ticket status]
        J3[Create action items<br/>from post-mortem]
        J4[Link related tickets]
        J5[Search past incidents<br/>for patterns]
        J6[Track action item<br/>completion]
    end

    subgraph "AI SRE -> Confluence"
        C1[Create post-mortem page]
        C2[Read runbook pages]
        C3[Update runbook pages<br/>after incidents]
        C4[Create investigation<br/>notebooks]
        C5[Search documentation<br/>for known issues]
    end

    subgraph "Jira Project Structure"
        INC_PROJ[INCIDENT Project<br/>Issue Type: Incident]
        ACTION_PROJ[ACTION Project<br/>Issue Type: Action Item]
        SRE_PROJ[SRE Project<br/>Issue Type: Task]
    end

    subgraph "Confluence Space"
        PM_SPACE[SRE / Post-Mortems]
        RB_SPACE[SRE / Runbooks]
        KB_SPACE[SRE / Knowledge Base]
    end

    J1 --> INC_PROJ
    J3 --> ACTION_PROJ
    J6 --> ACTION_PROJ
    C1 --> PM_SPACE
    C2 --> RB_SPACE
    C3 --> RB_SPACE
    C5 --> KB_SPACE
```

**Jira Issue Templates**:

```yaml
# Incident ticket template
incident_ticket:
  project: INCIDENT
  issue_type: Incident
  fields:
    summary: "[SEV-{severity}] {title}"
    priority: "{mapped_priority}"
    labels: ["ai-sre", "incident", "{service}"]
    components: ["{service}"]
    description: |
      h2. Incident Details
      * *Severity:* SEV-{severity}
      * *Detected:* {detection_time}
      * *Service:* {service}
      * *Slack Channel:* #{channel_name}

      h2. Impact
      {impact_description}

      h2. Root Cause
      _To be filled during investigation_

      h2. Resolution
      _To be filled at resolution_
    custom_fields:
      incident_start: "{detection_time}"
      incident_end: null
      incident_commander: "{ic_name}"
      slack_channel: "{channel_url}"

# Action item ticket template
action_item_ticket:
  project: ACTION
  issue_type: "Action Item"
  fields:
    summary: "[AI-{incident_id}] {action_description}"
    priority: "{priority}"
    assignee: "{owner}"
    due_date: "{due_date}"
    labels: ["ai-sre", "post-mortem", "{incident_id}"]
    description: |
      h2. Context
      This action item was identified in the post-mortem for {incident_id}.

      h2. Action Required
      {detailed_description}

      h2. Post-Mortem
      [{incident_id} Post-Mortem|{confluence_url}]

      h2. Acceptance Criteria
      {criteria}
    links:
      - type: "is caused by"
        key: "{incident_ticket_key}"
```

---

### Slack Integration

```mermaid
graph TB
    subgraph "Slack -> AI SRE"
        S_CMD[Slash Commands<br/>/incident /postmortem<br/>/runbook /escalate /status<br/>/investigate /deploy /rollback<br/>/slo /oncall]
        S_MENTION[App Mentions<br/>@ai-sre help]
        S_REACTION[Reactions<br/>:white_check_mark: :x: for approvals]
        S_MSG[Messages<br/>in incident channels]
    end

    subgraph "AI SRE -> Slack"
        S_CREATE[Create channels<br/>incident channels]
        S_POST[Post messages<br/>context, updates, summaries]
        S_PIN[Pin messages<br/>important context]
        S_TOPIC[Set channel topic<br/>severity, status, IC]
        S_INVITE[Invite users<br/>responders]
        S_DM[Direct messages<br/>notifications]
    end

    subgraph "Channel Structure"
        INC_CHAN["#inc-{date}-{slug}<br/>Per-incident channels"]
        INCIDENTS["#incidents<br/>All incident declarations"]
        ALERTS["#alerts<br/>Non-incident alerts"]
        OPS_HEALTH["#ops-health<br/>Health reports"]
        SRE_WEEKLY["#sre-weekly<br/>Weekly reports"]
        POST_MORTEMS["#post-mortems<br/>Post-mortem links"]
        SECURITY["#security<br/>Security events"]
        OPS_COST["#ops-cost<br/>Cost alerts"]
        DEPLOYMENTS["#deploy-{service}<br/>Deployment notifications"]
        RUNBOOK_IMP["#sre-runbook-improvements<br/>Runbook improvement suggestions"]
    end

    S_CMD --> S_CREATE
    S_CMD --> S_POST
    S_MENTION --> S_POST
    S_REACTION --> S_POST

    S_CREATE --> INC_CHAN
    S_POST --> INC_CHAN
    S_POST --> INCIDENTS
    S_POST --> ALERTS
    S_POST --> OPS_HEALTH
    S_POST --> SRE_WEEKLY
    S_POST --> POST_MORTEMS
    S_POST --> SECURITY
    S_POST --> OPS_COST
    S_POST --> DEPLOYMENTS
    S_POST --> RUNBOOK_IMP
    S_PIN --> INC_CHAN
    S_TOPIC --> INC_CHAN
    S_INVITE --> INC_CHAN
```

**Channel Naming Convention**:

| Pattern | Purpose | Example | Lifecycle |
|---------|---------|---------|-----------|
| `#inc-{YYYY-MM-DD}-{slug}` | Active incident | `#inc-2026-03-22-checkout-errors` | Archived 7 days after resolution |
| `#incidents` | All incident declarations | Permanent | Never archived |
| `#alerts` | Non-incident alerts | Permanent | Never archived |
| `#ops-health` | Health reports (15 min) | Permanent | Never archived |
| `#ops-cost` | Cost alerts and reports | Permanent | Never archived |
| `#sre-weekly` | Weekly SRE report | Permanent | Never archived |
| `#post-mortems` | Post-mortem announcements | Permanent | Never archived |
| `#security` | Security events | Permanent | Never archived |
| `#deploy-{service}` | Deployment notifications | Per-service | Never archived |
| `#sre-runbook-improvements` | Runbook improvement suggestions | Permanent | Never archived |
| `#ai-sre-health` | AI SRE self-monitoring | Permanent | Never archived |

---

### Cloudflare Integration

```mermaid
graph LR
    subgraph "Cloudflare -> AI SRE"
        CW1[DDoS mitigation alerts]
        CW2[WAF block events]
        CW3[Origin error spikes]
        CW4[SSL certificate events]
        CW5[Traffic anomalies]
    end

    subgraph "AI SRE -> Cloudflare"
        CR1[Query analytics<br/>traffic, cache, errors]
        CR2[Enable Under Attack mode]
        CR3[Update WAF rules]
        CR4[Enable maintenance page<br/>via Workers]
        CR5[Update DNS records<br/>for failover]
        CR6[Purge cache]
        CR7[Check SSL certificate<br/>expiry]
    end

    subgraph "Cloudflare Services"
        CF_DNS[DNS]
        CF_WAF[WAF / DDoS]
        CF_WORKERS[Workers]
        CF_CACHE[Cache]
        CF_SSL[SSL/TLS]
        CF_ANALYTICS[Analytics]
    end

    CW1 --> CF_WAF
    CW2 --> CF_WAF
    CW3 --> CF_ANALYTICS
    CW4 --> CF_SSL
    CR1 --> CF_ANALYTICS
    CR2 --> CF_WAF
    CR3 --> CF_WAF
    CR4 --> CF_WORKERS
    CR5 --> CF_DNS
    CR6 --> CF_CACHE
    CR7 --> CF_SSL
```

**Cloudflare Worker for Maintenance Page**:

```javascript
// maintenance-page-worker.js
// Deployed as a Cloudflare Worker; toggled via KV flag

const MAINTENANCE_KV = "AI_SRE_FLAGS";

export default {
  async fetch(request, env) {
    const maintenance = await env[MAINTENANCE_KV].get("maintenance_mode");

    if (maintenance === "true") {
      const bypassToken = request.headers.get("X-Bypass-Maintenance");
      if (bypassToken === env.BYPASS_TOKEN) {
        return fetch(request);
      }

      return new Response(maintenanceHTML(), {
        status: 503,
        headers: {
          "Content-Type": "text/html",
          "Retry-After": "300",
        },
      });
    }

    return fetch(request);
  },
};

function maintenanceHTML() {
  return `<!DOCTYPE html>
<html>
<head><title>Scheduled Maintenance</title></head>
<body>
  <h1>We'll be right back</h1>
  <p>We're performing scheduled maintenance. Please check back in a few minutes.</p>
  <p>Status: <a href="https://status.example.com">status.example.com</a></p>
</body>
</html>`;
}
```

---

## Cross-Platform Coordination Flows

### Flow 1: Deployment with Observability

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant GH as GitHub
    participant AWS as AWS (ECS)
    participant DD as Datadog
    participant CF as Cloudflare
    participant Slack
    participant AI as AI SRE

    Dev->>GH: Merge PR to main
    GH->>GH: CI pipeline runs
    GH->>AWS: Deploy new task definition
    GH->>Slack: Post to #deploy-{service}
    AWS->>DD: New tasks emit metrics
    AI->>DD: Monitor error rate (5 min window)

    alt Errors spike
        DD->>AI: Alert: error rate > 2%
        AI->>GH: Identify PR that was just deployed
        AI->>Slack: /incident declare SEV-2 ...
        AI->>AWS: Rollback (after approval)
        AI->>CF: Enable maintenance page (if needed)
    else Healthy
        AI->>Slack: Post in #deploy-{service}: "Deploy healthy"
        AI->>DD: Close observation window
    end
```

### Flow 2: DDoS Attack Response

```mermaid
sequenceDiagram
    participant CF as Cloudflare
    participant AI as AI SRE
    participant Slack
    participant AWS
    participant DD as Datadog
    participant Jira

    CF->>AI: DDoS mitigation triggered (webhook)
    AI->>CF: Query: attack vector, traffic volume, origin impact
    AI->>DD: Query: origin health, error rate
    AI->>AWS: Query: ECS/ALB health

    AI->>Slack: /incident declare SEV-1 DDoS attack detected
    AI->>Slack: Post attack details (vector, volume, geo)

    alt Origin impacted
        AI->>CF: Enable Under Attack mode
        AI->>CF: Tighten WAF rules for attack vector
        AI->>AWS: Scale ECS service (buffer capacity)
        AI->>Slack: Post: "Mitigation actions taken"
    else Edge absorbed
        AI->>Slack: Post: "Attack absorbed at edge, monitoring"
    end

    AI->>DD: Monitor for 30 minutes
    AI->>CF: Query: attack subsided?

    alt Attack subsided
        AI->>CF: Disable Under Attack mode
        AI->>Slack: Post: "Attack subsided, removing mitigations"
        AI->>Jira: Create ticket: review WAF rules
    else Ongoing
        AI->>Slack: /escalate team security-oncall
        AI->>Slack: /escalate vendor cloudflare
    end
```

### Flow 3: Database Failover

```mermaid
sequenceDiagram
    participant CW as CloudWatch
    participant DD as Datadog
    participant AI as AI SRE
    participant Slack
    participant AWS as AWS (RDS)
    participant CF as Cloudflare
    participant Jira

    CW->>AI: Alarm: RDS primary unreachable
    DD->>AI: Alert: DB connection errors across services
    AI->>AI: Correlate: DB failure affecting multiple services

    AI->>Slack: /incident declare SEV-1 RDS primary failure
    AI->>AWS: Describe RDS instance (check auto-failover status)

    alt Auto-failover in progress
        AWS-->>AI: Failover to standby initiated
        AI->>Slack: Post: "RDS auto-failover in progress"
        AI->>DD: Monitor: replication lag + connection errors
        AI->>Slack: Post: "Failover complete, verifying"
    else Manual failover needed
        AI->>Slack: Post: "Manual failover required"
        AI->>Slack: "Promote read replica? React to approve"
        Slack-->>AI: Approved
        AI->>AWS: Promote read replica
    end

    AI->>DD: Verify: connection errors resolved
    AI->>CF: Verify: no edge impact

    AI->>Jira: Create ticket: investigate primary failure
    AI->>Jira: Create ticket: verify backup + replication
```

### Flow 4: Certificate Renewal

```mermaid
sequenceDiagram
    participant CF as Cloudflare
    participant AI as AI SRE
    participant Slack
    participant AWS as AWS (ACM)
    participant Jira

    CF->>AI: Alert: SSL certificate expires in 5 days
    AI->>CF: Query: certificate details, auto-renewal status
    AI->>AWS: Query: ACM certificate status

    alt Cloudflare auto-renewal enabled
        AI->>Slack: Post to #security: "Certificate renewal in progress"
        AI->>CF: Verify: new certificate issued
        AI->>Slack: Post: "Certificate renewed successfully"
    else Manual renewal needed
        AI->>Slack: Post to #security: "Manual certificate renewal required"
        AI->>Jira: Create P1 ticket: renew certificate
        AI->>Slack: Page security-oncall
    end
```

---

## Component Architecture

### MCP Server Topology

```mermaid
graph LR
    subgraph "Claude Code Process"
        CC[Claude Code CLI<br/>+ Agent SDK]
    end

    subgraph "stdio MCP Servers — Local"
        AWS_MCP["@awslabs/core-mcp-server<br/>Transport: stdio"]
        DD_MCP["@datadog/mcp-server<br/>Transport: stdio"]
        GH_MCP["@anthropic-ai/github-mcp-server<br/>Transport: stdio"]
        JIRA_MCP["@anthropic-ai/atlassian-mcp-server<br/>Transport: stdio"]
        SLACK_MCP["@anthropic-ai/slack-mcp-server<br/>Transport: stdio"]
        CF_MCP["@cloudflare/workers-mcp<br/>Transport: stdio"]
    end

    subgraph "Remote MCP Servers"
        AWS_KNOW["AWS Knowledge MCP<br/>mcp.aws.amazon.com<br/>Transport: HTTP/SSE"]
    end

    CC -->|stdio| AWS_MCP
    CC -->|stdio| DD_MCP
    CC -->|stdio| GH_MCP
    CC -->|stdio| JIRA_MCP
    CC -->|stdio| SLACK_MCP
    CC -->|stdio| CF_MCP
    CC -->|HTTP/SSE| AWS_KNOW
```

### Skill-Agent Relationship

```mermaid
graph TB
    subgraph "Agents — Autonomous, Event-Driven"
        IC[Incident Commander]
        AC[Alert Correlator]
        PM[Post-Mortem Writer]
        RCA[Root Cause Analyzer]
        RE[Runbook Executor]
        HM[Health Monitor]
    end

    subgraph "Skills — Invocable, Task-Oriented"
        S_INCIDENT[sre-incident]
        S_TRIAGE[sre-triage]
        S_INVESTIGATE[sre-investigate]
        S_CORRELATE[sre-correlate]
        S_ROLLBACK[sre-rollback]
        S_SCALE[sre-scale]
        S_FAILOVER[sre-failover]
        S_POSTMORTEM[sre-postmortem]
        S_MONITOR[sre-monitor]
        S_SLO[sre-slo]
        S_TIMELINE[sre-timeline]
        S_LOGDIVE[sre-logdive]
        S_HANDOFF[sre-handoff]
        S_COST[sre-cost]
    end

    IC --> S_INCIDENT
    IC --> S_TRIAGE
    IC --> S_TIMELINE

    RCA --> S_INVESTIGATE
    RCA --> S_CORRELATE
    RCA --> S_LOGDIVE

    RE --> S_ROLLBACK
    RE --> S_SCALE
    RE --> S_FAILOVER

    PM --> S_POSTMORTEM
    PM --> S_TIMELINE

    AC --> S_CORRELATE
    AC --> S_TRIAGE

    HM --> S_MONITOR
    HM --> S_SLO
    HM --> S_COST
    HM --> S_HANDOFF

    subgraph "Slash Commands — User-Invoked"
        CMD_INC[/incident]
        CMD_PM[/postmortem]
        CMD_RB[/runbook]
        CMD_ESC[/escalate]
        CMD_STATUS[/status]
        CMD_INV[/investigate]
        CMD_DEP[/deploy]
        CMD_ROLL[/rollback]
        CMD_SLO[/slo]
        CMD_OC[/oncall]
    end

    CMD_INC --> IC
    CMD_PM --> PM
    CMD_RB --> RE
    CMD_ESC --> IC
    CMD_STATUS --> HM
    CMD_INV --> RCA
    CMD_DEP --> RE
    CMD_ROLL --> RE
    CMD_SLO --> HM
    CMD_OC --> HM
```

---

## Security Architecture

```mermaid
graph TB
    subgraph "Secrets Management"
        VAULT[AWS Secrets Manager<br/>or HashiCorp Vault]
        ENV[Environment Variables<br/>in Agent Runtime]
    end

    subgraph "Authentication"
        AWS_AUTH[AWS IAM Role<br/>Least privilege]
        GH_AUTH[GitHub App Token<br/>Scoped to repos]
        DD_AUTH[Datadog API + App Keys<br/>Scoped to read + monitors]
        JIRA_AUTH[Atlassian API Token<br/>Scoped to projects]
        SLACK_AUTH[Slack Bot Token<br/>Scoped to channels]
        CF_AUTH[Cloudflare API Token<br/>Scoped to zones]
    end

    subgraph "Authorization"
        APPROVAL[Human Approval Gates]
        RBAC[Role-Based Access<br/>per command/action]
        AUDIT_LOG[Immutable Audit Log<br/>every action recorded]
    end

    subgraph "Network"
        VPC[Private VPC<br/>Agent runtime]
        TLS[TLS 1.3<br/>all connections]
        IP_ALLOW[IP Allowlisting<br/>webhook sources]
    end

    VAULT --> ENV
    ENV --> AWS_AUTH
    ENV --> GH_AUTH
    ENV --> DD_AUTH
    ENV --> JIRA_AUTH
    ENV --> SLACK_AUTH
    ENV --> CF_AUTH

    AWS_AUTH --> APPROVAL
    GH_AUTH --> APPROVAL
    DD_AUTH --> RBAC
    JIRA_AUTH --> RBAC
    SLACK_AUTH --> RBAC
    CF_AUTH --> APPROVAL

    APPROVAL --> AUDIT_LOG
    RBAC --> AUDIT_LOG
```

### Principle of Least Privilege

| Platform | Read | Write | Admin | Destructive |
|----------|------|-------|-------|-------------|
| AWS | All metrics, logs, configs | Deploy, scale (with approval) | No | No (rollback only) |
| Datadog | All data | Create monitors, dashboards | No | No deletes |
| GitHub | All repos, PRs, issues | Create PRs, comment, trigger workflows | No | No force push |
| Jira | All issues, boards | Create/update issues | No | No project admin |
| Slack | All channels | Send messages, create channels | No | No workspace admin |
| Cloudflare | All zones, analytics | Deploy Workers, manage DNS (with approval) | No | No zone deletion |

### Token Rotation Schedule

| Token | Rotation | Method |
|-------|----------|--------|
| AWS IAM | Continuous | IAM role with STS temporary credentials (auto) |
| GitHub App | 1 hour | GitHub App installation token (auto-refreshes) |
| Datadog API/App | 90 days | Manual rotation, stored in Secrets Manager |
| Atlassian API | 90 days | Manual rotation, stored in Secrets Manager |
| Slack Bot | 365 days | Regenerate in Slack App settings |
| Cloudflare API | 90 days | Manual rotation, stored in Secrets Manager |

---

## Deployment Architecture

```mermaid
graph TB
    subgraph "Agent Runtime — AWS"
        ECS_AGENT[ECS Fargate Task<br/>AI SRE Agent Fleet]
        SIDECAR[Logging Sidecar<br/>CloudWatch + Datadog]
    end

    subgraph "Supporting Infrastructure"
        ALB[Application Load Balancer<br/>Webhook receiver]
        APIGW[API Gateway<br/>Slash command endpoint]
        SQS[SQS Queue<br/>Alert buffer]
        DDB[DynamoDB<br/>Agent state store]
        S3_LOGS[S3 Bucket<br/>Audit logs archive]
        SM[Secrets Manager<br/>API tokens]
    end

    subgraph "Monitoring — Meta"
        DD_META[Datadog<br/>Monitor the monitor]
        CW_META[CloudWatch<br/>Agent health]
        SLACK_META[Slack #ai-sre-health<br/>Self-monitoring]
    end

    ALB --> ECS_AGENT
    APIGW --> SQS
    SQS --> ECS_AGENT
    ECS_AGENT --> DDB
    ECS_AGENT --> S3_LOGS
    ECS_AGENT --> SM
    ECS_AGENT --> SIDECAR
    SIDECAR --> DD_META
    SIDECAR --> CW_META
    ECS_AGENT --> SLACK_META
```

### Deployment Configuration (Terraform)

```hcl
# terraform/ai-sre-agent.tf
resource "aws_ecs_task_definition" "ai_sre" {
  family                   = "ai-sre-agent"
  requires_compatibilities = ["FARGATE"]
  cpu                      = 2048   # 2 vCPU
  memory                   = 4096   # 4 GB
  network_mode             = "awsvpc"

  container_definitions = jsonencode([
    {
      name  = "ai-sre-agent"
      image = "${aws_ecr_repository.ai_sre.repository_url}:latest"
      environment = [
        { name = "AGENT_CONFIG", value = "/config/agent_config.yml" },
        { name = "LOG_LEVEL", value = "INFO" },
      ]
      secrets = [
        { name = "ANTHROPIC_API_KEY", valueFrom = aws_secretsmanager_secret.anthropic.arn },
        { name = "DD_API_KEY", valueFrom = aws_secretsmanager_secret.datadog_api.arn },
        { name = "DD_APP_KEY", valueFrom = aws_secretsmanager_secret.datadog_app.arn },
        { name = "SLACK_BOT_TOKEN", valueFrom = aws_secretsmanager_secret.slack.arn },
        { name = "GITHUB_TOKEN", valueFrom = aws_secretsmanager_secret.github.arn },
        { name = "ATLASSIAN_API_TOKEN", valueFrom = aws_secretsmanager_secret.atlassian.arn },
        { name = "CF_API_TOKEN", valueFrom = aws_secretsmanager_secret.cloudflare.arn },
      ]
      logConfiguration = {
        logDriver = "awslogs"
        options = {
          "awslogs-group"         = "/ecs/ai-sre-agent"
          "awslogs-region"        = var.aws_region
          "awslogs-stream-prefix" = "agent"
        }
      }
    }
  ])
}

resource "aws_ecs_service" "ai_sre" {
  name            = "ai-sre-agent"
  cluster         = aws_ecs_cluster.production.id
  task_definition = aws_ecs_task_definition.ai_sre.arn
  desired_count   = 1
  launch_type     = "FARGATE"

  network_configuration {
    subnets         = var.private_subnets
    security_groups = [aws_security_group.ai_sre.id]
  }
}

resource "aws_dynamodb_table" "ai_sre_state" {
  name         = "ai-sre-state"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "agent_id"
  range_key    = "incident_id"

  attribute {
    name = "agent_id"
    type = "S"
  }
  attribute {
    name = "incident_id"
    type = "S"
  }

  ttl {
    attribute_name = "expires_at"
    enabled        = true
  }
}
```

---

## Failure Modes and Resilience

### What happens when the AI SRE itself fails?

```mermaid
flowchart TD
    A{AI SRE Health Check} -->|Healthy| B[Normal operation]
    A -->|Unhealthy| C{Which component?}

    C -->|Agent runtime down| D[ECS auto-restart<br/>Datadog alerts #ai-sre-health<br/>Fallback: manual incident process]
    C -->|MCP server down| E[Graceful degradation<br/>Skip affected platform<br/>Alert: reduced capability]
    C -->|Anthropic API down| F[Queue incoming events in SQS<br/>Alert SRE team<br/>Fallback: manual process]
    C -->|State store down| G[Continue stateless<br/>Log to local disk<br/>Reconcile when restored]
    C -->|Slack down| H[Fall back to PagerDuty<br/>Log actions to Jira<br/>Post-hoc Slack updates]
```

### Resilience Measures

| Failure | Detection | Mitigation | Recovery |
|---------|-----------|------------|----------|
| Agent crash | ECS health check (30s) | Auto-restart (ECS) | Resume from DynamoDB state |
| Anthropic API outage | Health check + latency | Queue events in SQS | Process queue on recovery |
| Slack outage | API error rate | Route to PagerDuty + email | Replay queued messages |
| Datadog outage | Health check | Fall back to CloudWatch only | Resume dual-source |
| AWS MCP failure | Tool call errors | Alert, skip AWS queries | Restart MCP server |
| State store failure | DynamoDB health | Continue stateless | Reconcile from audit log |
| Cloudflare MCP failure | Tool call errors | Skip edge checks | Restart MCP server |
| GitHub MCP failure | Tool call errors | Skip deploy correlation | Restart MCP server |

### Self-Monitoring

The AI SRE monitors itself using the same tools it uses for production:

```yaml
# self_monitoring.yml
monitors:
  - name: "AI SRE Agent Health"
    type: datadog
    query: "avg:ecs.service.running{service:ai-sre-agent}"
    threshold: "< 1"
    severity: CRITICAL
    notify: "#ai-sre-health, pagerduty:sre-oncall"

  - name: "AI SRE Response Time"
    type: datadog
    query: "avg:ai_sre.command.latency{*}"
    threshold: "> 30s"
    severity: WARNING
    notify: "#ai-sre-health"

  - name: "AI SRE Error Rate"
    type: datadog
    query: "sum:ai_sre.tool_call.errors{*}.as_count() / sum:ai_sre.tool_call.total{*}.as_count() * 100"
    threshold: "> 5%"
    severity: HIGH
    notify: "#ai-sre-health"

  - name: "AI SRE Audit Log Gap"
    type: cloudwatch
    query: "Time since last audit log entry"
    threshold: "> 5m"
    severity: WARNING
    notify: "#ai-sre-health"

  - name: "AI SRE API Cost"
    type: custom
    query: "Daily Anthropic API spend"
    threshold: "> $500/day"
    severity: WARNING
    notify: "#ops-cost"

  - name: "AI SRE SQS Queue Depth"
    type: cloudwatch
    query: "ApproximateNumberOfMessagesVisible for ai-sre-events queue"
    threshold: "> 100"
    severity: WARNING
    notify: "#ai-sre-health"
```

---

## Implementation Roadmap

```mermaid
gantt
    title AI SRE Implementation Roadmap
    dateFormat  YYYY-MM-DD
    section Phase 1: Foundation
    Install MCP servers              :done, p1a, 2026-03-01, 7d
    Configure authentication         :done, p1b, after p1a, 3d
    Set up Slack app                 :done, p1c, after p1a, 5d
    Create base skills               :done, p1d, after p1b, 7d

    section Phase 2: Core Agents
    Alert Correlator (L1)            :active, p2a, 2026-03-22, 14d
    Health Monitor (L1)              :active, p2b, 2026-03-22, 14d
    Incident Commander (L1)          :p2c, after p2a, 14d
    Runbook Executor (L1)            :p2d, after p2b, 14d

    section Phase 3: Advanced
    Root Cause Analyzer (L1)         :p3a, after p2c, 14d
    Post-Mortem Writer (L1)          :p3b, after p2c, 14d
    Promote agents to L2             :p3c, after p3a, 30d

    section Phase 4: Maturity
    Promote agents to L3             :p4a, after p3c, 60d
    Cross-platform failover          :p4b, after p3c, 30d
    Cost optimization agent          :p4c, after p4a, 30d
    Promote select agents to L4      :p4d, after p4a, 90d
```

### Phase 1: Foundation (Weeks 1-3)
- Install and configure all 6 MCP servers
- Set up Slack app with all slash commands
- Create core skills (monitor, investigate, rollback, correlate)
- Establish channel structure and alert routing
- Deploy agent runtime infrastructure (ECS + DynamoDB + SQS)
- Configure webhook endpoints for Datadog, CloudWatch, Cloudflare, GitHub

### Phase 2: Core Agents (Weeks 4-6)
- Deploy Alert Correlator at L1 (advisory mode, recommends but does not act)
- Deploy Health Monitor at L1 (report-only mode)
- Deploy Incident Commander at L1 (human does everything, AI suggests)
- Deploy Runbook Executor at L1 (dry-run mode only)
- Begin tracking success metrics and collecting feedback

### Phase 3: Advanced Agents (Weeks 7-10)
- Deploy Root Cause Analyzer and Post-Mortem Writer
- Promote Alert Correlator and Health Monitor to L2 (supervised execution)
- Begin promoting Incident Commander to L2 for well-understood scenarios
- Conduct first security review of all agent permissions and audit logs
- Run first chaos engineering test against the AI SRE system

### Phase 4: Maturity (Weeks 11-20+)
- Promote agents to L3 (semi-autonomous) based on track record
- Implement cross-region failover orchestration
- Add cost optimization agent
- Evaluate L4 (autonomous) promotion for Alert Correlator
- Conduct quarterly security audit
- Publish internal reliability report on AI SRE effectiveness
