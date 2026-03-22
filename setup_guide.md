# Claude Code Setup Guide

> Step-by-step guide to setting up Claude Code with CLAUDE.md, MCP servers, skills, agents, and optimal folder structure.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Install Claude Code](#install-claude-code)
3. [Initial Configuration](#initial-configuration)
4. [Create Your CLAUDE.md](#create-your-claudemd)
5. [Configure MCP Servers](#configure-mcp-servers)
6. [Set Up Skills](#set-up-skills)
7. [Set Up Subagents](#set-up-subagents)
8. [Configure Permissions & Hooks](#configure-permissions--hooks)
9. [Set Up Folder Structure](#set-up-folder-structure)
10. [Verify Your Setup](#verify-your-setup)
11. [Daily Workflow](#daily-workflow)

---

## Prerequisites

Before starting, ensure you have:

- **Operating System**: macOS or Linux (Windows via WSL)
- **Node.js**: v18+ (required for MCP servers that use `npx`)
- **Git**: Installed and configured
- **A Claude account**: Pro, Team, or Enterprise subscription, or Anthropic Console API access
- **Terminal**: iTerm2, WezTerm, or built-in Terminal

Optional but recommended:
- **GitHub CLI** (`gh`): For GitHub integration
- **jq**: For JSON processing in scripts
- **tmux**: For session persistence (especially for remote usage)

---

## Install Claude Code

### macOS / Linux

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

### Windows (WSL)

```bash
# Inside WSL
curl -fsSL https://claude.ai/install.sh | bash
```

### Verify Installation

```bash
claude --version
claude doctor    # Diagnose any issues
```

### First Launch

```bash
claude
```

On first launch, Claude Code will:
1. Prompt you to log in (browser-based auth)
2. Create `~/.claude/` directory for global configuration
3. Detect your shell and suggest terminal setup

Run terminal setup for optimal keybindings:
```
/terminal-setup
```

---

## Initial Configuration

### Global Personal CLAUDE.md

Create a personal CLAUDE.md that applies to all your sessions:

```bash
mkdir -p ~/.claude
cat > ~/.claude/CLAUDE.md << 'EOF'
# Personal Defaults

## Code Style
- Prefer TypeScript over JavaScript
- Use functional programming patterns where appropriate
- Write self-documenting code; add comments only for "why", not "what"

## Git
- Use conventional commits (feat:, fix:, chore:, docs:, refactor:, test:)
- Never force push to main/master
- Always create feature branches

## Communication
- Be concise in commit messages and PR descriptions
- When unsure about an approach, explain the trade-offs before proceeding

## Testing
- Write tests for new functionality
- Run relevant tests after changes, not the full suite
EOF
```

### Global Settings

Claude Code stores settings in `~/.claude/settings.json`. You can edit directly or use the CLI:

```bash
# Set default model
claude config set model sonnet

# Set default effort level
claude config set effort high
```

---

## Create Your CLAUDE.md

### Option 1: Auto-Generate with /init

In your project directory:

```bash
cd /path/to/your/project
claude
```

Then inside Claude Code:
```
/init
```

This analyzes your codebase and generates a starter CLAUDE.md based on detected build systems, test frameworks, and code patterns.

### Option 2: Create Manually

Create `CLAUDE.md` in your project root:

```bash
cat > CLAUDE.md << 'HEREDOC'
# Project: [Your Project Name]

## Tech Stack
- Language: TypeScript 5.x
- Framework: Next.js 14 (App Router)
- Database: PostgreSQL with Prisma ORM
- Testing: Vitest + Playwright
- Styling: Tailwind CSS

## Common Commands
- `npm run dev` - Start development server
- `npm run build` - Production build
- `npm run test` - Run unit tests (Vitest)
- `npm run test:e2e` - Run E2E tests (Playwright)
- `npm run typecheck` - TypeScript type checking
- `npm run lint` - ESLint
- `npm run db:migrate` - Run database migrations
- `npm run db:generate` - Generate Prisma client after schema changes

## Code Style
- Use ES modules (import/export), not CommonJS
- Destructure imports when possible
- IMPORTANT: Always run typecheck after code changes
- Prefer server components; use 'use client' only when necessary

## Architecture
- App Router pages in src/app/
- Server actions in src/actions/
- Database models in prisma/schema.prisma
- Shared utilities in src/lib/
- React components in src/components/
- API routes in src/app/api/

## Testing
- Prefer running single tests: `npx vitest run path/to/test`
- Write tests alongside source files: `foo.ts` -> `foo.test.ts`
- E2E tests go in tests/e2e/

## Git Conventions
- Branch naming: feature/description, fix/description
- Conventional commits required
- Always include PR description with Summary and Test Plan sections

## Gotchas
- Database migrations require `npm run db:generate` after schema changes
- The CI runs on Node 20
- Environment variables validated at build time - check .env.example
- Rate limiting middleware must be tested with `TEST_RATE_LIMIT=true`

See @README.md for project overview.
See @docs/architecture.md for detailed architecture docs.
HEREDOC
```

### Adding Imports

CLAUDE.md supports `@path/to/file` imports for referencing other files:

```markdown
# CLAUDE.md
See @README.md for project overview.
See @docs/api-conventions.md for API patterns.
See @.claude/rules/testing.md for detailed testing rules.
```

### Path-Specific Rules

For rules that only apply to certain directories:

```bash
mkdir -p .claude/rules

cat > .claude/rules/frontend.md << 'EOF'
# Frontend Rules (src/components/, src/app/)
- Use React Server Components by default
- Only add 'use client' when using hooks or browser APIs
- All components must have TypeScript props interface
- Use Tailwind utility classes, no custom CSS
EOF

cat > .claude/rules/api.md << 'EOF'
# API Rules (src/app/api/)
- All endpoints must validate input with Zod
- Return consistent error format: { error: string, code: number }
- Include rate limiting on all public endpoints
- Log all 5xx errors to monitoring
EOF
```

---

## Configure MCP Servers

### Recommended Starter Set

Set up the most commonly useful MCP servers:

#### 1. GitHub (Essential)

```bash
# Ensure gh CLI is installed first
brew install gh  # macOS
gh auth login

# Add GitHub MCP server (user-scoped)
claude mcp add github \
  --scope user \
  -- npx -y @modelcontextprotocol/server-github
```

Or with environment variable for token:

```bash
export GITHUB_TOKEN=$(gh auth token)
claude mcp add-json github "{
  \"command\": \"npx\",
  \"args\": [\"-y\", \"@modelcontextprotocol/server-github\"],
  \"env\": {
    \"GITHUB_PERSONAL_ACCESS_TOKEN\": \"$GITHUB_TOKEN\"
  }
}" --scope user
```

#### 2. Playwright (Browser Testing)

```bash
claude mcp add playwright \
  --scope user \
  -- npx -y @playwright/mcp@latest
```

#### 3. Filesystem (Enhanced File Operations)

```bash
claude mcp add filesystem \
  --scope user \
  -- npx -y @modelcontextprotocol/server-filesystem /path/to/allowed/directory
```

#### 4. PostgreSQL (If Using Postgres)

```bash
claude mcp add-json postgres "{
  \"command\": \"npx\",
  \"args\": [\"-y\", \"@modelcontextprotocol/server-postgres\"],
  \"env\": {
    \"DATABASE_URL\": \"postgresql://user:pass@localhost:5432/mydb\"
  }
}" --scope local
```

### Project-Level MCP Config

Create `.mcp.json` in your project root (check into git, but keep secrets in environment variables):

```json
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    }
  }
}
```

### Verify MCP Servers

Inside Claude Code:
```
/mcp
```

This shows all configured servers and their connection status.

---

## Set Up Skills

### Create the Skills Directory

```bash
mkdir -p .claude/skills/{deploy,fix-issue,code-review,create-component}
mkdir -p ~/.claude/skills/{explain-code,quick-commit}
```

### Project Skill: Fix GitHub Issue

```bash
cat > .claude/skills/fix-issue/SKILL.md << 'EOF'
---
name: fix-issue
description: Fix a GitHub issue by number
disable-model-invocation: true
argument-hint: "[issue-number]"
---

Fix GitHub issue $ARGUMENTS:

1. Use `gh issue view $ARGUMENTS` to get the issue details
2. Understand the problem described
3. Search the codebase for relevant files
4. Implement the necessary changes
5. Write tests to verify the fix
6. Run tests and ensure they pass
7. Run typecheck and lint
8. Create a descriptive commit following conventional commits
9. Push the branch and create a PR referencing the issue

IMPORTANT: Always create a feature branch named `fix/$ARGUMENTS-short-description`.
EOF
```

### Project Skill: Deploy

```bash
cat > .claude/skills/deploy/SKILL.md << 'EOF'
---
name: deploy
description: Deploy the application to staging or production
disable-model-invocation: true
argument-hint: "[staging|production]"
---

Deploy to $ARGUMENTS:

1. Ensure all tests pass: `npm run test`
2. Ensure typecheck passes: `npm run typecheck`
3. Build the application: `npm run build`
4. If deploying to production:
   - Verify we are on the main branch
   - Check that CI is green
   - Create a git tag with the current version
5. Run the deployment: `npm run deploy:$ARGUMENTS`
6. Verify the deployment health check
7. Report the deployment URL and status
EOF
```

### Project Skill: Code Review

```bash
cat > .claude/skills/code-review/SKILL.md << 'EOF'
---
name: code-review
description: Review code changes for quality, security, and best practices
context: fork
agent: Explore
allowed-tools: Read, Grep, Glob, Bash(git *)
---

## Context
- PR diff: !`git diff main...HEAD`
- Changed files: !`git diff main...HEAD --name-only`

## Review Checklist

Review the changes above for:

1. **Correctness**: Does the code do what it claims?
2. **Security**: Any injection, auth, or data exposure issues?
3. **Performance**: Any N+1 queries, unnecessary loops, or memory leaks?
4. **Testing**: Are edge cases covered? Are tests meaningful?
5. **Style**: Does it follow project conventions?
6. **Documentation**: Are complex parts explained?

Format output as:
- CRITICAL: Must fix before merge
- WARNING: Should fix, but not blocking
- SUGGESTION: Consider improving
- GOOD: Things done well (encourage positive patterns)
EOF
```

### Project Skill: Create Component

```bash
cat > .claude/skills/create-component/SKILL.md << 'EOF'
---
name: create-component
description: Create a new React component following project patterns
argument-hint: "[ComponentName]"
---

Create a new React component named $ARGUMENTS:

1. Look at existing components in src/components/ for patterns
2. Create the component file: `src/components/$ARGUMENTS/$ARGUMENTS.tsx`
3. Create the test file: `src/components/$ARGUMENTS/$ARGUMENTS.test.tsx`
4. Create an index barrel export: `src/components/$ARGUMENTS/index.ts`
5. Follow these conventions:
   - Use TypeScript with explicit Props interface
   - Export as named export, not default
   - Use Tailwind for styling
   - Include JSDoc comment describing the component
6. Write basic unit tests
7. Run typecheck to verify
EOF
```

### Personal Skill: Explain Code

```bash
cat > ~/.claude/skills/explain-code/SKILL.md << 'EOF'
---
name: explain-code
description: Explains code with visual diagrams and analogies. Use when asking how code works.
---

When explaining code, always include:

1. **Start with an analogy**: Compare the code to something from everyday life
2. **Draw a diagram**: Use ASCII art or mermaid to show flow/structure
3. **Walk through step-by-step**: Explain what happens in sequence
4. **Highlight gotchas**: Common mistakes or misconceptions
5. **Summarize**: One-sentence TL;DR

Keep explanations conversational and accessible.
EOF
```

### Personal Skill: Quick Commit

```bash
cat > ~/.claude/skills/quick-commit/SKILL.md << 'EOF'
---
name: quick-commit
description: Create a well-formatted conventional commit
disable-model-invocation: true
argument-hint: "[optional: commit type override]"
---

Create a git commit for the current staged changes:

1. Run `git diff --cached` to see staged changes
2. If nothing is staged, run `git diff` and ask what to stage
3. Determine the appropriate commit type:
   - feat: new feature
   - fix: bug fix
   - refactor: code restructuring
   - docs: documentation
   - test: adding/updating tests
   - chore: maintenance
4. Write a concise commit message following conventional commits
5. Create the commit
6. Show the commit hash and summary

If $ARGUMENTS is provided, use it as the commit type override.
EOF
```

---

## Set Up Subagents

### Create the Agents Directory

```bash
mkdir -p .claude/agents
mkdir -p ~/.claude/agents
```

### Security Reviewer Agent

```bash
cat > .claude/agents/security-reviewer.md << 'EOF'
---
name: security-reviewer
description: Reviews code for security vulnerabilities. Use proactively after code changes.
tools: Read, Grep, Glob, Bash
model: sonnet
memory: project
---

You are a senior security engineer conducting a thorough security review.

When invoked:
1. Run `git diff HEAD~1` to see recent changes (or review specified files)
2. Focus on modified files and their dependencies

Review for:
- **Injection**: SQL injection, XSS, command injection, path traversal
- **Authentication**: Missing auth checks, token handling, session management
- **Authorization**: Privilege escalation, IDOR, missing access controls
- **Data Exposure**: Leaked secrets, PII in logs, verbose errors
- **Dependencies**: Known vulnerabilities in imported packages
- **Configuration**: Insecure defaults, missing security headers

Format findings by severity:
- CRITICAL: Exploitable vulnerability, must fix immediately
- HIGH: Significant risk, fix before merge
- MEDIUM: Moderate risk, should address
- LOW: Minor concern, consider fixing
- INFO: Best practice suggestion

Include specific line references and remediation code.

Update your agent memory with patterns you discover in this codebase.
EOF
```

### Test Writer Agent

```bash
cat > .claude/agents/test-writer.md << 'EOF'
---
name: test-writer
description: Writes comprehensive tests for code. Use when tests are needed.
tools: Read, Write, Edit, Bash, Grep, Glob
model: sonnet
---

You are an expert test engineer. Write thorough, meaningful tests.

When invoked:
1. Read the source code to understand behavior
2. Identify edge cases, error paths, and boundary conditions
3. Write tests following project conventions
4. Run the tests to verify they pass
5. Ensure good coverage of the critical paths

Testing principles:
- Test behavior, not implementation
- Each test should test one thing
- Use descriptive test names: "should [expected behavior] when [condition]"
- Avoid mocks unless testing external service boundaries
- Include positive, negative, and edge case tests
- Test error handling explicitly
EOF
```

### Documentation Agent

```bash
cat > ~/.claude/agents/doc-writer.md << 'EOF'
---
name: doc-writer
description: Generates and updates documentation. Use when docs need writing or updating.
tools: Read, Write, Edit, Grep, Glob
model: sonnet
---

You are a technical writer. Create clear, accurate documentation.

When invoked:
1. Read the relevant source code
2. Understand the public API and usage patterns
3. Write documentation that helps developers use the code

Documentation style:
- Start with a one-line summary
- Include a quick start example
- Document all public APIs with parameters and return types
- Include common usage patterns
- Note gotchas and edge cases
- Keep examples runnable and tested
EOF
```

### Using the /agents Command

You can also create agents interactively:

```
/agents
```

Select "Create new agent" and follow the guided setup. Claude can generate the agent configuration for you based on a description.

---

## Configure Permissions & Hooks

### Permission Configuration

Edit `.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Bash(npx *)",
      "Bash(git *)",
      "Bash(gh *)",
      "Read",
      "Glob",
      "Grep"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(sudo *)"
    ]
  }
}
```

Or configure interactively:
```
/permissions
```

### Set Up Hooks

Hooks run scripts automatically at specific points in Claude's workflow. Unlike CLAUDE.md instructions (which are advisory), hooks are **deterministic**.

#### Auto-Lint After File Edits

Add to `.claude/settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx eslint --fix $CLAUDE_FILE_PATH 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

#### Notification Hook (for Remote Usage)

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "if [ -n \"$SSH_CONNECTION\" ]; then curl -s -d \"Claude finished task\" https://ntfy.sh/your-topic; fi"
          }
        ]
      }
    ]
  }
}
```

You can also create hooks interactively:
```
/hooks
```

Or ask Claude to create them:
```
Write a hook that runs eslint after every file edit
```

---

## Set Up Folder Structure

### For This Framework Repository

```bash
# Create the full directory structure
mkdir -p .claude/{skills,agents,rules}
mkdir -p claudefiles/{decisions,skills,agents,archive}
mkdir -p learning/{sessions,frameworks,guidance,archive}

# Create essential files if they don't exist
touch claudefiles/changelog.md
touch claudefiles/context.md
touch learning/guidance/prompt_guidance.md
touch learning/frameworks/prompt_framework.md
```

### For a New Project

```bash
# Initialize a new project with Claude Code support
cd /path/to/new/project
git init

# Create Claude Code structure
mkdir -p .claude/{skills,agents,rules}

# Generate CLAUDE.md from codebase analysis
claude -p "/init"

# Or create manually
touch CLAUDE.md
touch .mcp.json

# Add to .gitignore
cat >> .gitignore << 'EOF'

# Claude Code local files
.claude/settings.local.json
.claude/agent-memory-local/
EOF
```

### Directory Purpose Reference

```
.claude/                        # Claude Code configuration
  CLAUDE.md                     # Alternative location for project instructions
  settings.json                 # Shared settings (check into git)
  settings.local.json           # Personal overrides (git-ignored)
  skills/                       # Custom slash commands
    <skill-name>/SKILL.md       # One directory per skill
  agents/                       # Custom subagents
    <agent-name>.md             # One file per agent
  rules/                        # Path-specific rules
    <rule-name>.md              # Loaded based on file paths
  agent-memory/                 # Shared agent memory (optional, git)
  agent-memory-local/           # Local agent memory (git-ignored)

CLAUDE.md                       # Root project instructions (check into git)
.mcp.json                       # Shared MCP server config (check into git)
```

---

## Verify Your Setup

Run through this checklist to verify everything works:

### 1. Health Check

```
claude doctor
```

### 2. Verify CLAUDE.md is Loaded

```
claude
> What project instructions do you see?
```

### 3. Verify MCP Servers

```
/mcp
```

All configured servers should show as connected.

### 4. Verify Skills

```
/help
```

Custom skills should appear in the command list. Try invoking one:

```
/explain-code src/index.ts
```

### 5. Verify Agents

```
/agents
```

Should list your custom agents alongside built-in ones.

### 6. Test a Full Workflow

```
# Start with plan mode
/plan

# Explore the codebase
> Read the project structure and tell me about the architecture

# Switch to normal mode (Shift+Tab)
> Create a simple utility function with tests

# Verify
> Run the tests

# Clean up
/clear
```

---

## Daily Workflow

### Starting Your Day

```bash
# Resume yesterday's session
claude --continue

# Or start fresh
claude

# Check what's going on
> What were we working on? Check git log for recent changes.
```

### Working on a Feature

```
# 1. Plan (Shift+Tab to plan mode)
> I need to implement [feature]. Read the relevant code and create a plan.

# 2. Review the plan (Ctrl+G to edit in your editor)

# 3. Implement (Shift+Tab back to normal mode)
> Implement the plan. Write tests and run them.

# 4. Review
> Use a subagent to review this code for security issues.

# 5. Commit
/quick-commit

# 6. PR
> Create a PR with a detailed description.
```

### Context Management During the Day

```
# Between unrelated tasks
/clear

# When context is getting full but you need to continue
/compact Focus on the auth changes

# Quick side question
/btw What's the syntax for TypeScript mapped types again?

# Name important sessions for later
/rename oauth-migration
```

### End of Day

```
# Save your session state
/rename feature-x-day-2

# Check costs
/cost
```

---

## Sources

- [Claude Code Product Page](https://claude.com/product/claude-code)
- [Claude Code Best Practices](https://code.claude.com/docs/en/best-practices)
- [Claude Code Skills Documentation](https://code.claude.com/docs/en/skills)
- [Claude Code Subagents Documentation](https://code.claude.com/docs/en/sub-agents)
- [Claude Code MCP Documentation](https://code.claude.com/docs/en/mcp)
- [Claude Code Cheat Sheet](https://computingforgeeks.com/claude-code-cheat-sheet/)
- [CLAUDE.md Best Practices - UX Planet](https://uxplanet.org/claude-md-best-practices-1ef4f861ce7c)
- [Claude Code Best Practice - GitHub](https://github.com/shanraisshan/claude-code-best-practice)
