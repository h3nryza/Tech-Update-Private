# Module 04: MCP Servers Deep Dive

---

## Learning Objectives

By the end of this module, you will be able to:

- [ ] Explain what MCP (Model Context Protocol) is and why it matters
- [ ] Configure MCP servers for Claude Code
- [ ] Connect Claude to GitHub, Slack, databases, and cloud services
- [ ] Build a custom MCP server for your specific needs
- [ ] Manage MCP server security and permissions

---

## 1. What Is MCP?

MCP (Model Context Protocol) is an open protocol that lets AI assistants connect to external tools and data sources. Think of it as **USB for AI** -- a standard way to plug in new capabilities.

### Without MCP

```
You:    "What's the status of PR #42?"
Claude: "I don't have access to GitHub. Please check github.com and
         paste the information here."
```

### With MCP

```
You:    "What's the status of PR #42?"
Claude: "PR #42 'Add user profiles' by @alice:
         - Status: Open, 2 approvals, 1 change requested
         - CI: All checks passing
         - Files changed: 7 (+342, -28)
         - Last updated: 2 hours ago"
```

### How MCP Works

```mermaid
sequenceDiagram
    participant You
    participant CC as Claude Code
    participant MCP as MCP Server
    participant API as External API

    You->>CC: "Check PR #42"
    CC->>CC: Determine: needs GitHub access
    CC->>MCP: Call github.get_pull_request(42)
    MCP->>API: GET /repos/owner/repo/pulls/42
    API-->>MCP: PR data (JSON)
    MCP-->>CC: Structured PR information
    CC->>You: Formatted summary of PR #42
```

---

## 2. Configuring MCP Servers

### Configuration Location

MCP servers are configured in your Claude Code settings:

**Project-level** (`.claude/settings.json`):
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

**Global** (`~/.claude/settings.json`):
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed/dir"]
    }
  }
}
```

### Server Types

| Type | How It Runs | Best For |
|------|------------|----------|
| **stdio** | Local process, communicates via stdin/stdout | Most common, local tools |
| **SSE** | HTTP server with Server-Sent Events | Remote servers, shared tools |

---

## 3. Essential MCP Servers

### GitHub Server

Gives Claude access to repositories, pull requests, issues, and actions.

**Install:**
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

**What Claude can do:**
- Read and create pull requests
- Review code changes
- Manage issues
- Check CI/CD status
- Search repositories

**Example interactions:**
```
"Create a PR from this branch to main with a description of the changes"
"List all open issues labeled 'bug'"
"What's failing in the CI pipeline for PR #42?"
```

### Slack Server

Connects Claude to your Slack workspace.

```json
{
  "mcpServers": {
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}"
      }
    }
  }
}
```

**What Claude can do:**
- Read channel messages
- Post messages and updates
- Search conversations
- Create threads

### PostgreSQL Server

Direct database access for Claude.

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

**What Claude can do:**
- Query the database
- Inspect table schemas
- Analyze data
- Help write and test SQL queries

**Example interactions:**
```
"Show me the schema for the users table"
"How many orders were placed last week?"
"Find users who signed up but never made a purchase"
```

**Safety note:** Use a read-only database user for the MCP server unless you specifically need write access.

### Filesystem Server

Extended file access beyond the project directory.

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/path/to/docs",
        "/path/to/data"
      ]
    }
  }
}
```

### Web Search / Fetch

Let Claude access web content.

```json
{
  "mcpServers": {
    "fetch": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-fetch"]
    }
  }
}
```

---

## 4. Using Multiple MCP Servers Together

The real power comes from combining servers:

```mermaid
graph TB
    CC[Claude Code] --> GH[GitHub MCP]
    CC --> SL[Slack MCP]
    CC --> DB[PostgreSQL MCP]
    CC --> DD[Datadog MCP]

    GH --> |"Read PRs, manage issues"| GITHUB[GitHub]
    SL --> |"Send notifications"| SLACK[Slack]
    DB --> |"Query data"| POSTGRES[PostgreSQL]
    DD --> |"Check metrics"| DATADOG[Datadog]

    style CC fill:#d1c4e9
    style GH fill:#c8e6c9
    style SL fill:#c8e6c9
    style DB fill:#c8e6c9
    style DD fill:#c8e6c9
```

**Example: Full-Stack Debugging**

```
You:   Users are reporting slow checkouts. Investigate.

Claude: Investigating slow checkouts...

        [Using Datadog MCP] Checkout endpoint p99 latency is 4200ms
        (normally 400ms). Spike started at 14:00 UTC.

        [Using GitHub MCP] Last deployment was at 13:55 UTC - PR #87
        "Add discount code validation" by @bob.

        [Using PostgreSQL MCP] The discount_codes table is missing an
        index on the 'code' column. The new validation feature does
        a full table scan for each checkout.

        [Using Slack MCP] Posted in #engineering:
        "Investigating slow checkouts - likely missing DB index from PR #87"

        Root cause: PR #87 added a query on discount_codes.code without
        an index. Fix: add an index.

        Shall I create the migration and a fix PR?
```

---

## 5. Security and Permissions

### The Principle of Least Privilege

| Server | Recommended Permissions |
|--------|----------------------|
| GitHub | Read-only unless creating PRs/issues is needed |
| PostgreSQL | Read-only user (separate user for migrations) |
| Slack | Post to specific channels only |
| Cloud provider | Read-only for monitoring, scoped write for specific actions |
| Filesystem | Minimum necessary directories |

### Environment Variables

Never hardcode tokens in config files:

```bash
# Set in your shell profile (~/.zshrc, ~/.bashrc)
export GITHUB_TOKEN="ghp_xxxxxxxxxxxx"
export SLACK_BOT_TOKEN="xoxb-xxxxxxxxxxxx"
export DATABASE_URL="postgresql://readonly:password@host:5432/mydb"
```

Reference them in MCP config:
```json
{
  "env": {
    "GITHUB_TOKEN": "${GITHUB_TOKEN}"
  }
}
```

### Audit What MCP Servers Can Do

Before adding an MCP server, understand its capabilities:

```bash
# Check what tools an MCP server exposes
npx @modelcontextprotocol/server-github --list-tools
```

Review the tool list and ensure you're comfortable with each capability.

---

## 6. Building a Custom MCP Server

For tools without existing MCP servers, you can build your own.

### Basic Structure (TypeScript)

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server({
  name: "my-custom-server",
  version: "1.0.0",
}, {
  capabilities: {
    tools: {},
  },
});

// Define a tool
server.setRequestHandler("tools/list", async () => ({
  tools: [{
    name: "get_service_status",
    description: "Check the health status of a service",
    inputSchema: {
      type: "object",
      properties: {
        service_name: {
          type: "string",
          description: "Name of the service to check",
        },
      },
      required: ["service_name"],
    },
  }],
}));

// Implement the tool
server.setRequestHandler("tools/call", async (request) => {
  if (request.params.name === "get_service_status") {
    const { service_name } = request.params.arguments;
    // Your logic here -- call an API, check a database, etc.
    const status = await checkServiceHealth(service_name);
    return {
      content: [{
        type: "text",
        text: JSON.stringify(status, null, 2),
      }],
    };
  }
});

// Start the server
const transport = new StdioServerTransport();
await server.connect(transport);
```

### Register Your Custom Server

```json
{
  "mcpServers": {
    "my-custom": {
      "command": "node",
      "args": ["./mcp-servers/my-custom-server/index.js"]
    }
  }
}
```

---

## 7. Try It Yourself

### Exercise 1: Set Up GitHub MCP

1. Create a GitHub personal access token
2. Configure the GitHub MCP server
3. Ask Claude: "What are the open PRs in this repo?"
4. Ask Claude: "Create an issue titled 'Set up CI pipeline'"

### Exercise 2: Connect a Database

1. Set up the PostgreSQL MCP server (use a local dev database)
2. Ask Claude: "What tables exist in this database?"
3. Ask Claude: "Show me the schema for the [table] table"
4. Ask Claude: "Write a query to find [specific data]"

### Exercise 3: Multi-Server Workflow

1. Configure at least 2 MCP servers
2. Ask Claude a question that requires both:
   - "Check if the database schema matches what the code expects"
   - "Find the PR that last modified the user model and check if the migration was included"

---

## Quiz

**Q1: What is MCP and what problem does it solve?**

<details>
<summary>Answer</summary>

MCP (Model Context Protocol) is an open protocol that standardizes how AI assistants connect to external tools and data sources. It solves the problem of AI tools being isolated from the real systems they need to interact with -- without MCP, Claude can only work with what's in your local files. With MCP, it can read GitHub PRs, query databases, post to Slack, check monitoring dashboards, and more.

</details>

**Q2: Why should you use a read-only database user for the PostgreSQL MCP server?**

<details>
<summary>Answer</summary>

Principle of least privilege. If Claude can only read from the database, it can't accidentally delete data, modify records, or drop tables. Use a separate user with write permissions only for specific operations like running migrations, and only when explicitly needed.

</details>

**Q3: How do you safely store MCP server credentials?**

<details>
<summary>Answer</summary>

Store credentials as environment variables in your shell profile (`~/.zshrc` or `~/.bashrc`), not in configuration files. Reference them in MCP config using the `${VARIABLE_NAME}` syntax. This prevents tokens from being committed to git or exposed in config files.

</details>

---

## Next Module

Master advanced patterns and techniques. Continue to [Module 05: Advanced Usage](05_advanced.md).
