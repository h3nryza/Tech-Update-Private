---
description: Common vibe ops tasks — deploy checks, infrastructure review, monitoring setup, dependency audits, and more
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
argument-hint: "[task: deploy-check | infra-review | deps-audit | health-check | monitor-setup | scale-plan | cleanup | status]"
---

# Vibe Ops Task Runner

You are running a vibe ops task. Parse `$ARGUMENTS` to determine which task to execute.

If `$ARGUMENTS` is empty, print the task menu and ask the user to choose:

```
Available Vibe Ops Tasks:
  /vibeops deploy-check    — Pre-deployment readiness check
  /vibeops infra-review    — Review infrastructure configuration
  /vibeops deps-audit      — Audit dependencies for updates and vulnerabilities
  /vibeops health-check    — Check project health (tests, lint, types, build)
  /vibeops monitor-setup   — Set up or review monitoring configuration
  /vibeops scale-plan      — Generate a scaling plan for current architecture
  /vibeops cleanup         — Clean up stale branches, old artifacts, unused deps
  /vibeops status          — Full project status dashboard
```

---

## Task: deploy-check

Pre-deployment readiness assessment:

1. **Git status**: Check for uncommitted changes, verify branch
2. **Tests**: Run the test suite and report pass/fail counts
3. **Type checking**: Run typecheck if TypeScript project
4. **Lint**: Run linter and report issues
5. **Build**: Attempt a production build
6. **Dependencies**: Check for known vulnerabilities (`npm audit` / `pip audit` / equivalent)
7. **Environment**: Verify required environment variables are documented
8. **Recent changes**: Summarize what changed since last tag/release

Report as a checklist:
```
Deploy Readiness Check:
[PASS] Git status clean
[PASS] All tests passing (142/142)
[WARN] 2 lint warnings (non-blocking)
[PASS] Build succeeds
[FAIL] npm audit found 1 high severity vulnerability
[PASS] Environment variables documented

Verdict: NOT READY — fix npm audit finding before deploying
```

---

## Task: infra-review

Review infrastructure configuration files:

1. **Scan for config files**: Look for Dockerfile, docker-compose.yml, Terraform files, CloudFormation, Kubernetes manifests, serverless.yml, Procfile, fly.toml, vercel.json, netlify.toml, railway.json, render.yaml, etc.
2. **For each found**:
   - Check for security best practices (no hardcoded secrets, minimal permissions)
   - Check for production readiness (health checks, resource limits, restart policies)
   - Check for cost optimization opportunities
   - Check for missing configurations (logging, monitoring, backups)
3. **Report findings** by category: Security, Reliability, Cost, Missing

---

## Task: deps-audit

Comprehensive dependency audit:

1. **Detect package manager**: npm, yarn, pnpm, pip, poetry, cargo, go modules, etc.
2. **Check for vulnerabilities**: Run the appropriate audit command
3. **Check for outdated packages**: List packages with available updates
4. **Categorize updates**:
   - Security fixes (must update)
   - Major versions (breaking changes, plan carefully)
   - Minor/patch versions (safe to update)
5. **Check for unused dependencies**: Identify packages that appear unused
6. **Report** with actionable recommendations

---

## Task: health-check

Full project health assessment:

1. **Repository health**:
   - Git status (clean/dirty)
   - Branch count and stale branches
   - Last commit date
   - Open PRs count (if GitHub MCP available)
2. **Code health**:
   - Test pass rate
   - Lint issue count
   - Type error count (if applicable)
   - Build status
3. **Documentation health**:
   - README exists and is non-empty
   - CLAUDE.md exists
   - Key docs are up to date (check modification dates)
4. **Dependency health**:
   - Vulnerability count
   - Outdated package count
5. **Framework health** (if using vibe coding framework):
   - Check `claudefiles/context.md` is recent
   - Check `claudefiles/changelog.md` has recent entries
   - Check `learning/sessions/` has recent session logs

Print a health score card:
```
Project Health: 85/100

Repository:    [====------] Good
Code Quality:  [========--] Excellent
Documentation: [======----] Good
Dependencies:  [====------] Needs Attention
Framework:     [==========] Excellent
```

---

## Task: monitor-setup

Guide monitoring configuration:

1. **Assess current monitoring**: Search for existing monitoring config (Sentry, Datadog, New Relic, Prometheus, etc.)
2. **Identify gaps**: What is not being monitored?
3. **Recommend setup** for:
   - Error tracking (uncaught exceptions, failed requests)
   - Performance monitoring (response times, throughput)
   - Uptime monitoring (health check endpoints)
   - Log aggregation (structured logging)
   - Alerting (thresholds and notification channels)
4. **Generate configuration** for the recommended tools
5. **Create runbook stubs** for common alert scenarios

---

## Task: scale-plan

Generate a scaling assessment:

1. **Current architecture**: Map the system components from code and config files
2. **Identify bottlenecks**: Database connections, single points of failure, stateful components
3. **Scaling recommendations**:
   - Horizontal scaling opportunities (stateless services)
   - Vertical scaling needs (memory/CPU bound components)
   - Caching opportunities (read-heavy endpoints, computed values)
   - Database optimization (indexes, connection pooling, read replicas)
   - CDN opportunities (static assets, API caching)
4. **Cost estimates**: Rough cost implications of scaling recommendations
5. **Priority order**: What to scale first based on likely bottlenecks

Output as a mermaid architecture diagram plus recommendations table.

---

## Task: cleanup

Project cleanup tasks:

1. **Git cleanup**:
   - List branches merged into main that can be deleted
   - List branches older than 90 days with no activity
   - Offer to delete stale branches (with confirmation)
2. **Dependency cleanup**:
   - Identify unused dependencies
   - Identify duplicate packages
   - Offer to remove (with confirmation)
3. **File cleanup**:
   - Find large files that might not belong in git (>1MB)
   - Find generated files that should be git-ignored
   - Find orphaned test files (tests for deleted source files)
4. **Framework cleanup** (if using vibe coding framework):
   - Archive session logs older than 90 days
   - Archive decision files older than 90 days
   - Consolidate guidance/framework files if over 300 lines
5. **Report** everything found and ask before making any changes

---

## Task: status

Full project status dashboard:

1. **Git**: Current branch, uncommitted changes, ahead/behind remote
2. **Recent activity**: Last 5 commits with authors and dates
3. **Open work**: Open PRs and issues (if GitHub MCP available)
4. **Framework state**: Latest session log, last context update, recent decisions
5. **Health summary**: Quick pass/fail for tests, lint, build
6. **Next steps**: Based on context.md and recent activity, suggest what to work on next

Format as a clean dashboard:
```
=== Project Status Dashboard ===
Branch: feature/auth-refactor (3 ahead, 1 behind main)
Last commit: 2h ago — "feat: add JWT refresh token support"

Recent Activity:
  [+] feat: add JWT refresh token support (2h ago)
  [+] test: add auth middleware tests (3h ago)
  [+] refactor: extract token validation (yesterday)

Open Work:
  PR #42: Auth refactor (draft, 3 files changed)
  Issue #38: Token expiry not handled gracefully

Health: Tests PASS | Lint PASS | Build PASS | Deps 1 WARNING

Framework:
  Last session: 2026-03-22
  Context: Updated
  Pending: Update skills master

Suggested Next Steps:
  1. Complete PR #42 (auth refactor)
  2. Address Issue #38 (related to current work)
  3. Update dependency with known vulnerability
```

---

## Rules

- **Always report, then act.** Show the user what you found before making changes.
- **Ask before destructive operations.** Never delete branches, remove dependencies, or modify config without explicit confirmation.
- **Log significant findings** to `claudefiles/changelog.md` if the Prime Directive is active.
- **Use the project's actual tools.** Detect the package manager, test runner, linter, etc. from project config — do not assume npm/jest/eslint.
