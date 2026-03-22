# GitHub Slash Commands

## Command Reference

| Command | Description | Example |
|---------|-------------|---------|
| `/gh-review` | Review a PR | `/gh-review Review PR #42` |
| `/gh-triage` | Triage an issue | `/gh-triage Triage issue #99` |
| `/gh-implement` | Implement an issue | `/gh-implement Implement #55` |
| `/gh-release` | Create a release | `/gh-release Prepare v2.1.0` |
| `/gh-fix-ci` | Fix failing CI | `/gh-fix-ci Fix the failing tests in run 12345` |
| `/gh-cleanup` | Clean up stale branches | `/gh-cleanup Delete merged branches` |
| `/gh-status` | Repository status overview | `/gh-status` |

---

## `/gh-review`

### Skill Definition

```yaml
---
name: gh-review
description: Review a pull request for quality, security, and correctness
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
---
```

### Usage

```
# Review a specific PR
/gh-review Review PR #42

# Review with specific focus
/gh-review Review #42 focusing on security implications

# Review the current branch's PR
/gh-review Review my current PR
```

### What Happens

1. Fetches PR diff and metadata with `gh pr view` and `gh pr diff`
2. Reads the full context of changed files
3. Analyzes changes against project conventions
4. Posts inline review comments on GitHub
5. Provides overall summary with approve/request-changes

---

## `/gh-triage`

### Usage

```
# Triage a specific issue
/gh-triage Triage issue #99

# Batch triage
/gh-triage Triage all unlabeled issues

# Triage with context
/gh-triage Triage #99 - this might be related to the auth refactor
```

---

## `/gh-implement`

### Usage

```
# Implement an issue
/gh-implement Implement issue #55

# Implement with guidance
/gh-implement Implement #55 using the existing auth middleware pattern

# Implement with constraints
/gh-implement Implement #55 but don't change the database schema
```

### What Happens

1. Reads the issue description and comments
2. Analyzes the codebase for relevant context
3. Creates a feature branch: `feat/issue-55-short-desc`
4. Implements the solution
5. Writes tests
6. Runs the test suite
7. Creates a PR linked to the issue

---

## `/gh-release`

### Usage

```
# Auto-detect version bump
/gh-release Create a new release

# Specific version
/gh-release Prepare v2.1.0

# Pre-release
/gh-release Create release candidate for v3.0.0

# Hotfix release
/gh-release Hotfix release for the auth bug fix
```

---

## `/gh-fix-ci`

### Usage

```
# Fix the latest failing run
/gh-fix-ci Fix the failing CI

# Fix a specific run
/gh-fix-ci Fix run 12345678

# Diagnose without fixing
/gh-fix-ci Why is CI failing on PR #42?
```

### What Happens

1. Fetches failed run logs with `gh run view --log-failed`
2. Identifies the root cause
3. Categorizes the failure type
4. Attempts to fix (test code, config, dependencies)
5. Pushes fix and verifies CI passes

---

## `/gh-cleanup`

### Usage

```
# Delete merged branches
/gh-cleanup Delete all merged branches

# Remove stale branches
/gh-cleanup Remove branches with no commits in 60 days

# Audit branches
/gh-cleanup List branches that should be cleaned up
```

---

## `/gh-status`

### Skill Definition

```yaml
---
name: gh-status
description: Quick overview of repository status - open PRs, issues, CI health
user-invocable: true
allowed-tools:
  - Bash
---
```

### Usage

```
/gh-status
```

### Output Example

```
## Repository Status

### Open Pull Requests (3)
| # | Title | Author | Status | Age |
|---|-------|--------|--------|-----|
| 42 | Add auth flow | @alice | Review needed | 2d |
| 41 | Fix memory leak | @bob | CI failing | 3d |
| 39 | Update deps | @carol | Approved | 1d |

### CI Health
- Last 10 runs: 8 passed, 2 failed (80% pass rate)
- Average duration: 4m 32s
- Failing: `test-integration` job (flaky)

### Issues
- Open: 15 (3 P0, 5 P1, 4 P2, 3 P3)
- Unassigned: 4
- Stale (>30 days): 2

### Activity (last 7 days)
- 12 commits to main
- 5 PRs merged
- 8 issues opened, 6 closed
```

---

## Creating Custom GitHub Commands

```bash
mkdir -p .claude/skills/gh-custom
```

`.claude/skills/gh-custom/SKILL.md`:

```yaml
---
name: gh-custom
description: Your custom GitHub command
user-invocable: true
allowed-tools:
  - Bash
  - Read
---
```

```markdown
# Custom GitHub Command

Your instructions here. Use `gh` CLI for all GitHub operations.
```
