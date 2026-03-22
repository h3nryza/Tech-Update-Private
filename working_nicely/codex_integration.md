# Claude Code + OpenAI Codex CLI Integration

## Codex CLI Overview

Codex CLI is OpenAI's open-source terminal coding agent. It can read, edit, and execute code with configurable autonomy levels.

- **Repo**: https://github.com/openai/codex
- **Docs**: https://developers.openai.com/codex/cli

## Installation

```bash
npm install -g @openai/codex
```

## Authentication (Free Options)

### Option 1: ChatGPT Account (No API Key)

The first time you run `codex`, it opens a browser for ChatGPT login. This works with:
- ChatGPT Free (limited time offer)
- ChatGPT Plus ($20/mo)
- ChatGPT Pro ($200/mo)
- ChatGPT Business/Enterprise/Edu

```bash
# First run triggers browser auth
codex "explain this codebase"

# Or use device code auth if browser doesn't work
codex login --device-auth
```

### Option 2: API Key

```bash
export OPENAI_API_KEY="sk-..."
codex "explain this codebase"
```

## Codex CLI Approval Modes

```bash
# Suggest mode (default) -- asks approval for everything
codex --approval-mode suggest "refactor auth module"

# Auto-edit mode -- auto-applies file changes, asks for shell commands
codex --approval-mode auto-edit "add error handling"

# Full-auto mode -- no confirmation needed (use with caution)
codex --approval-mode full-auto "run tests and fix failures"

# Shorthand flags
codex --full-auto "run tests and fix failures"

# Switch modes during a session
# Type: /mode auto-edit
```

## Using Claude Code and Codex CLI Together

### Workflow 1: Claude Plans, Codex Executes

Use Claude Code for planning and architecture, then hand off implementation to Codex:

```bash
# Step 1: Claude Code analyzes and creates a plan
claude "Analyze this codebase and create a detailed implementation plan for adding OAuth2 support. Write the plan to PLAN.md"

# Step 2: Codex executes the plan autonomously
codex --full-auto "Read PLAN.md and implement all the changes described in it"
```

### Workflow 2: Parallel Specialists (tmux)

Run both in split panes working on different parts of the codebase:

```bash
# Terminal pane 1: Claude Code handles complex refactoring
claude "Refactor the database layer to use connection pooling"

# Terminal pane 2: Codex handles test writing
codex --auto-edit "Write comprehensive unit tests for the auth module"
```

### Workflow 3: Claude Code Review of Codex Changes

```bash
# Codex makes changes
codex --auto-edit "Add input validation to all API endpoints"

# Claude reviews the diff
claude "Review the git diff and identify any issues, security concerns, or missed edge cases"
```

### Workflow 4: Cross-Verification

Ask the same question to both tools and compare:

```bash
# Ask Claude
claude "What are the security vulnerabilities in src/auth.ts?"

# Ask Codex
codex "What are the security vulnerabilities in src/auth.ts?"

# Compare outputs manually for higher confidence
```

## MCP Bridge: PAL MCP Server

PAL MCP Server lets Claude Code call OpenAI models (and vice versa) through MCP:

```bash
# Install
git clone https://github.com/BeehiveInnovations/pal-mcp-server.git
cd pal-mcp-server
./run-server.sh

# Configure in .mcp.json (project root)
```

Example `.mcp.json`:
```json
{
  "mcpServers": {
    "pal": {
      "command": "node",
      "args": ["/path/to/pal-mcp-server/dist/index.js"],
      "env": {
        "OPENAI_API_KEY": "sk-...",
        "GEMINI_API_KEY": "...",
        "DEFAULT_MODEL": "gemini-2.5-pro"
      }
    }
  }
}
```

With PAL installed, Claude Code can ask GPT models for a second opinion:

```
> Use the pal tool to ask GPT-5.3 to review this authentication implementation
```

## Claude Code Bridge (CCB)

For visual side-by-side operation with split panes:

```bash
git clone https://github.com/bfly123/claude_code_bridge.git
cd claude_code_bridge
./install.sh install
```

CCB provides a split-pane terminal (via tmux or WezTerm) where Claude, Codex, and Gemini each have their own visible pane.

## Key Differences to Leverage

| Aspect | Claude Code | Codex CLI |
|--------|-------------|-----------|
| **Strength** | Complex reasoning, multi-file refactoring | Autonomous execution, sandbox safety |
| **Context** | 1M tokens (Opus 4.6) | Large but model-dependent |
| **Autonomy** | Developer-in-the-loop by default | Full-auto mode available |
| **Best for** | Architecture, planning, review | Implementation, testing, CI/CD tasks |
| **MCP** | Full MCP support | MCP support |
| **Agent Teams** | Built-in agent teams feature | Single agent |

## Sources

- [Codex CLI GitHub](https://github.com/openai/codex)
- [Codex Authentication Docs](https://developers.openai.com/codex/auth)
- [Codex CLI Approval Modes](https://developers.openai.com/codex/cli/features)
- [PAL MCP Server](https://github.com/BeehiveInnovations/pal-mcp-server)
- [Claude Code Bridge](https://github.com/bfly123/claude_code_bridge)
- [Codex vs Claude Code Comparison](https://www.builder.io/blog/codex-vs-claude-code)
