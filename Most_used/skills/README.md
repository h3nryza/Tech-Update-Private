# Most Popular Claude Code Skills

> A curated overview of the most commonly used skills in Claude Code and similar AI coding frameworks, based on community adoption and ecosystem research.

---

## What Are Skills?

Skills are reusable instruction sets that extend Claude Code's capabilities. Each skill lives in a `.claude/skills/<skill-name>/SKILL.md` file containing YAML frontmatter (metadata) and markdown instructions that Claude follows when the skill is invoked.

Skills replaced the older `.claude/commands/` system. Existing commands still work, but new ones should use the skills format for features like frontmatter control, supporting files, and model-invocation settings.

## Skill File Structure

```
.claude/skills/
  my-skill/
    SKILL.md          # Required: frontmatter + instructions
    supporting.md     # Optional: additional context files
    templates/        # Optional: template files referenced by instructions
```

### SKILL.md Template

```yaml
---
name: my-skill-name
description: >
  Clear description of what this skill does.
  Used by Claude to decide when to activate.
disable-model-invocation: false   # true = user-only trigger (e.g., deploy)
---

# My Skill Name

## When to Use
- Trigger conditions

## Instructions
1. Step-by-step execution guide

## Examples
- Input/output examples
```

---

## Top 10 Most Popular Skills (Ranked by Community Adoption)

| Rank | Skill | Category | Description |
|------|-------|----------|-------------|
| 1 | **Commit / Conventional Commits** | Git | Formats commit messages using Conventional Commits spec |
| 2 | **PR Review** | Code Quality | Reviews pull requests with structured feedback |
| 3 | **Test Runner / TDD** | Testing | Runs tests and drives test-driven development cycles |
| 4 | **Refactor** | Code Quality | Systematic code refactoring with safety checks |
| 5 | **Deploy** | DevOps | Orchestrates build, test, and deployment pipelines |
| 6 | **Explain Code** | Education | Explains code with diagrams and analogies |
| 7 | **Security Audit** | Security | Scans for vulnerabilities and insecure patterns |
| 8 | **Documentation Generator** | Docs | Generates API docs, READMEs, and inline comments |
| 9 | **Database Migration** | Data | Creates and validates database migration scripts |
| 10 | **Performance Optimizer** | Performance | Profiles and suggests performance improvements |

---

## Skill Ecosystem Resources

### Official

- [Extend Claude with Skills](https://code.claude.com/docs/en/skills) -- Official documentation
- [anthropics/skills](https://github.com/anthropics/skills) -- Anthropic's public skill repository
- [Skill Authoring Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)

### Community

- [awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) -- Curated list of skills, hooks, slash commands, and plugins
- [awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills) -- Community skills collection
- [awesome-agent-skills](https://github.com/VoltAgent/awesome-agent-skills) -- 500+ agent skills compatible with Claude Code and other tools
- [awesome-claude-code-toolkit](https://github.com/rohitg00/awesome-claude-code-toolkit) -- 135 agents, 35 skills, 42 commands, 150+ plugins
- [claude-code-skill-factory](https://github.com/alirezarezvani/claude-code-skill-factory) -- Toolkit for building production-ready skills

### Skill Libraries

- [obra/superpowers](https://github.com/obra/superpowers) -- 20+ battle-tested skills including TDD, debugging, and collaboration
- [antigravity-awesome-skills](https://github.com/sickn33/antigravity-awesome-skills) -- 1,300+ installable agentic skills with CLI installer

---

## Detailed Skill Guides in This Directory

| File | Skill |
|------|-------|
| [commit_skill.md](commit_skill.md) | Auto-commit with Conventional Commits |
| [review_pr_skill.md](review_pr_skill.md) | Pull request review |
| [test_runner_skill.md](test_runner_skill.md) | Test runner and TDD |
| [refactor_skill.md](refactor_skill.md) | Code refactoring |
| [deploy_skill.md](deploy_skill.md) | Deployment orchestration |

---

## Quick Install Pattern

To add any skill to your project:

```bash
mkdir -p .claude/skills/my-skill
cat > .claude/skills/my-skill/SKILL.md << 'EOF'
---
name: my-skill
description: Description here
---
# Instructions here
EOF
```

Or use an installer like [antigravity-awesome-skills](https://github.com/sickn33/antigravity-awesome-skills):

```bash
npx @sickn33/awesome-skills install <skill-name>
```
