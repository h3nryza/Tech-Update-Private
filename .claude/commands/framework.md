---
description: Explain the vibe coding/ops methodology — philosophy, components, workflows, and how everything fits together
allowed-tools: Read, Glob, Grep
argument-hint: "[topic: overview | prime-directive | claude-md | skills | agents | mcp | workflow | structure | anti-patterns]"
---

# Vibe Coding/Ops Framework Explainer

You are explaining the vibe coding and vibe ops methodology. This is a reference and teaching command — it reads project files and explains concepts clearly.

## Parse Arguments

Parse `$ARGUMENTS` to determine what to explain. If empty, show the topic menu and give the full overview.

Available topics:
- `overview` — What is this framework and why does it exist?
- `prime-directive` — The governance system and session logging
- `claude-md` — The CLAUDE.md configuration system
- `skills` — Custom slash commands and skills
- `agents` — Subagents and agent orchestration
- `mcp` — MCP servers and external tool integration
- `workflow` — The four-phase development workflow
- `structure` — Directory structure and file organization
- `anti-patterns` — Common mistakes and how to avoid them

---

## Topic: overview

Read `methodology.md` if it exists in the project, then explain:

### What is Vibe Coding?

Vibe coding, coined by Andrej Karpathy in February 2025, is a development approach where the programmer shifts from manually writing code to guiding, testing, and giving feedback about AI-generated code. The spectrum runs from "pure vibe coding" (accept everything, no review) to "agentic engineering" (orchestrate AI agents with full oversight).

### What is Vibe Ops?

Vibe Ops extends the philosophy to infrastructure, deployment, monitoring, and maintenance. Instead of writing Terraform or Kubernetes manifests manually, you describe infrastructure needs in natural language and let AI handle implementation.

### What is This Framework?

This framework provides the governance, structure, and tooling to practice vibe coding and vibe ops responsibly. It includes:

1. **Prime Directive** — Immutable rules for session logging, decision transparency, and knowledge capture
2. **CLAUDE.md System** — Persistent context configuration at multiple levels
3. **Custom Commands** — Slash commands for common workflows (`/setup`, `/incident`, `/vibeops`)
4. **Skills & Agents** — Reusable AI capabilities and specialized subagents
5. **MCP Servers** — External tool integration (GitHub, databases, browsers)
6. **Learning Loop** — Every session teaches both the AI and the user to work better together

### The Core Philosophy

The framework exists because vibe coding without governance is prototyping. With governance — session logs, decision docs, skill extraction, prompt coaching — it becomes a sustainable engineering practice.

---

## Topic: prime-directive

Read `new_prime_directive.md` and explain each section:

1. **Session Logging** — Every interaction is documented with raw prompts, interpretations, and timestamps
2. **Decision Transparency** — Every non-trivial decision is documented with alternatives considered
3. **Prompt Coaching** — The AI teaches you to prompt better over time
4. **Skill Extraction** — Reusable patterns are extracted into skills and agents automatically
5. **Immutable Changelog** — Append-only record of all changes
6. **Context Maintenance** — Living document tracking project state across sessions
7. **Commit Prefix Convention** — Optional session-scoped commit prefixes

Explain why each rule exists and how it creates compounding value over time.

---

## Topic: claude-md

Explain the CLAUDE.md configuration system:

1. **File hierarchy**: Personal (`~/.claude/CLAUDE.md`) > Org > Project > Subdirectory
2. **What to include**: Commands Claude cannot guess, code style rules, architectural decisions, gotchas
3. **What to exclude**: Anything Claude can figure out from code, standard conventions, long tutorials
4. **Key principles**: Keep under 200 lines, use pointers not copies, prune ruthlessly, check into git
5. **Imports**: `@path/to/file` syntax for referencing other files
6. **Path-specific rules**: `.claude/rules/*.md` for directory-scoped instructions

Read the project's `.claude/CLAUDE.md` and `CLAUDE.md` (if they exist) and explain what is configured.

---

## Topic: skills

Explain the skills system:

1. **What skills are**: Reusable instructions in `.claude/commands/` or `.claude/skills/` that create slash commands
2. **Project vs personal**: `.claude/commands/` for project, `~/.claude/commands/` for personal
3. **Skills vs commands**: Skills (`.claude/skills/<name>/SKILL.md`) support additional features like supporting files; commands (`.claude/commands/<name>.md`) are the simpler format
4. **Key frontmatter**: name, description, allowed-tools, model, disable-model-invocation, argument-hint
5. **$ARGUMENTS**: Dynamic input passed after the command name
6. **Built-in commands**: /batch, /debug, /simplify, /loop, /claude-api

List all custom commands available in this project by scanning `.claude/commands/` and `.claude/skills/`.

---

## Topic: agents

Explain the subagent system:

1. **What subagents are**: Specialized AI assistants running in their own context window
2. **Why they matter**: They keep exploration out of your main context
3. **Built-in agents**: Explore (fast, read-only), Plan (research), General-purpose
4. **Custom agents**: `.claude/agents/*.md` for project, `~/.claude/agents/*.md` for personal
5. **Key features**: Persistent memory, background execution, isolated worktrees, tool restrictions
6. **Configuration**: name, description, tools, model, memory, isolation

List all custom agents available in this project by scanning `.claude/agents/`.

---

## Topic: mcp

Explain MCP (Model Context Protocol) servers:

1. **What MCP servers do**: Extend Claude with external tool access (GitHub, databases, browsers, etc.)
2. **Configuration scopes**: Local (`~/.claude.json`), Project (`.mcp.json`), User
3. **Popular servers**: GitHub, Playwright, PostgreSQL, Filesystem, Figma, Notion, Slack, Sentry
4. **Setup commands**: `claude mcp add`, `claude mcp add-json`, `claude mcp list`, `claude mcp remove`
5. **The reasoning**: Claude is most effective when it can verify its own work and access real data

Check if `.mcp.json` exists and explain what is configured.

---

## Topic: workflow

Explain the four-phase development workflow:

1. **EXPLORE** (Plan Mode) — Read files, understand code, ask questions
2. **PLAN** (Plan Mode) — Create implementation plan, edit in external editor (Ctrl+G)
3. **IMPLEMENT** (Normal Mode) — Code, test, verify, fix failures
4. **COMMIT** (Normal Mode) — Commit with message, open PR

Also explain:
- When to skip planning (small, obvious changes)
- Context management (/clear, /compact, /rewind)
- The verification loop (test-first, self-validation)
- Session management best practices

---

## Topic: structure

Explain the directory structure and why each directory exists:

```
.claude/                  — Claude Code configuration (check into git)
  commands/               — Custom slash commands
  skills/                 — Skills with supporting files
  agents/                 — Custom subagents
  rules/                  — Path-specific rules
claudefiles/              — AI governance and learning logs
  decisions/              — Decision documentation per session
  skills/                 — Extracted skill definitions
  agents/                 — Extracted agent patterns
  changelog.md            — Immutable append-only change log
  context.md              — Living project context
  archive/                — Archived old files (>90 days)
learning/                 — Session learning and coaching
  sessions/               — Daily session logs
  frameworks/             — Reusable prompt frameworks
  guidance/               — Prompt coaching feedback
  archive/                — Archived old guidance
```

Explain the reasoning: governance files are separate from Claude config, learning files are separate from both, and everything is designed for append-only or date-scoped patterns to preserve history.

---

## Topic: anti-patterns

Explain the common anti-patterns and how to avoid them:

1. **Kitchen Sink Session** — Mixing unrelated tasks in one session. Fix: `/clear` between tasks.
2. **Correction Loop** — Repeated failed corrections polluting context. Fix: `/clear` and write a better initial prompt.
3. **Over-Specified CLAUDE.md** — Too long, Claude ignores half. Fix: Prune ruthlessly, use hooks for mandatory behaviors.
4. **Trust-Then-Verify Gap** — Accepting plausible code without testing. Fix: Always provide verification.
5. **Infinite Exploration** — Unscoped investigation filling context. Fix: Use subagents or scope narrowly.
6. **Never Updating CLAUDE.md** — Same mistakes repeated. Fix: Add a note every time Claude errs ("compounding engineering").

---

## Formatting Rules

- Use mermaid diagrams where they help explain workflows or architecture
- Keep explanations conversational but technically accurate
- Include concrete examples where helpful
- Reference actual project files when they exist
- If the user asked about a specific topic, stay focused on that topic
- If explaining the overview, keep it to a high-level summary with pointers to dig deeper
