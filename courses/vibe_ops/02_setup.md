# Module 02: Setting Up Vibe Ops

---

## Learning Objectives

By the end of this module, you will be able to:

- [ ] Set up a Slack workspace for ChatOps
- [ ] Configure an AI orchestrator (Claude) for operations tasks
- [ ] Connect AI to infrastructure via MCP servers
- [ ] Implement basic safety guardrails for automated operations
- [ ] Verify the end-to-end flow works

---

## 1. Architecture Overview

Before setting up, understand what we're building:

```mermaid
graph TB
    subgraph "Human Layer"
        DEV[Developer]
        SLACK[Slack / Teams]
    end

    subgraph "AI Layer"
        BOT[AI Bot / Agent]
        CLAUDE[Claude / LLM]
    end

    subgraph "Integration Layer"
        MCP1[MCP: GitHub]
        MCP2[MCP: Cloud Provider]
        MCP3[MCP: Monitoring]
        MCP4[MCP: Database]
    end

    subgraph "Infrastructure Layer"
        GH[GitHub Actions]
        AWS[AWS / GCP / Azure]
        DD[Datadog / Grafana]
        DB[Database]
    end

    DEV --> SLACK
    SLACK --> BOT
    BOT --> CLAUDE
    CLAUDE --> MCP1
    CLAUDE --> MCP2
    CLAUDE --> MCP3
    CLAUDE --> MCP4
    MCP1 --> GH
    MCP2 --> AWS
    MCP3 --> DD
    MCP4 --> DB

    style DEV fill:#e1f5fe
    style CLAUDE fill:#d1c4e9
    style MCP1 fill:#c8e6c9
    style MCP2 fill:#c8e6c9
    style MCP3 fill:#c8e6c9
    style MCP4 fill:#c8e6c9
```

---

## 2. Setting Up the Chat Interface

### Option A: Slack (Recommended)

**Step 1: Create a dedicated channel**

Create channels for operations:
- `#ops` -- General operations commands and responses
- `#ops-alerts` -- Automated alerts and notifications
- `#ops-incidents` -- Active incident management

**Step 2: Create a Slack Bot**

1. Go to [api.slack.com/apps](https://api.slack.com/apps)
2. Click "Create New App" > "From scratch"
3. Name it "VibeOps Bot" and select your workspace
4. Under "OAuth & Permissions", add these scopes:
   - `chat:write` -- Send messages
   - `commands` -- Handle slash commands
   - `channels:history` -- Read channel messages
   - `channels:read` -- List channels
   - `files:read` -- Read shared files
5. Install the app to your workspace
6. Copy the Bot User OAuth Token (starts with `xoxb-`)

**Step 3: Set up slash commands**

Create these slash commands under your Slack app:

| Command | Description | Example |
|---------|-------------|---------|
| `/deploy` | Trigger a deployment | `/deploy staging main` |
| `/status` | Check service status | `/status api-service` |
| `/scale` | Scale a service | `/scale api-service 5` |
| `/logs` | Fetch recent logs | `/logs api-service 50` |
| `/incident` | Start an incident | `/incident payment-service down` |

### Option B: Microsoft Teams

The same concepts apply, but you'll use:
- Teams channels instead of Slack channels
- Power Automate or a custom bot for commands
- Microsoft Bot Framework for the AI integration

---

## 3. Setting Up the AI Orchestrator

### Using Claude Code as Your Orchestrator

Claude Code can act as your operations brain. Set it up with operational context:

**Create an operations CLAUDE.md:**

```markdown
# Operations Context

## Infrastructure
- Cloud: AWS (us-east-1, us-west-2)
- Orchestration: Kubernetes (EKS)
- CI/CD: GitHub Actions
- Monitoring: Datadog
- Database: PostgreSQL on RDS

## Services
- api-service: Main REST API (Node.js, 3 replicas default)
- web-frontend: React app served via CloudFront
- worker-service: Background job processor (Python, 2 replicas)
- auth-service: Authentication (Go, 2 replicas)

## Safety Rules
- NEVER modify production without explicit user confirmation
- NEVER delete databases or persistent volumes
- ALWAYS show the planned action and estimated impact before executing
- Scaling above 10 replicas requires cost confirmation
- Database migrations require team lead approval

## Runbooks
- High CPU on api-service: Check for N+1 queries, scale if organic traffic
- Memory leak on worker-service: Restart pods, check recent deployments
- 5xx spike: Check dependent services, recent deployments, database health
```

### Connecting MCP Servers

MCP (Model Context Protocol) servers give Claude access to external tools:

```bash
# Example: Adding GitHub MCP server to Claude Code
# In your claude code settings or .claude.json:
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "your-github-token"
      }
    },
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-your-token"
      }
    }
  }
}
```

---

## 4. Safety Guardrails

This is the most important section. AI-driven operations needs guardrails.

### The Permission Matrix

```mermaid
graph TD
    A[Operation Requested] --> B{What environment?}
    B -->|Development| C[Auto-approve]
    B -->|Staging| D{Destructive?}
    B -->|Production| E{Risk Level?}

    D -->|No| F[Auto-approve]
    D -->|Yes| G[Require confirmation]

    E -->|Low: status check| H[Auto-approve]
    E -->|Medium: scaling, restart| I[Require confirmation]
    E -->|High: deploy, config change| J[Require 2 approvals]
    E -->|Critical: data, security| K[Block - manual only]

    style C fill:#c8e6c9
    style F fill:#c8e6c9
    style H fill:#c8e6c9
    style G fill:#fff9c4
    style I fill:#fff9c4
    style J fill:#ffe0b2
    style K fill:#ffcdd2
```

### Implementing Guardrails

Define guardrails in your operations config:

```yaml
# ops-guardrails.yaml
environments:
  development:
    auto_approve: all

  staging:
    auto_approve:
      - status_check
      - log_query
      - scale_up (max 5 replicas)
      - deploy (from main branch only)
    require_confirmation:
      - scale_down
      - restart_service
      - config_change
    blocked:
      - delete_resource

  production:
    auto_approve:
      - status_check
      - log_query
    require_confirmation:
      - deploy (canary only)
      - scale_up (max 10 replicas)
      - restart_service
    require_team_lead:
      - full_deploy
      - config_change
      - database_migration
    blocked:
      - delete_resource
      - modify_security_groups
      - change_iam_policies
```

### Audit Logging

Every operation must be logged:

```
[2026-03-22T14:30:00Z] USER=@henry ACTION=deploy TARGET=staging/api-service
  SOURCE=main#abc123 APPROVED_BY=auto STATUS=success DURATION=45s
```

---

## 5. A Minimal Working Setup

If you want to start quickly without full infrastructure, here's a minimal local setup:

### Step 1: Create a project directory

```bash
mkdir vibe-ops-lab
cd vibe-ops-lab
```

### Step 2: Create the operations context

Create `CLAUDE.md` with your infrastructure description (even if fictional for learning).

### Step 3: Create mock infrastructure scripts

```bash
mkdir scripts
```

Ask Claude Code:

> Create a set of mock infrastructure scripts in the scripts/ directory that simulate:
> - deploy.sh (prints deployment steps with a 2-second delay between each)
> - status.sh (prints current service status with random health indicators)
> - scale.sh (accepts a service name and replica count, prints the scaling action)
> - logs.sh (generates 20 lines of fake log output with timestamps)
>
> These are for learning purposes -- they should print realistic output but not actually modify anything.

### Step 4: Test the flow

In Claude Code, try:

> Check the status of all services

> Deploy the latest version to staging

> Show me the recent logs for the api-service

> Scale the worker-service to 4 replicas

---

## 6. Try It Yourself

### Exercise: Set Up Your Ops Environment

1. Create a `vibe-ops-lab` directory
2. Write a `CLAUDE.md` describing a fictional infrastructure (3 services, a database, and monitoring)
3. Ask Claude Code to create mock scripts
4. Practice giving natural language operations commands
5. Ask Claude to add a guardrail check before running any "production" commands

<details>
<summary>Sample CLAUDE.md for the exercise</summary>

```markdown
# MyApp Operations

## Infrastructure
- Cloud: AWS (us-east-1)
- Container orchestration: Docker Compose (local), ECS (cloud)
- CI/CD: GitHub Actions
- Monitoring: CloudWatch

## Services
- web-app: React frontend (port 3000)
- api-server: Express.js API (port 8080, 2 replicas)
- background-worker: Python Celery worker (1 replica)

## Database
- PostgreSQL 15 on RDS (db.t3.medium)
- Redis for caching and job queues

## Safety Rules
- Ask for confirmation before any destructive operation
- Show estimated cost impact for scaling operations
- Never modify production without explicit "CONFIRM PRODUCTION" from the user
```

</details>

---

## Quiz

**Q1: What are MCP servers and why are they important for Vibe Ops?**

<details>
<summary>Answer</summary>

MCP (Model Context Protocol) servers are bridges that connect AI assistants like Claude to external tools and services (GitHub, AWS, Slack, Datadog, etc.). They are important because they allow the AI to take real actions on your infrastructure rather than just suggesting commands -- the AI can read statuses, trigger deployments, and query logs directly.

</details>

**Q2: Why are guardrails essential for AI-driven operations?**

<details>
<summary>Answer</summary>

Without guardrails, an AI could execute dangerous operations automatically -- deleting databases, exposing security groups, or incurring massive cloud costs. Guardrails ensure that high-risk operations require human confirmation, critical operations are blocked entirely from automation, and every action is audited for accountability.

</details>

**Q3: What channels should a Vibe Ops Slack workspace have?**

<details>
<summary>Answer</summary>

At minimum: `#ops` for general operations commands, `#ops-alerts` for automated alerts and notifications, and `#ops-incidents` for active incident management. Additional channels can be added for specific services or environments.

</details>

---

## Next Module

The chat interface is your operations command center. Continue to [Module 03: ChatOps Fundamentals](03_chatops.md).
