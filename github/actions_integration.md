# Using Claude Code in GitHub Actions

## Overview

The official `claude-code-action` (published by Anthropic) runs Claude Code inside a GitHub Actions runner, giving it full access to your repository, file structure, git history, and diff context. Built on the Claude Agent SDK, it enables AI-powered code review, issue implementation, and CI/CD automation.

## Architecture

```mermaid
graph TB
    subgraph "GitHub Events"
        PR_OPEN[PR Opened]
        PR_COMMENT[PR Comment @claude]
        ISSUE_COMMENT[Issue Comment @claude]
        PUSH[Push to Main]
        SCHEDULE[Cron Schedule]
    end

    subgraph "GitHub Actions Runner"
        CHECKOUT[actions/checkout]
        CLAUDE_ACTION[anthropics/claude-code-action@v1]
        SDK[Claude Agent SDK]
    end

    subgraph "Claude Code Capabilities"
        READ_CODE[Read Repository Code]
        WRITE_CODE[Write/Edit Files]
        RUN_TESTS[Run Tests]
        GH_CLI[Use gh CLI]
        MCP_TOOLS[Use MCP Tools]
    end

    subgraph "Outputs"
        PR_REVIEW[PR Review Comments]
        NEW_PR[New Pull Request]
        ISSUE_COMMENT_OUT[Issue Comments]
        CODE_CHANGES[Code Changes]
    end

    PR_OPEN --> CHECKOUT
    PR_COMMENT --> CHECKOUT
    ISSUE_COMMENT --> CHECKOUT
    PUSH --> CHECKOUT
    SCHEDULE --> CHECKOUT
    CHECKOUT --> CLAUDE_ACTION
    CLAUDE_ACTION --> SDK
    SDK --> READ_CODE
    SDK --> WRITE_CODE
    SDK --> RUN_TESTS
    SDK --> GH_CLI
    SDK --> MCP_TOOLS
    READ_CODE --> PR_REVIEW
    WRITE_CODE --> CODE_CHANGES
    CODE_CHANGES --> NEW_PR
    GH_CLI --> ISSUE_COMMENT_OUT
```

## Setup

### Step 1: Install via CLI (Recommended)

```bash
claude /install-github-app
```

This automatically:
- Installs the Claude Code GitHub App on your repository
- Creates the `ANTHROPIC_API_KEY` secret
- Generates the workflow file

### Step 2: Manual Setup

#### Create the API Key Secret

```bash
# Using gh CLI
gh secret set ANTHROPIC_API_KEY --body "sk-ant-..."
```

#### Create the Workflow File

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
    # Only run on @claude mentions or labeled issues
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

---

## Workflow Recipes

### Recipe 1: Auto PR Review

Review every PR automatically when opened or updated.

```yaml
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
          model: claude-sonnet-4-20250514
          prompt: |
            Review this pull request thoroughly. Check for:

            1. **Bugs**: Logic errors, off-by-one errors, null pointer issues
            2. **Security**: Injection vulnerabilities, hardcoded secrets, auth bypass
            3. **Performance**: N+1 queries, unnecessary re-renders, memory leaks
            4. **Tests**: Missing test coverage for new code paths
            5. **Style**: Consistency with existing codebase patterns

            For each issue found, provide:
            - The file and line number
            - What the problem is
            - A concrete fix using a suggestion block

            End with an overall summary and recommendation (approve or request changes).
```

### Recipe 2: Issue Auto-Implementation

When an issue is labeled `claude`, automatically implement it.

```yaml
name: Claude Auto-Implement
on:
  issues:
    types: [labeled]

jobs:
  implement:
    if: github.event.label.name == 'claude'
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
          prompt: |
            Implement the feature or fix described in this issue.

            Steps:
            1. Read the issue description carefully
            2. Analyze the codebase to understand the relevant code
            3. Create a feature branch named feat/issue-${{ github.event.issue.number }}
            4. Implement the changes
            5. Write tests for new functionality
            6. Run the test suite to verify
            7. Create a pull request that references this issue

            Use `Fixes #${{ github.event.issue.number }}` in the PR description.
          timeout_minutes: 30
```

### Recipe 3: Nightly Security Scan

```yaml
name: Claude Security Scan
on:
  schedule:
    - cron: '0 2 * * *'  # 2 AM daily

jobs:
  security-scan:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      issues: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            Perform a security audit of this codebase:

            1. Check for hardcoded secrets, API keys, or credentials
            2. Review authentication and authorization logic
            3. Check for SQL injection, XSS, and CSRF vulnerabilities
            4. Review dependency versions for known CVEs
            5. Check for insecure configurations

            If any issues are found, create a GitHub issue with:
            - Title: "Security: [brief description]"
            - Label: security
            - Severity classification
            - Remediation steps
```

### Recipe 4: Auto-Fix CI Failures

```yaml
name: Claude CI Doctor
on:
  workflow_run:
    workflows: ["Build & Test"]
    types: [completed]

jobs:
  fix-ci:
    if: github.event.workflow_run.conclusion == 'failure'
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.workflow_run.head_branch }}
          fetch-depth: 0

      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            The CI pipeline just failed. Diagnose and fix the issue.

            1. Run `gh run view ${{ github.event.workflow_run.id }} --log-failed` to see failure logs
            2. Identify the root cause
            3. Fix the issue if it's a code problem (not infrastructure)
            4. Push the fix to the current branch
            5. Comment on the PR explaining what was fixed

            Do NOT fix flaky tests by adding retries. Fix the root cause.
```

### Recipe 5: Release Automation

```yaml
name: Claude Release
on:
  workflow_dispatch:
    inputs:
      release_type:
        description: 'Release type (major, minor, patch)'
        required: true
        type: choice
        options:
          - patch
          - minor
          - major

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            Create a ${{ inputs.release_type }} release:

            1. Determine the new version number (bump ${{ inputs.release_type }})
            2. Generate a changelog from commits since the last release
            3. Update version in package.json / pyproject.toml / relevant files
            4. Create a release PR with the changelog
            5. After merge, create a git tag and GitHub release
```

---

## Configuration Reference

### Action Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `anthropic_api_key` | Anthropic API key | Yes | - |
| `model` | Claude model ID | No | `claude-sonnet-4-20250514` |
| `prompt` | Custom prompt/instructions | No | Auto-detect from event |
| `max_tokens` | Maximum response tokens | No | 4096 |
| `timeout_minutes` | Execution timeout | No | 10 |
| `allowed_tools` | Allowed tool list (JSON array) | No | All tools |
| `disallowed_tools` | Blocked tool list (JSON array) | No | None |
| `mcp_config` | MCP server configuration (JSON) | No | None |

### Using MCP Servers in Actions

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
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
          }
        }
      }
```

### Choosing a Model

| Model | Best For | Cost |
|-------|----------|------|
| `claude-sonnet-4-20250514` | PR reviews, quick tasks | Lower |
| `claude-opus-4-6` | Complex implementations, architecture | Higher |

For implementation tasks, use Opus. For reviews and triage, Sonnet is typically sufficient and more cost-effective.

---

## Security Best Practices

1. **API Key**: Store as a repository or organization secret, never hardcode
2. **Permissions**: Use minimum required permissions in the workflow
3. **Branch Protection**: Require PR reviews even for Claude-generated PRs
4. **Allowed Tools**: Restrict tools for sensitive repositories
5. **Timeout**: Set appropriate timeouts to prevent runaway costs
6. **Model Selection**: Use Sonnet for routine tasks to manage costs

```yaml
# Restrictive example for sensitive repos
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    allowed_tools: '["Read", "Grep", "Glob"]'  # Read-only
    timeout_minutes: 5
    model: claude-sonnet-4-20250514
```

## Sources

- [Claude Code GitHub Actions Docs](https://code.claude.com/docs/en/github-actions)
- [claude-code-action Repository](https://github.com/anthropics/claude-code-action)
- [Claude Code Action Marketplace](https://github.com/marketplace/actions/claude-code-action-official)
- [Workflow Recipes Guide](https://systemprompt.io/guides/claude-code-github-actions)
- [CI/CD Pipeline Design Guide](https://dev.to/myougatheaxo/cicd-pipeline-design-with-claude-code-github-actions-from-zero-to-deploy-2b14)
