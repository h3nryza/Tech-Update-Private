# Skills Master Index

> Canonical reference of all skills documented across the Claude Framework. Updated each session.

Last updated: 2026-03-23

---

## Summary

| Category | Count |
|----------|-------|
| Core (Most Used) | 5 |
| AWS | 6 |
| GitHub | 5 |
| Datadog | 5 |
| Atlassian | 4 |
| Slack | 4 |
| Cloudflare | 5 |
| ChatOps / AI SRE | 14 |
| Multi-Tool Integration | 7 |
| Testing | 11 |
| Documentation | 10 |
| **Total** | **76** |

---

## Core Skills (Most Used)

| Skill Name | Category | Description | Source File |
|------------|----------|-------------|------------|
| Commit / Conventional Commits | Git | Formats commit messages using Conventional Commits spec with hooks and validation | [Most_used/skills/commit_skill.md](../../Most_used/skills/commit_skill.md) |
| PR Review | Code Quality | Reviews pull requests with 5-dimension scoring (correctness, security, perf, maintainability, testing) | [Most_used/skills/review_pr_skill.md](../../Most_used/skills/review_pr_skill.md) |
| Test Runner / TDD | Testing | Runs tests and drives RED-GREEN-REFACTOR TDD cycles | [Most_used/skills/test_runner_skill.md](../../Most_used/skills/test_runner_skill.md) |
| Refactor | Code Quality | Pattern catalog with safety verification for systematic refactoring | [Most_used/skills/refactor_skill.md](../../Most_used/skills/refactor_skill.md) |
| Deploy | DevOps | Multi-platform deploy orchestration with rollback procedures | [Most_used/skills/deploy_skill.md](../../Most_used/skills/deploy_skill.md) |

---

## AWS Skills

| Skill Name | Category | Description | Source File |
|------------|----------|-------------|------------|
| aws-deploy | Deploy | Deploy infrastructure and applications using CDK, CloudFormation, or SAM | [aws/skills.md](../../aws/skills.md) |
| aws-monitor | Monitoring | Query CloudWatch metrics/logs, analyze alarms and health | [aws/skills.md](../../aws/skills.md) |
| aws-scale | Scaling | Auto-scaling actions for AWS resources (ASG, ECS, Lambda) | [aws/skills.md](../../aws/skills.md) |
| aws-debug | Debugging | Debug AWS resource issues using logs, metrics, and traces | [aws/skills.md](../../aws/skills.md) |
| aws-security-audit | Security | Audit AWS resources for security anti-patterns and compliance | [aws/skills.md](../../aws/skills.md) |
| aws-cost-analysis | Cost | Analyze AWS costs, identify waste, and recommend optimizations | [aws/skills.md](../../aws/skills.md) |

---

## GitHub Skills

| Skill Name | Category | Description | Source File |
|------------|----------|-------------|------------|
| gh-pr-review | Code Quality | Review pull requests for bugs, security, performance, and code quality | [github/skills.md](../../github/skills.md) |
| gh-issue-triage | Project Management | Triage and categorize GitHub issues with labels and assignments | [github/skills.md](../../github/skills.md) |
| gh-cicd | CI/CD | Manage GitHub Actions workflows, debug failures, optimize pipelines | [github/skills.md](../../github/skills.md) |
| gh-release | Release | Manage releases with changelogs, tags, and semantic versioning | [github/skills.md](../../github/skills.md) |
| gh-branch | Git | Branch management with naming conventions and cleanup | [github/skills.md](../../github/skills.md) |

---

## Datadog Skills

| Skill Name | Category | Description | Source File |
|------------|----------|-------------|------------|
| dd-health | Monitoring | Check overall system health from Datadog monitors, alerts, service status | [datadog/skills.md](../../datadog/skills.md) |
| dd-logs | Investigation | Search and analyze Datadog logs for debugging and investigation | [datadog/skills.md](../../datadog/skills.md) |
| dd-metrics | Monitoring | Query and analyze Datadog metrics with anomaly detection | [datadog/skills.md](../../datadog/skills.md) |
| dd-dashboard | Visualization | Create and manage Datadog dashboards programmatically | [datadog/skills.md](../../datadog/skills.md) |
| dd-monitors | Alerting | Create, update, and manage Datadog monitors and alert rules | [datadog/skills.md](../../datadog/skills.md) |

---

## Atlassian Skills

| Skill Name | Category | Description | Source File |
|------------|----------|-------------|------------|
| jira-issues | Project Management | Create, update, search, and manage Jira issues with JQL | [atlassian/skills.md](../../atlassian/skills.md) |
| jira-sprint | Project Management | Sprint management including planning, tracking, and reporting | [atlassian/skills.md](../../atlassian/skills.md) |
| confluence-kb | Documentation | Create and manage Confluence knowledge base pages | [atlassian/skills.md](../../atlassian/skills.md) |
| jira-github-sync | Integration | Sync Jira issues with GitHub PRs, branches, and commits | [atlassian/skills.md](../../atlassian/skills.md) |

---

## Slack Skills

| Skill Name | Category | Description | Source File |
|------------|----------|-------------|------------|
| slack-notify | Notifications | Send structured notifications to Slack channels (deploys, alerts, reports) | [slack/skills.md](../../slack/skills.md) |
| slack-incident | Incident Response | Create and manage incident channels with structured workflows | [slack/skills.md](../../slack/skills.md) |
| slack-thread-summary | Communication | Summarize long Slack threads into concise digests | [slack/skills.md](../../slack/skills.md) |
| slack-standup | Communication | Automated standup collection and summary in Slack | [slack/skills.md](../../slack/skills.md) |

---

## Cloudflare Skills

| Skill Name | Category | Description | Source File |
|------------|----------|-------------|------------|
| cf-workers | Edge Computing | Create, deploy, and manage Cloudflare Workers | [cloudflare/skills.md](../../cloudflare/skills.md) |
| cf-dns | DNS | Manage DNS records, zones, and configurations | [cloudflare/skills.md](../../cloudflare/skills.md) |
| cf-security | Security | Configure WAF rules, rate limiting, bot management, and SSL | [cloudflare/skills.md](../../cloudflare/skills.md) |
| cf-cache | Performance | Manage cache rules, purge strategies, and cache optimization | [cloudflare/skills.md](../../cloudflare/skills.md) |
| cf-pages | Deploy | Deploy static sites and JAMstack apps to Cloudflare Pages | [cloudflare/skills.md](../../cloudflare/skills.md) |

---

## ChatOps / AI SRE Skills

| Skill Name | Category | Description | Source File |
|------------|----------|-------------|------------|
| sre-incident | Incident Response | Full incident response lifecycle across all platforms | [chatops/skills.md](../../chatops/skills.md) |
| sre-triage | Incident Response | Severity assessment and initial categorization of incidents | [chatops/skills.md](../../chatops/skills.md) |
| sre-investigate | Investigation | Root cause analysis with multi-signal correlation | [chatops/skills.md](../../chatops/skills.md) |
| sre-correlate | Investigation | Alert correlation across monitoring platforms | [chatops/skills.md](../../chatops/skills.md) |
| sre-rollback | Remediation | Safe deployment rollback with verification | [chatops/skills.md](../../chatops/skills.md) |
| sre-scale | Remediation | Auto-scaling actions for capacity management | [chatops/skills.md](../../chatops/skills.md) |
| sre-failover | Remediation | Failover orchestration across regions and services | [chatops/skills.md](../../chatops/skills.md) |
| sre-postmortem | Documentation | Post-mortem generation with timeline and action items | [chatops/skills.md](../../chatops/skills.md) |
| sre-monitor | Monitoring | Cross-platform health monitoring dashboard | [chatops/skills.md](../../chatops/skills.md) |
| sre-slo | Monitoring | SLO tracking and reporting with burn-rate alerts | [chatops/skills.md](../../chatops/skills.md) |
| sre-timeline | Documentation | Incident timeline builder with event correlation | [chatops/skills.md](../../chatops/skills.md) |
| sre-logdive | Investigation | Deep log analysis with pattern detection | [chatops/skills.md](../../chatops/skills.md) |
| sre-handoff | Operations | On-call handoff with context transfer | [chatops/skills.md](../../chatops/skills.md) |
| sre-cost | Cost | Cost anomaly detection and alerting | [chatops/skills.md](../../chatops/skills.md) |

---

## Multi-Tool Integration Skills

| Skill Name | Category | Description | Source File |
|------------|----------|-------------|------------|
| Gemini Second Opinion | Code Review | Get a free second opinion from Gemini CLI on current changes | [working_nicely/example_skills.md](../../working_nicely/example_skills.md) |
| Multi-Model Analysis | Code Review | Analyze code using multiple AI models for higher confidence | [working_nicely/example_skills.md](../../working_nicely/example_skills.md) |
| Gemini Research | Research | Use Gemini CLI with Google Search grounding for research | [working_nicely/example_skills.md](../../working_nicely/example_skills.md) |
| Codex Auto-Implement | Implementation | Hand off implementation tasks to Codex CLI in full-auto mode | [working_nicely/example_skills.md](../../working_nicely/example_skills.md) |
| Cross-Tool Test Suite | Testing | Run tests across multiple AI tools for coverage | [working_nicely/example_skills.md](../../working_nicely/example_skills.md) |
| Memorix Context Loader | Context | Load and share context across AI tool sessions | [working_nicely/example_skills.md](../../working_nicely/example_skills.md) |
| Local Model Fallback | Resilience | Fall back to local models when API tools are unavailable | [working_nicely/example_skills.md](../../working_nicely/example_skills.md) |

---

## Testing Skills

| Skill Name | Category | Description | Source File |
|------------|----------|-------------|------------|
| Quality Audit | Quality | Audit codebase against quality metrics and standards | [testing/quality_metrics.md](../../testing/quality_metrics.md) |
| Quality Gate Setup | Quality | Set up automated quality gates in CI/CD pipelines | [testing/quality_metrics.md](../../testing/quality_metrics.md) |
| Metrics Dashboard Generator | Quality | Generate quality metrics dashboards from test data | [testing/quality_metrics.md](../../testing/quality_metrics.md) |
| Security Audit | Security | SAST/DAST scanning, dependency checks, secret detection | [testing/fuzzing_security.md](../../testing/fuzzing_security.md) |
| Fuzz Test Generator | Testing | Generate fuzz tests for APIs and parsers | [testing/fuzzing_security.md](../../testing/fuzzing_security.md) |
| Dependency Security Check | Security | Scan dependencies for known vulnerabilities | [testing/fuzzing_security.md](../../testing/fuzzing_security.md) |
| Test Strategy Recommender | Testing | Recommend test strategies based on codebase analysis | [testing/test_strategies.md](../../testing/test_strategies.md) |
| Regression Guard | Testing | Guard against regressions with targeted test generation | [testing/test_strategies.md](../../testing/test_strategies.md) |
| AI Code Audit | Quality | Audit AI-generated code for common issues | [testing/test_strategies.md](../../testing/test_strategies.md) |
| Test Review Agent | Testing | Review test quality and coverage gaps | [testing/test_generation.md](../../testing/test_generation.md) |
| TDD Workflow | Testing | Guided TDD workflow with AI assistance | [testing/test_generation.md](../../testing/test_generation.md) |

---

## Documentation Skills

| Skill Name | Category | Description | Source File |
|------------|----------|-------------|------------|
| api_doc_writer | Documentation | Generate OpenAPI specs and API documentation from code | [documentation/api_docs.md](../../documentation/api_docs.md) |
| sdk_doc_writer | Documentation | Generate SDK documentation with examples | [documentation/api_docs.md](../../documentation/api_docs.md) |
| architecture_mapper | Documentation | Generate C4 model architecture diagrams from code | [documentation/architecture_docs.md](../../documentation/architecture_docs.md) |
| adr_writer | Documentation | Write Architecture Decision Records | [documentation/architecture_docs.md](../../documentation/architecture_docs.md) |
| tech_debt_tracker | Documentation | Track and document technical debt | [documentation/architecture_docs.md](../../documentation/architecture_docs.md) |
| onboarding_builder | Documentation | Generate getting-started guides for new developers | [documentation/onboarding_docs.md](../../documentation/onboarding_docs.md) |
| doc_freshness_keeper | Documentation | Monitor and update stale documentation | [documentation/onboarding_docs.md](../../documentation/onboarding_docs.md) |
| diagram_creator | Documentation | Generate mermaid diagrams from code structure | [documentation/diagram_generation.md](../../documentation/diagram_generation.md) |
| changelog_writer | Documentation | Generate changelogs from commit history | [documentation/changelog_automation.md](../../documentation/changelog_automation.md) |
| commit_enforcer | Documentation | Enforce Conventional Commits format | [documentation/changelog_automation.md](../../documentation/changelog_automation.md) |

---

## How to Use This Index

1. **Find a skill** by scanning the tables above or searching by category
2. **Read the source file** linked in the last column for full implementation details
3. **Copy the skill** into your project's `.claude/skills/` directory
4. See [How_to_use.md](../../How_to_use.md) for detailed installation instructions

---

*Part of the [Claude Framework](../../README.md). See [index.md](../../index.md) for the full file listing.*
