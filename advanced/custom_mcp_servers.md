# Custom MCP Servers

> Build your own Model Context Protocol servers to extend Claude Code with custom tools, resources, and prompts.

---

## Table of Contents

- [What is MCP?](#what-is-mcp)
- [Architecture](#architecture)
- [Quick Start: Python MCP Server](#quick-start-python-mcp-server)
- [Quick Start: TypeScript MCP Server](#quick-start-typescript-mcp-server)
- [Registering with Claude Code](#registering-with-claude-code)
- [Advanced Patterns](#advanced-patterns)
- [Testing and Debugging](#testing-and-debugging)
- [Production Deployment](#production-deployment)
- [Real-World Examples](#real-world-examples)

---

## What is MCP?

The Model Context Protocol (MCP) is an open standard that lets AI assistants connect to external data sources and tools through a client-server architecture. Your custom MCP server exposes:

- **Tools**: Functions Claude can call (like an API endpoint)
- **Resources**: Data Claude can read (like a file or database record)
- **Prompts**: Pre-built prompt templates Claude can use

```mermaid
graph LR
    subgraph "Claude Code (Host)"
        Client[MCP Client]
    end

    subgraph "Your MCP Server"
        Server[MCP Server]
        T1[Tool: query_database]
        T2[Tool: create_ticket]
        R1[Resource: schema://tables]
        P1[Prompt: sql_review]
    end

    Client <-->|"JSON-RPC over stdio/SSE"| Server
    Server --> T1
    Server --> T2
    Server --> R1
    Server --> P1
```

---

## Architecture

MCP follows a strict client-server model:

| Component | Role |
|-----------|------|
| **Host** | The application (Claude Code) that the user interacts with |
| **Client** | Maintains a 1:1 connection with a single MCP server |
| **Server** | Exposes tools, resources, and prompts via JSON-RPC |

### Transport Options

| Transport | Use Case | Setup |
|-----------|----------|-------|
| **stdio** | Local development, CLI tools | Server runs as a child process |
| **SSE (Server-Sent Events)** | Remote servers, shared services | Server runs as HTTP service |
| **Streamable HTTP** | Production deployments | Stateless HTTP with optional streaming |

---

## Quick Start: Python MCP Server

### Prerequisites

```bash
pip install "mcp[cli]>=1.2.0"
```

### Starter Template

Create `my_mcp_server.py`:

```python
#!/usr/bin/env python3
"""Custom MCP server template for Claude Code."""

from mcp.server.fastmcp import FastMCP

# Initialize the server
mcp = FastMCP(
    name="my-custom-server",
    version="1.0.0",
)


# --- Tools ---

@mcp.tool()
def search_codebase(query: str, file_type: str = "") -> str:
    """Search the codebase for a pattern and return matching files with context.

    Args:
        query: The search pattern (supports regex)
        file_type: Optional file extension filter (e.g., ".py", ".ts")
    """
    import subprocess

    cmd = ["rg", "--json", "-C", "2", query]
    if file_type:
        cmd.extend(["--glob", f"*{file_type}"])

    result = subprocess.run(cmd, capture_output=True, text=True, timeout=30)

    if result.returncode == 0:
        return result.stdout[:10000]  # Limit output size
    return f"No matches found for: {query}"


@mcp.tool()
def run_sql_query(query: str, database: str = "main") -> str:
    """Execute a read-only SQL query against a project database.

    Args:
        query: The SQL SELECT query to execute
        database: Database name (default: "main")
    """
    import sqlite3

    if not query.strip().upper().startswith("SELECT"):
        return "Error: Only SELECT queries are allowed for safety."

    conn = sqlite3.connect(f"{database}.db")
    try:
        cursor = conn.execute(query)
        columns = [desc[0] for desc in cursor.description]
        rows = cursor.fetchall()
        # Format as a table
        header = " | ".join(columns)
        separator = " | ".join(["---"] * len(columns))
        body = "\n".join(" | ".join(str(v) for v in row) for row in rows[:100])
        return f"{header}\n{separator}\n{body}"
    except Exception as e:
        return f"Query error: {e}"
    finally:
        conn.close()


@mcp.tool()
def create_github_issue(title: str, body: str, labels: str = "") -> str:
    """Create a GitHub issue in the current repository.

    Args:
        title: Issue title
        body: Issue description (markdown)
        labels: Comma-separated labels
    """
    import subprocess

    cmd = ["gh", "issue", "create", "--title", title, "--body", body]
    if labels:
        cmd.extend(["--label", labels])

    result = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
    if result.returncode == 0:
        return f"Issue created: {result.stdout.strip()}"
    return f"Failed to create issue: {result.stderr}"


# --- Resources ---

@mcp.resource("schema://project-structure")
def get_project_structure() -> str:
    """Return the project directory structure."""
    import subprocess

    result = subprocess.run(
        ["find", ".", "-type", "f", "-not", "-path", "./.git/*",
         "-not", "-path", "./node_modules/*", "-not", "-path", "./__pycache__/*"],
        capture_output=True, text=True, timeout=10,
    )
    return result.stdout


@mcp.resource("schema://database/{db_name}")
def get_database_schema(db_name: str) -> str:
    """Return the schema for a SQLite database."""
    import sqlite3

    conn = sqlite3.connect(f"{db_name}.db")
    cursor = conn.execute(
        "SELECT sql FROM sqlite_master WHERE type='table' ORDER BY name"
    )
    schemas = [row[0] for row in cursor.fetchall() if row[0]]
    conn.close()
    return "\n\n".join(schemas)


# --- Prompts ---

@mcp.prompt()
def code_review(file_path: str) -> str:
    """Generate a code review prompt for a specific file."""
    return f"""Review the code in {file_path} for:
1. Bug risks and edge cases
2. Performance issues
3. Security vulnerabilities
4. Code style and readability
5. Missing error handling

Provide specific line-by-line feedback with suggested fixes."""


@mcp.prompt()
def migration_plan(source: str, target: str) -> str:
    """Generate a migration planning prompt."""
    return f"""Create a detailed migration plan from {source} to {target}:

1. Analyze all dependencies on {source}
2. Map each feature/API to its {target} equivalent
3. Identify breaking changes and required adaptations
4. Propose a phased migration order (minimize risk)
5. List required test updates
6. Estimate effort for each phase"""


# --- Run ---

if __name__ == "__main__":
    mcp.run()
```

### Run It

```bash
# Test locally
python my_mcp_server.py

# Or use the MCP Inspector for visual debugging
mcp dev my_mcp_server.py
```

---

## Quick Start: TypeScript MCP Server

### Prerequisites

```bash
npm init -y
npm install @modelcontextprotocol/sdk zod
npm install -D typescript @types/node tsx
```

### Starter Template

Create `src/server.ts`:

```typescript
#!/usr/bin/env node
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";
import { execSync } from "child_process";
import { readFileSync, existsSync } from "fs";

const server = new McpServer({
  name: "my-custom-server",
  version: "1.0.0",
});

// --- Tools ---

server.tool(
  "lint_file",
  "Run the project linter on a specific file and return issues",
  {
    filePath: z.string().describe("Path to the file to lint"),
    autofix: z.boolean().default(false).describe("Auto-fix issues if possible"),
  },
  async ({ filePath, autofix }) => {
    const fixFlag = autofix ? "--fix" : "";
    try {
      const result = execSync(
        `npx eslint ${fixFlag} --format json "${filePath}"`,
        { encoding: "utf-8", timeout: 30000 }
      );
      const parsed = JSON.parse(result);
      const issues = parsed[0]?.messages || [];
      if (issues.length === 0) {
        return { content: [{ type: "text", text: "No lint issues found." }] };
      }
      const report = issues
        .map(
          (i: any) =>
            `Line ${i.line}:${i.column} [${i.severity === 2 ? "error" : "warn"}] ${i.message} (${i.ruleId})`
        )
        .join("\n");
      return { content: [{ type: "text", text: report }] };
    } catch (e: any) {
      return {
        content: [{ type: "text", text: `Lint failed: ${e.message}` }],
        isError: true,
      };
    }
  }
);

server.tool(
  "dependency_check",
  "Check for outdated or vulnerable dependencies",
  {},
  async () => {
    try {
      const audit = execSync("npm audit --json 2>/dev/null", {
        encoding: "utf-8",
        timeout: 60000,
      });
      const parsed = JSON.parse(audit);
      const vulns = parsed.vulnerabilities || {};
      const summary = Object.entries(vulns)
        .map(([name, info]: [string, any]) => `${name}: ${info.severity}`)
        .join("\n");
      return {
        content: [
          {
            type: "text",
            text: summary || "No vulnerabilities found.",
          },
        ],
      };
    } catch (e: any) {
      return {
        content: [{ type: "text", text: `Audit failed: ${e.message}` }],
        isError: true,
      };
    }
  }
);

server.tool(
  "test_file",
  "Run tests for a specific file or test pattern",
  {
    pattern: z.string().describe("Test file pattern or path"),
    verbose: z.boolean().default(false),
  },
  async ({ pattern, verbose }) => {
    const verboseFlag = verbose ? "--verbose" : "";
    try {
      const result = execSync(
        `npx jest ${verboseFlag} --json "${pattern}" 2>/dev/null`,
        { encoding: "utf-8", timeout: 120000 }
      );
      const parsed = JSON.parse(result);
      const summary = `Tests: ${parsed.numPassedTests} passed, ${parsed.numFailedTests} failed, ${parsed.numTotalTests} total`;
      const failures = parsed.testResults
        .flatMap((r: any) => r.assertionResults)
        .filter((a: any) => a.status === "failed")
        .map((a: any) => `FAIL: ${a.fullName}\n  ${a.failureMessages.join("\n  ")}`)
        .join("\n\n");
      return {
        content: [
          { type: "text", text: failures ? `${summary}\n\n${failures}` : summary },
        ],
      };
    } catch (e: any) {
      return {
        content: [{ type: "text", text: `Test run failed: ${e.message}` }],
        isError: true,
      };
    }
  }
);

// --- Resources ---

server.resource(
  "project-config",
  "config://project",
  async (uri) => {
    const configs = ["package.json", "tsconfig.json", ".eslintrc.json"]
      .filter(existsSync)
      .map((f) => `### ${f}\n\`\`\`json\n${readFileSync(f, "utf-8")}\n\`\`\``)
      .join("\n\n");
    return {
      contents: [{ uri: uri.href, mimeType: "text/markdown", text: configs }],
    };
  }
);

// --- Start ---

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("MCP server running on stdio");
}

main().catch(console.error);
```

### Package configuration

Add to `package.json`:

```json
{
  "type": "module",
  "scripts": {
    "build": "tsc",
    "start": "node dist/server.js",
    "dev": "tsx src/server.ts"
  }
}
```

---

## Registering with Claude Code

### Project-Level Registration

Add to `.claude/settings.json`:

```json
{
  "mcpServers": {
    "my-python-server": {
      "command": "python3",
      "args": ["./tools/my_mcp_server.py"],
      "env": {
        "DATABASE_URL": "sqlite:///./data/main.db"
      }
    },
    "my-ts-server": {
      "command": "npx",
      "args": ["tsx", "./tools/src/server.ts"],
      "env": {}
    }
  }
}
```

### User-Level Registration

Add to `~/.claude/settings.json`:

```json
{
  "mcpServers": {
    "my-global-tools": {
      "command": "python3",
      "args": ["/home/user/mcp-tools/server.py"]
    }
  }
}
```

### Verify Registration

```bash
# In Claude Code, check available MCP tools
claude "List all available MCP tools"

# Or use the MCP Inspector
mcp dev ./tools/my_mcp_server.py
```

---

## Advanced Patterns

### Pattern 1: Database-Backed Tool with Connection Pooling

```python
from mcp.server.fastmcp import FastMCP
import asyncpg

mcp = FastMCP("db-tools")

pool = None

@mcp.tool()
async def query_db(sql: str, params: list[str] = []) -> str:
    """Execute a read-only SQL query with parameterized inputs."""
    global pool
    if pool is None:
        pool = await asyncpg.create_pool(
            dsn="postgresql://user:pass@localhost/mydb",
            min_size=2, max_size=10,
        )

    if not sql.strip().upper().startswith("SELECT"):
        return "Error: Only SELECT queries allowed."

    async with pool.acquire() as conn:
        rows = await conn.fetch(sql, *params)
        if not rows:
            return "No results."
        columns = list(rows[0].keys())
        header = " | ".join(columns)
        body = "\n".join(
            " | ".join(str(row[c]) for c in columns) for row in rows[:100]
        )
        return f"{header}\n{body}"
```

### Pattern 2: Tool with Authentication

```python
import os
import httpx
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("api-tools")

@mcp.tool()
async def fetch_jira_issue(issue_key: str) -> str:
    """Fetch a Jira issue by key (e.g., PROJ-123)."""
    base_url = os.environ["JIRA_BASE_URL"]
    token = os.environ["JIRA_API_TOKEN"]
    email = os.environ["JIRA_EMAIL"]

    async with httpx.AsyncClient() as client:
        resp = await client.get(
            f"{base_url}/rest/api/3/issue/{issue_key}",
            auth=(email, token),
        )
        resp.raise_for_status()
        data = resp.json()
        fields = data["fields"]
        return (
            f"**{issue_key}: {fields['summary']}**\n"
            f"Status: {fields['status']['name']}\n"
            f"Assignee: {fields.get('assignee', {}).get('displayName', 'Unassigned')}\n"
            f"Priority: {fields['priority']['name']}\n\n"
            f"{fields.get('description', 'No description')}"
        )
```

### Pattern 3: Composite Tool Server (Multiple Data Sources)

```python
from mcp.server.fastmcp import FastMCP
import subprocess
import json

mcp = FastMCP("dev-tools")

@mcp.tool()
def git_log_analysis(since: str = "1 week ago", author: str = "") -> str:
    """Analyze recent git history for patterns and hot spots.

    Args:
        since: Time range (e.g., "1 week ago", "2026-01-01")
        author: Filter by author name (optional)
    """
    author_flag = f'--author="{author}"' if author else ""
    log = subprocess.run(
        f'git log --since="{since}" {author_flag} --numstat --format="%H|%an|%s"',
        shell=True, capture_output=True, text=True,
    ).stdout

    files_changed = {}
    for line in log.strip().split("\n"):
        if "\t" in line:
            parts = line.split("\t")
            if len(parts) == 3:
                adds, dels, path = parts
                if path not in files_changed:
                    files_changed[path] = {"changes": 0, "additions": 0, "deletions": 0}
                files_changed[path]["changes"] += 1
                files_changed[path]["additions"] += int(adds) if adds != "-" else 0
                files_changed[path]["deletions"] += int(dels) if dels != "-" else 0

    # Sort by change frequency (hot spots)
    hot_spots = sorted(files_changed.items(), key=lambda x: x[1]["changes"], reverse=True)[:20]

    report = "## Hot Spots (Most Changed Files)\n\n"
    report += "| File | Changes | +Lines | -Lines |\n|------|---------|--------|--------|\n"
    for path, stats in hot_spots:
        report += f"| {path} | {stats['changes']} | +{stats['additions']} | -{stats['deletions']} |\n"

    return report
```

---

## Testing and Debugging

### MCP Inspector

The official visual debugging tool:

```bash
# Install
pip install mcp[cli]

# Launch inspector for your server
mcp dev my_mcp_server.py

# Opens a web UI where you can:
# - See all registered tools, resources, prompts
# - Test tool calls with custom inputs
# - Inspect JSON-RPC messages
# - Debug errors in real-time
```

### Unit Testing Tools

```python
import pytest
from my_mcp_server import search_codebase, run_sql_query

def test_search_codebase():
    result = search_codebase("def main", ".py")
    assert "def main" in result or "No matches" in result

def test_sql_query_blocks_writes():
    result = run_sql_query("DROP TABLE users")
    assert "Only SELECT" in result

def test_sql_query_read_only():
    result = run_sql_query("SELECT 1 as test")
    assert "test" in result
```

### Integration Testing

```python
import subprocess
import json

def test_mcp_server_starts():
    """Verify the server starts and responds to initialization."""
    proc = subprocess.Popen(
        ["python3", "my_mcp_server.py"],
        stdin=subprocess.PIPE,
        stdout=subprocess.PIPE,
        stderr=subprocess.PIPE,
    )

    # Send MCP initialize request
    init_request = json.dumps({
        "jsonrpc": "2.0",
        "id": 1,
        "method": "initialize",
        "params": {
            "protocolVersion": "2024-11-05",
            "capabilities": {},
            "clientInfo": {"name": "test", "version": "1.0"}
        }
    })

    proc.stdin.write(f"Content-Length: {len(init_request)}\r\n\r\n{init_request}".encode())
    proc.stdin.flush()

    # Read response (simplified -- real impl needs Content-Length parsing)
    output = proc.stdout.readline()
    proc.terminate()
    assert output  # Server responded
```

---

## Production Deployment

### Docker Deployment

```dockerfile
FROM python:3.12-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY my_mcp_server.py .

# MCP servers communicate over stdio by default
# For remote deployment, use SSE transport
ENV MCP_TRANSPORT=sse
ENV MCP_PORT=8080

EXPOSE 8080
CMD ["python", "my_mcp_server.py"]
```

### SSE Transport for Remote Servers

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("remote-tools")

# ... define tools ...

if __name__ == "__main__":
    import os
    transport = os.environ.get("MCP_TRANSPORT", "stdio")
    if transport == "sse":
        mcp.run(transport="sse", port=int(os.environ.get("MCP_PORT", 8080)))
    else:
        mcp.run()
```

Register the remote server:

```json
{
  "mcpServers": {
    "remote-tools": {
      "url": "http://internal-server:8080/sse"
    }
  }
}
```

---

## Real-World Examples

### 1. Project Knowledge Base Server

A server that indexes your project docs and provides semantic search:

```python
@mcp.tool()
def search_docs(query: str) -> str:
    """Search project documentation using keyword matching."""
    import glob
    results = []
    for f in glob.glob("docs/**/*.md", recursive=True):
        with open(f) as fh:
            content = fh.read()
            if query.lower() in content.lower():
                # Extract surrounding context
                idx = content.lower().index(query.lower())
                snippet = content[max(0, idx-200):idx+200]
                results.append(f"### {f}\n...{snippet}...")
    return "\n\n".join(results[:5]) or "No matching docs found."
```

### 2. Infrastructure Status Server

```python
@mcp.tool()
def check_service_health(service: str) -> str:
    """Check the health of a deployed service."""
    import httpx
    endpoints = {
        "api": "https://api.example.com/health",
        "web": "https://www.example.com/health",
        "worker": "https://worker.example.com/health",
    }
    url = endpoints.get(service)
    if not url:
        return f"Unknown service: {service}. Available: {list(endpoints.keys())}"
    try:
        resp = httpx.get(url, timeout=10)
        return f"{service}: {'healthy' if resp.status_code == 200 else 'unhealthy'} (HTTP {resp.status_code})"
    except Exception as e:
        return f"{service}: unreachable ({e})"
```

### 3. Metrics Query Server

```python
@mcp.tool()
def query_metrics(metric_name: str, duration: str = "1h") -> str:
    """Query application metrics from Prometheus/Datadog."""
    import httpx
    resp = httpx.get(
        f"http://prometheus:9090/api/v1/query",
        params={"query": f'{metric_name}[{duration}]'},
        timeout=15,
    )
    data = resp.json()
    results = data.get("data", {}).get("result", [])
    if not results:
        return f"No data for metric: {metric_name}"
    lines = []
    for r in results:
        labels = r.get("metric", {})
        values = r.get("values", [])
        lines.append(f"Labels: {labels}\nLast value: {values[-1][1] if values else 'N/A'}")
    return "\n\n".join(lines)
```

---

## Sources

- [Build an MCP Server - Official Docs](https://modelcontextprotocol.io/docs/develop/build-server)
- [MCP Server Step-by-Step Guide](https://composio.dev/content/mcp-server-step-by-step-guide-to-building-from-scrtch)
- [How to Build an MCP Server (2026)](https://www.leanware.co/insights/how-to-build-mcp-server)
- [How to Build Your Own MCP Server - Builder.io](https://www.builder.io/blog/mcp-server)
- [How to Build an MCP Server - IBM](https://www.ibm.com/think/tutorials/how-to-build-an-mcp-server)
