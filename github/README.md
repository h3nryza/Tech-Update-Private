# GitHub Integration with Claude Code

## Overview

Claude Code has first-class GitHub integration through the official `claude-code-action` GitHub Action, the `gh` CLI, and MCP servers. This enables AI-powered PR reviews, issue triage, CI/CD automation, and autonomous code changes directly within your GitHub workflow.

## Architecture

```mermaid
graph TB
    subgraph "GitHub"
        ISSUES[Issues]
        PRS[Pull Requests]
        ACTIONS[GitHub Actions]
        COMMENTS[PR Comments / @claude]
        RELEASES[Releases]
    end

    subgraph "Claude Code"
        CLI[Claude Code CLI]
        SDK[Claude Agent SDK]
        ACTION[claude-code-action]
    end

    subgraph "MCP Layer"
        GH_MCP[GitHub MCP Server]
        GH_CLI[gh CLI tools]
    end

    subgraph "Capabilities"
        REVIEW[Code Review]
        TRIAGE[Issue Triage]
        IMPLEMENT[Implementation]
        CICD[CI/CD Automation]
    end

    COMMENTS -->|@claude mention| ACTION
    ISSUES -->|Assignment| ACTION
    ACTION --> SDK
    SDK --> REVIEW
    SDK --> TRIAGE
    SDK --> IMPLEMENT
    CLI --> GH_MCP
    CLI --> GH_CLI
    GH_MCP --> PRS
    GH_MCP --> ISSUES
    GH_CLI --> ACTIONS
    GH_CLI --> RELEASES
```

## Quick Start

### 1. Install the GitHub App

```bash
# In your repo, run:
claude /install-github-app
```

This guides you through:
- Installing the Claude Code GitHub App
- Setting up the `ANTHROPIC_API_KEY` secret
- Creating the initial workflow file

### 2. Manual Setup

If you prefer manual setup:

```yaml
# .github/workflows/claude.yml
name: Claude Code
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
  issues:
    types: [opened, assigned]
  pull_request:
    types: [opened, synchronize]

jobs:
  claude:
    if: |
      (github.event_name == 'issue_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'pull_request_review_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'issues' && contains(github.event.issue.labels.*.name, 'claude')) ||
      (github.event_name == 'pull_request')
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
          model: claude-sonnet-4-20250514
```

### 3. Configure MCP for Local Development

```bash
# Add GitHub MCP server for local Claude Code
claude mcp add github \
  --transport stdio \
  -- npx -y @anthropic-ai/github-mcp-server
```

## Key Features

### Mention @claude on PRs
Comment `@claude` on any PR to get:
- Code review with actionable feedback
- Bug detection and security analysis
- Performance improvement suggestions
- Implementation of requested changes

### Mention @claude on Issues
Comment `@claude fix this` on an issue to have Claude:
- Analyze the issue description
- Create a branch
- Implement the fix
- Run tests
- Open a PR

### Automated PR Review
Configure automatic review on every PR:

```yaml
# .github/workflows/claude-review.yml
name: Claude PR Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            Review this PR for:
            1. Bugs and logic errors
            2. Security vulnerabilities
            3. Performance issues
            4. Code style and best practices
            Provide specific, actionable feedback with code suggestions.
          model: claude-sonnet-4-20250514
```

## Configuration Options

| Parameter | Description | Default |
|-----------|-------------|---------|
| `anthropic_api_key` | API key (required) | - |
| `model` | Model to use | `claude-sonnet-4-20250514` |
| `prompt` | Custom instruction prompt | Auto-detect from context |
| `max_tokens` | Max response tokens | 4096 |
| `timeout_minutes` | Action timeout | 10 |
| `allowed_tools` | Tools Claude can use | All |
| `disallowed_tools` | Tools to block | None |

## File Index

- [skills.md](skills.md) - GitHub skills (PR review, issue triage, CI/CD)
- [agents.md](agents.md) - GitHub agents
- [slash_commands.md](slash_commands.md) - GitHub slash commands
- [actions_integration.md](actions_integration.md) - GitHub Actions integration guide

## Sources

- [Claude Code GitHub Actions Docs](https://code.claude.com/docs/en/github-actions)
- [claude-code-action Repository](https://github.com/anthropics/claude-code-action)
- [Claude Code Action - GitHub Marketplace](https://github.com/marketplace/actions/claude-code-action-official)
- [GitHub Actions Workflow Recipes](https://systemprompt.io/guides/claude-code-github-actions)
