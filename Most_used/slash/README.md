# Most Popular Slash Commands in Claude Code

> A comprehensive reference of built-in and custom slash commands, ranked by community usage and utility.

---

## What Are Slash Commands?

Slash commands are text commands prefixed with `/` that trigger specific actions in Claude Code. There are two categories:

1. **Built-in commands**: Hardcoded into the Claude Code CLI (e.g., `/clear`, `/compact`, `/help`)
2. **Custom commands/skills**: User-defined in `.claude/commands/` (legacy) or `.claude/skills/` (current)

The skills system is the recommended approach going forward, but slash commands remain the primary interface for invoking both built-in and custom actions.

---

## Built-in Commands (Ranked by Usage)

| Rank | Command | Description | When to Use |
|------|---------|-------------|-------------|
| 1 | `/compact` | Summarize conversation to reclaim context window | When context fills up in long sessions |
| 2 | `/clear` | Clear conversation history and start fresh | Between unrelated tasks |
| 3 | `/help` | Show available commands and usage info | When unsure what's available |
| 4 | `/diff` | Interactive diff viewer of all changes made | To review what Claude changed |
| 5 | `/model` | Switch between Claude models mid-session | When you need a different model capability |
| 6 | `/cost` | Show token usage and cost for the session | To monitor API spend |
| 7 | `/init` | Generate a CLAUDE.md file by analyzing the codebase | First setup of a new repository |
| 8 | `/hooks` | Interactive hook configuration | Setting up automation rules |
| 9 | `/vim` | Enable vim keybindings for prompt input | For vim users |
| 10 | `/loop` | Run a task in a loop with conditions | Iterative refinement workflows |
| 11 | `/batch` | Process multiple items in batch | Bulk operations across files |
| 12 | `/simplify` | Three-agent parallel code review | Replaced deprecated `/review` |

---

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Shift+Tab` | Toggle between Plan Mode and Normal Mode |
| `Ctrl+C` | Cancel current operation |
| `Ctrl+D` | Exit Claude Code |
| `Up Arrow` | Recall previous prompt |
| `Escape` | Clear current input |

---

## Custom Command Structure (Legacy)

Custom commands live in `.claude/commands/` as markdown files:

```
.claude/commands/
  commit.md           # Invoked as /commit
  review.md           # Invoked as /review
  deploy.md           # Invoked as /deploy
  project/
    setup.md          # Invoked as /project:setup
```

Each file contains plain markdown instructions that Claude follows when the command is invoked. No frontmatter needed (unlike skills).

### Example: `.claude/commands/commit.md`

```markdown
Create a git commit for the current changes.

1. Run `git status` and `git diff --staged` to understand changes
2. Use Conventional Commits format: type(scope): description
3. Stage only related files (never `git add .`)
4. Write a concise message focusing on WHY
5. Verify with `git log --oneline -1` after committing
```

---

## Skills Structure (Current)

Skills live in `.claude/skills/` and use YAML frontmatter:

```
.claude/skills/
  commit/
    SKILL.md           # Invoked as /commit (auto-detected from name)
```

See [skills/README.md](../skills/README.md) for full details on skill creation.

---

## Community Command Collections

| Repository | Commands | Description |
|------------|----------|-------------|
| [wshobson/commands](https://github.com/wshobson/commands) | 57 | 15 workflows + 42 tools, production-ready |
| [obra/superpowers](https://github.com/obra/superpowers) | 20+ | TDD, debugging, collaboration patterns |
| [awesome-claude-code-toolkit](https://github.com/rohitg00/awesome-claude-code-toolkit) | 42 | Part of comprehensive toolkit |

---

## Detailed Guide

See [common_commands.md](common_commands.md) for detailed examples and configurations of the most popular commands.

---

## Sources

- [Claude Code Slash Commands Reference](https://prg.sh/notes/Claude-Code-Slash-Commands)
- [Claude Code Complete Command Reference (2026)](https://smartscope.blog/en/generative-ai/claude/claude-code-reference-guide/)
- [Claude Code Interactive Mode Reference](https://claudefa.st/blog/guide/mechanics/interactive-mode)
- [5 New Claude Code Slash Commands](https://medium.com/@joe.njenga/5-new-claude-code-slash-commands-that-are-making-workflows-better-7bd416a5859a)
