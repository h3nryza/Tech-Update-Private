# Claude Code + Google Gemini CLI Integration

## Gemini CLI Overview

Gemini CLI is Google's open-source AI agent for the terminal. It provides access to Gemini 2.5 Pro with a 1M token context window.

- **Repo**: https://github.com/google-gemini/gemini-cli
- **Docs**: https://google-gemini.github.io/gemini-cli/

## Installation

```bash
# Quick start (recommended -- no global install needed)
npx @google/gemini-cli

# Or install globally
npm install -g @google/gemini-cli

# Requires Node.js 20+
```

## Authentication: FREE, No API Key Required

This is the standout feature of Gemini CLI. You only need a personal Google account.

```bash
# First run opens browser for Google login
gemini

# That's it. No API key. No credit card. No subscription.
```

### Free Tier Limits

- **1,000 requests per day** with a personal Google account
- **60 requests per minute**
- Access to **Gemini 2.5 Pro** with **1M token context**
- Includes Google Search grounding
- No credit card required

For most individual developers, this is effectively unlimited free usage. Google measured their own developers' usage and doubled it to set the 1,000 limit.

### Optional: API Key for Higher Limits

```bash
# Only needed if you exceed 1,000 req/day
export GEMINI_API_KEY="AIza..."
gemini
```

## Using Claude Code and Gemini CLI Together

### Workflow 1: Gemini for Large Context, Claude for Reasoning

Gemini's 1M token context is free. Use it to analyze large codebases, then hand findings to Claude:

```bash
# Gemini: Analyze entire codebase (free, handles massive context)
gemini "Read all files in src/ and create a comprehensive map of all function dependencies. Write output to ANALYSIS.md"

# Claude: Use that analysis for deep reasoning
claude "Read ANALYSIS.md and design a refactoring plan to reduce coupling between the modules"
```

### Workflow 2: Gemini for Search Grounding

Gemini CLI has built-in Google Search grounding -- it can look up current docs and APIs:

```bash
# Gemini: Research current best practices (uses Google Search)
gemini "Search for the latest Next.js 15 migration guide and summarize the breaking changes relevant to our codebase"

# Claude: Apply those findings
claude "Read MIGRATION_NOTES.md and implement the Next.js 15 migration changes"
```

### Workflow 3: Free Second Opinion

Use Gemini as a free code reviewer:

```bash
# Claude makes changes
claude "Implement rate limiting middleware for Express"

# Gemini reviews (free)
gemini "Review the git diff for the last commit. Check for security issues, performance problems, and missed edge cases"
```

### Workflow 4: Parallel Implementation

```bash
# tmux pane 1: Claude handles backend
claude "Implement the REST API endpoints for user management"

# tmux pane 2: Gemini handles frontend (free)
gemini "Create React components for the user management UI based on the API spec in api-spec.yaml"
```

## MCP Integration: Claude Code Calling Gemini

### Gemini MCP Server

Let Claude Code call Gemini models via MCP:

```bash
# Install
git clone https://github.com/RLabs-Inc/gemini-mcp.git
cd gemini-mcp
npm install && npm run build
```

Add to your `.mcp.json`:
```json
{
  "mcpServers": {
    "gemini": {
      "command": "node",
      "args": ["/path/to/gemini-mcp/dist/index.js"],
      "env": {
        "GEMINI_API_KEY": "AIza..."
      }
    }
  }
}
```

Now from Claude Code:
```
> Use the gemini tool to analyze this 500KB log file for error patterns
```

### Claude Gemini MCP Slim

A lightweight alternative:

```json
{
  "mcpServers": {
    "gemini-slim": {
      "command": "npx",
      "args": ["-y", "claude-gemini-mcp-slim"],
      "env": {
        "GEMINI_API_KEY": "AIza..."
      }
    }
  }
}
```

## Gemini CLI Configuration

Gemini CLI supports MCP servers too, so you can add shared tools:

```bash
# Gemini CLI config location
~/.gemini/settings.json
```

Example Gemini settings with MCP:
```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "memorix"],
      "env": {}
    }
  }
}
```

## Shared Memory with Memorix

Both Claude Code and Gemini CLI can share persistent memory:

```bash
npm install -g memorix
```

Add Memorix to both tools' MCP configs. Now decisions stored by Claude Code are visible to Gemini CLI and vice versa. Data is stored locally in `~/.memorix/data/`. No API keys, no cloud.

## Key Differences to Leverage

| Aspect | Claude Code | Gemini CLI |
|--------|-------------|------------|
| **Cost** | Requires subscription or API key | Free (1,000 req/day) |
| **Context** | 1M tokens (Opus 4.6) | 1M tokens (Gemini 2.5 Pro) |
| **Search** | No built-in web search | Google Search grounding built-in |
| **Strength** | Deep reasoning, complex refactoring | Large context, research, free usage |
| **MCP** | Full MCP support | MCP support |
| **Agent Teams** | Built-in | Not built-in |
| **Best for** | Implementation, review, architecture | Research, analysis, second opinions |

## The "Free AI Development" Stack

For developers who want maximum capability at minimum cost:

1. **Gemini CLI** (free) -- Primary tool for routine coding tasks, research, analysis
2. **Claude Code** (subscription) -- Complex reasoning tasks, architecture decisions
3. **Memorix** (free) -- Shared memory between them, no API keys needed

This gives you two world-class AI models with 1M token context each, plus shared persistent memory.

## Sources

- [Gemini CLI GitHub](https://github.com/google-gemini/gemini-cli)
- [Gemini CLI Docs](https://google-gemini.github.io/gemini-cli/)
- [Gemini CLI Free Tier Details](https://venturebeat.com/ai/google-is-redefining-enterprise-ai-economics-with-open-source-gemini-cli-that-will-be-free-for-the-majority-of-developers)
- [Gemini CLI Quotas and Pricing](https://geminicli.com/docs/resources/quota-and-pricing/)
- [Gemini MCP Server](https://github.com/RLabs-Inc/gemini-mcp)
- [Claude Gemini MCP Slim](https://lobehub.com/mcp/cmdaltctr-claude-gemini-mcp-slim)
- [Memorix](https://github.com/AVIDS2/memorix)
