# Commit Skill -- Auto-Commit with Conventional Commits

> The most widely adopted Claude Code skill. Formats git commit messages using the Conventional Commits specification and automates the staging/commit workflow.

---

## Why This Skill Exists

Without guidance, Claude produces inconsistent commit messages. This skill enforces a standard format, ensures meaningful descriptions, and can optionally run pre-commit checks before committing.

---

## SKILL.md Configuration

```yaml
---
name: commit
description: >
  Formats and creates git commit messages using Conventional Commits.
  Use when the user asks to commit, save changes, or mentions git commit.
---

# Commit Skill

## Conventional Commits Format

Every commit message MUST follow this format:

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### Types (required)

| Type | When to Use |
|------|------------|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, whitespace, semicolons (no logic change) |
| `refactor` | Code restructuring (no feature or fix) |
| `test` | Adding or updating tests |
| `chore` | Build, CI, tooling, dependencies |
| `perf` | Performance improvement |
| `ci` | CI/CD configuration changes |
| `build` | Build system or external dependency changes |
| `revert` | Reverts a previous commit |

### Scope (optional but encouraged)

The component or area affected: `auth`, `api`, `ui`, `db`, `config`, etc.

### Description Rules

- Use imperative mood: "add feature" not "added feature"
- Do not capitalize the first letter
- No period at the end
- Max 72 characters for the subject line
- Body wraps at 80 characters

## Workflow

1. Run `git status` to see all changes
2. Run `git diff --staged` and `git diff` to understand what changed
3. Group related changes logically -- if changes span unrelated areas, suggest multiple commits
4. Determine the correct type and scope from the diff
5. Write a concise description focusing on WHY, not WHAT
6. Stage the appropriate files (prefer specific files over `git add .`)
7. Create the commit

## Pre-Commit Checks

Before committing, verify:
- [ ] No secrets or credentials in staged files (.env, API keys, tokens)
- [ ] No large binary files accidentally staged
- [ ] Changes compile/build successfully (if applicable)
- [ ] Linter passes (if configured)

## Examples

### Simple feature commit
```bash
git add src/auth/oauth.ts
git commit -m "feat(auth): add OAuth2 PKCE flow for SPA clients"
```

### Bug fix with body
```bash
git add src/api/handler.ts src/api/handler.test.ts
git commit -m "$(cat <<'EOF'
fix(api): prevent race condition in concurrent request handler

The mutex was not being released on early return paths, causing
deadlocks under high concurrency. Added defer pattern to ensure
cleanup on all exit paths.

Closes #142
EOF
)"
```

### Breaking change
```bash
git commit -m "$(cat <<'EOF'
feat(api)!: remove deprecated v1 endpoints

BREAKING CHANGE: All /api/v1/* endpoints have been removed.
Clients must migrate to /api/v2/* equivalents.
See migration guide: docs/migration-v2.md
EOF
)"
```

### Multi-file chore
```bash
git add package.json package-lock.json
git commit -m "chore(deps): upgrade express to 5.1.0 for security patch"
```
```

---

## CLAUDE.md Integration

Add this to your project's `CLAUDE.md` to enforce commit standards without installing the full skill:

```markdown
## Git Commits

- Use Conventional Commits format: `type(scope): description`
- Types: feat, fix, docs, style, refactor, test, chore, perf, ci, build, revert
- Imperative mood, no capitalization, no trailing period
- Max 72 chars subject, 80 chars body wrap
- Stage specific files, never `git add .` or `git add -A`
- Check for secrets before committing
- If changes span unrelated areas, create separate commits
```

---

## Hook Integration

Pair this skill with a PostToolUse hook to validate commit messages:

```json
// .claude/settings.json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Bash",
        "command": "echo '$ARGUMENTS' | jq -r '.command' | grep -q '^git commit' && echo '$ARGUMENTS' | jq -r '.command' | grep -qP '^git commit -m \"(feat|fix|docs|style|refactor|test|chore|perf|ci|build|revert)(\\(.*\\))?!?: .+\"' || (echo 'Commit message does not follow Conventional Commits format' >&2; exit 2)"
      }
    ]
  }
}
```

---

## Sources

- [Conventional Commits Specification](https://www.conventionalcommits.org/)
- [Claude Code Skills Documentation](https://code.claude.com/docs/en/skills)
- [wshobson/commands](https://github.com/wshobson/commands) -- Production-ready slash commands collection
