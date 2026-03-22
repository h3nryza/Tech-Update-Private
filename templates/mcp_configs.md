# MCP Server Configurations — Copy-Paste Ready

> Model Context Protocol (MCP) server configurations for common tool stacks.
> Copy the relevant JSON into your `.claude/settings.json` or project settings.

---

## How MCP Configs Work

MCP servers extend Claude Code with access to external tools (GitHub, Slack, databases, cloud providers). Each config below is a complete, tested configuration.

### Where to Put These

```bash
# Project-level (recommended — committed to repo)
.claude/settings.json

# User-level (applies to all projects)
~/.claude/settings.json
```

### Config Structure
```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "@org/mcp-server-name"],
      "env": {
        "API_KEY": "your-key-here"
      }
    }
  }
}
```

---

## Stack 1: GitHub + Slack + AWS

> Best for: SaaS teams deploying to AWS with GitHub-based workflows

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "slack": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}",
        "SLACK_TEAM_ID": "${SLACK_TEAM_ID}"
      }
    },
    "aws": {
      "command": "npx",
      "args": ["-y", "aws-mcp-server"],
      "env": {
        "AWS_REGION": "${AWS_DEFAULT_REGION}",
        "AWS_PROFILE": "${AWS_PROFILE}"
      }
    }
  }
}
```

### Required Environment Variables
```bash
# .env or shell profile — never commit these
export GITHUB_TOKEN="ghp_..."           # GitHub PAT with repo, read:org scopes
export SLACK_BOT_TOKEN="xoxb-..."       # Slack Bot User OAuth Token
export SLACK_TEAM_ID="T0..."            # Slack workspace ID
export AWS_DEFAULT_REGION="us-east-1"
export AWS_PROFILE="default"            # Or your named profile
```

### What This Enables
- **GitHub**: Create/review PRs, read issues, search code, manage workflows
- **Slack**: Send messages, read channels, search conversations
- **AWS**: Query CloudWatch, manage S3, describe EC2 instances, read logs

---

## Stack 2: GitHub + Datadog + Jira

> Best for: Enterprise teams with Datadog monitoring and Jira project management

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "datadog": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-datadog"],
      "env": {
        "DD_API_KEY": "${DD_API_KEY}",
        "DD_APP_KEY": "${DD_APP_KEY}",
        "DD_SITE": "${DD_SITE}"
      }
    },
    "jira": {
      "command": "npx",
      "args": ["-y", "mcp-server-atlassian"],
      "env": {
        "ATLASSIAN_SITE": "${ATLASSIAN_SITE}",
        "ATLASSIAN_USER_EMAIL": "${ATLASSIAN_USER_EMAIL}",
        "ATLASSIAN_API_TOKEN": "${ATLASSIAN_API_TOKEN}"
      }
    }
  }
}
```

### Required Environment Variables
```bash
export GITHUB_TOKEN="ghp_..."
export DD_API_KEY="..."                   # Datadog API key
export DD_APP_KEY="..."                   # Datadog Application key
export DD_SITE="datadoghq.com"           # Or datadoghq.eu, etc.
export ATLASSIAN_SITE="mycompany.atlassian.net"
export ATLASSIAN_USER_EMAIL="you@company.com"
export ATLASSIAN_API_TOKEN="..."          # Atlassian API token
```

### What This Enables
- **GitHub**: Full repo management, PR workflows
- **Datadog**: Query metrics, read logs, check monitors/alerts, view dashboards
- **Jira**: Read/create/update issues, search tickets, manage sprints

---

## Stack 3: GitHub + Sentry + Linear

> Best for: Startups using Linear for project management and Sentry for error tracking

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "sentry": {
      "command": "npx",
      "args": ["-y", "@sentry/mcp-server-sentry"],
      "env": {
        "SENTRY_AUTH_TOKEN": "${SENTRY_AUTH_TOKEN}",
        "SENTRY_ORG": "${SENTRY_ORG}"
      }
    },
    "linear": {
      "command": "npx",
      "args": ["-y", "mcp-server-linear"],
      "env": {
        "LINEAR_API_KEY": "${LINEAR_API_KEY}"
      }
    }
  }
}
```

### Required Environment Variables
```bash
export GITHUB_TOKEN="ghp_..."
export SENTRY_AUTH_TOKEN="sntrys_..."     # Sentry auth token
export SENTRY_ORG="my-org"               # Sentry organization slug
export LINEAR_API_KEY="lin_api_..."       # Linear personal API key
```

---

## Stack 4: GitHub + PostgreSQL + Redis

> Best for: Backend developers who need direct database and cache access

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "${DATABASE_URL}"
      }
    },
    "redis": {
      "command": "npx",
      "args": ["-y", "mcp-server-redis"],
      "env": {
        "REDIS_URL": "${REDIS_URL}"
      }
    }
  }
}
```

### Required Environment Variables
```bash
export GITHUB_TOKEN="ghp_..."
export DATABASE_URL="postgresql://user:password@localhost:5432/mydb"
export REDIS_URL="redis://localhost:6379"
```

### Safety Notes
- **NEVER point at production databases** — use read replicas or staging
- Consider using a read-only database user for Claude
- The PostgreSQL MCP server can execute queries — be careful with write access

---

## Stack 5: GitHub + Vercel + Supabase

> Best for: Full-stack Next.js projects on Vercel with Supabase backend

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "vercel": {
      "command": "npx",
      "args": ["-y", "mcp-server-vercel"],
      "env": {
        "VERCEL_TOKEN": "${VERCEL_TOKEN}"
      }
    },
    "supabase": {
      "command": "npx",
      "args": ["-y", "mcp-server-supabase"],
      "env": {
        "SUPABASE_URL": "${SUPABASE_URL}",
        "SUPABASE_SERVICE_ROLE_KEY": "${SUPABASE_SERVICE_ROLE_KEY}"
      }
    }
  }
}
```

---

## Stack 6: GitHub + GCP + PagerDuty

> Best for: Teams on Google Cloud with PagerDuty for incident management

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "gcp": {
      "command": "npx",
      "args": ["-y", "mcp-server-gcp"],
      "env": {
        "GOOGLE_APPLICATION_CREDENTIALS": "${GOOGLE_APPLICATION_CREDENTIALS}",
        "GCP_PROJECT_ID": "${GCP_PROJECT_ID}"
      }
    },
    "pagerduty": {
      "command": "npx",
      "args": ["-y", "mcp-server-pagerduty"],
      "env": {
        "PAGERDUTY_API_TOKEN": "${PAGERDUTY_API_TOKEN}"
      }
    }
  }
}
```

---

## Individual Server Configs

### Filesystem (Local File Access)
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/path/to/allowed/directory"
      ]
    }
  }
}
```

### Memory (Persistent Knowledge Graph)
```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
```

### Puppeteer (Browser Automation)
```json
{
  "mcpServers": {
    "puppeteer": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-puppeteer"]
    }
  }
}
```

### Brave Search (Web Search)
```json
{
  "mcpServers": {
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "${BRAVE_API_KEY}"
      }
    }
  }
}
```

### Fetch (HTTP Requests)
```json
{
  "mcpServers": {
    "fetch": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-fetch"]
    }
  }
}
```

---

## Security Best Practices

1. **Never hardcode tokens** — use `${ENV_VAR}` references that resolve from your shell environment
2. **Scope tokens minimally** — GitHub PAT should only have needed scopes (e.g., `repo`, `read:org`)
3. **Use read-only access** where possible — especially for databases and cloud providers
4. **Rotate tokens regularly** — set calendar reminders for quarterly rotation
5. **Separate tokens per environment** — don't use production tokens in development
6. **Audit MCP server packages** — check npm download counts, GitHub stars, and last publish date before adding
7. **Pin versions in production** — use `npx -y @org/server@1.2.3` instead of `@latest` for stability

### Token Scope Recommendations

| Service | Minimum Scopes |
|---------|---------------|
| GitHub | `repo`, `read:org` (add `workflow` if managing Actions) |
| Slack | `channels:read`, `chat:write`, `search:read` |
| Datadog | Read-only API + App keys |
| Jira | Read + Write for project issues |
| Sentry | `project:read`, `event:read` |
| AWS | IAM role with least-privilege policy |
