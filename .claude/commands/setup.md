---
description: Bootstrap a new project with the vibe coding/ops framework — creates directories, CLAUDE.md, Prime Directive, skills, agents, and logging infrastructure
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
model: opus
argument-hint: "[project-path or . for current directory]"
---

# Vibe Coding/Ops Framework Bootstrap

You are setting up the complete vibe coding and vibe ops framework for a project. This creates all necessary directories, configuration files, and governance infrastructure.

## Target

The target project directory is: `$ARGUMENTS`

If `$ARGUMENTS` is empty, `.`, or not provided, use the **current working directory**.

## Step 1: Validate the Target

1. Confirm the target directory exists. If it does not, create it.
2. Check if a git repository exists. If not, run `git init`.
3. Check if a `.claude/` directory already exists. If so, warn the user: "Existing .claude/ directory detected. Proceeding will add files but will NOT overwrite existing ones. Continue?" — then proceed.

## Step 2: Create Directory Structure

Create the following directories (skip any that already exist):

```
.claude/
  commands/
  skills/
  agents/
  rules/
claudefiles/
  decisions/
  skills/
  agents/
  archive/
learning/
  sessions/
  frameworks/
  guidance/
  archive/
```

## Step 3: Create CLAUDE.md

If `CLAUDE.md` does not exist at the project root, create it with a starter template that includes:
- Placeholder sections for Tech Stack, Common Commands, Code Style, Architecture, Testing, Git Conventions, and Gotchas
- An `@new_prime_directive.md` import reference
- A note telling the user to customize each section

If `.claude/CLAUDE.md` does not exist, create it with the Prime Directive always-active rules (reference `new_prime_directive.md`, session logging, lightweight mode, decision transparency, prompt coaching, changelog, context maintenance, skill extraction, mermaid diagrams, error handling, immutability).

## Step 4: Create the Prime Directive

If `new_prime_directive.md` does not exist at the project root, create it with the full Prime Directive v2.0 including:
- Definitions (session, non-trivial decision, meaningful change)
- Section 1: Session Logging (prompt learning loop, decision/workflow transparency, skill/agent evolution, immutable changelog, project context, file growth management)
- Section 2: Update Cadence
- Section 3: Visualization (mermaid diagrams)
- Section 4: Supporting Documentation
- Section 5: Commit Prefix Convention
- Section 6: Error Handling
- Section 7: Immutability

## Step 5: Create Core Logging Files

Create these files if they do not exist (with headers only):

- `claudefiles/changelog.md` — Header: "# Changelog" with format instructions
- `claudefiles/context.md` — Header: "# Project Context" with section stubs (What, Where we are, Where we're going, Last session)
- `learning/guidance/prompt_guidance.md` — Header: "# Prompt Guidance" with intro
- `learning/frameworks/prompt_framework.md` — Header: "# Prompt Frameworks" with intro
- `claudefiles/skills/skills_master.md` — Header: "# Skills Master"
- `claudefiles/agents/agents_master.md` — Header: "# Agents Master"

## Step 6: Create Starter Slash Commands

Copy these command files into `.claude/commands/` if they do not already exist:
- `prime-directive.md` — The Prime Directive activation command
- `setup.md` — This setup command itself (for re-bootstrapping)

## Step 7: Create .gitignore Entries

Append to `.gitignore` (if entries are not already present):
```
# Claude Code local files
.claude/settings.local.json
.claude/agent-memory-local/
```

## Step 8: Create MCP Config Stub

If `.mcp.json` does not exist, create a stub with commented examples for common MCP servers (GitHub, Playwright, filesystem).

## Step 9: Log the Bootstrap

1. Create a session log entry in `learning/sessions/{today}_session.md`
2. Append a changelog entry to `claudefiles/changelog.md` recording the bootstrap
3. Update `claudefiles/context.md` with the initial project state

## Step 10: Report

Print a summary of everything that was created:
- Number of directories created
- Number of files created
- List of slash commands available
- Next steps for the user (customize CLAUDE.md, set up MCP servers, run `/prime-directive` to activate governance)

## Important Rules

- **Never overwrite existing files.** Always check first. If a file exists, skip it and note that it was skipped.
- **Use ISO 8601 timestamps** for all log entries.
- **Create parent directories** as needed (use `mkdir -p`).
- After completion, tell the user: "Framework bootstrapped. Run `/prime-directive` to activate governance for this session."
