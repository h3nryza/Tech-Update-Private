# Agents Master Index

> Canonical reference of all agent patterns documented across the Claude Framework. Updated each session.

Last updated: 2026-03-23

---

## Summary

| Category | Count |
|----------|-------|
| Core (Most Used) | 4 |
| AWS | 4 |
| GitHub | 4 |
| Datadog | 3 |
| Atlassian | 3 |
| Slack | 3 |
| Cloudflare | 3 |
| ChatOps / AI SRE | 6 |
| Multi-Tool Integration | 6 |
| Advanced (Orchestration Patterns) | 4 |
| Agent Framework Patterns | 6 |
| Advanced Specialist Agents | 3 |
| **Total** | **49** |

---

## Core Agents (Most Used)

| Agent Name | Category | Description | Source File |
|------------|----------|-------------|------------|
| Code Reviewer | Code Quality | Multi-dimensional weighted code review with scoring across correctness, security, performance, maintainability | [Most_used/agents/code_reviewer.md](../../Most_used/agents/code_reviewer.md) |
| Test Writer | Testing | Test generation across frameworks with edge case coverage and TDD support | [Most_used/agents/test_writer.md](../../Most_used/agents/test_writer.md) |
| Bug Fixer | Debugging | 7-phase systematic debugging: reproduce, isolate, root cause, fix, verify, regression test, document | [Most_used/agents/bug_fixer.md](../../Most_used/agents/bug_fixer.md) |
| Documentation Agent | Documentation | Inline docs, API docs, README generation, and documentation maintenance | [Most_used/agents/documentation.md](../../Most_used/agents/documentation.md) |

---

## AWS Agents

| Agent Name | Category | Description | Source File |
|------------|----------|-------------|------------|
| Infrastructure Agent | Infrastructure | Full lifecycle AWS infrastructure management -- design, deploy, monitor, optimize | [aws/agents.md](../../aws/agents.md) |
| Cost Agent | Cost Optimization | Autonomous AWS cost analysis, waste identification, and optimization recommendations | [aws/agents.md](../../aws/agents.md) |
| Security Agent | Security | AWS security posture assessment, compliance checking, and remediation | [aws/agents.md](../../aws/agents.md) |
| Deployment Agent | DevOps | Orchestrates AWS deployments with blue/green, canary, and rolling strategies | [aws/agents.md](../../aws/agents.md) |

---

## GitHub Agents

| Agent Name | Category | Description | Source File |
|------------|----------|-------------|------------|
| Code Review Agent | Code Quality | Autonomous PR review with consistent quality standards and inline suggestions | [github/agents.md](../../github/agents.md) |
| Issue Implementer Agent | Development | Takes a GitHub issue and autonomously implements it (code, tests, PR) | [github/agents.md](../../github/agents.md) |
| CI/CD Doctor Agent | CI/CD | Diagnoses and fixes GitHub Actions failures, optimizes pipeline performance | [github/agents.md](../../github/agents.md) |
| Release Agent | Release | Manages release lifecycle: changelog, version bump, tag, publish | [github/agents.md](../../github/agents.md) |

---

## Datadog Agents

| Agent Name | Category | Description | Source File |
|------------|----------|-------------|------------|
| Alert Analyst | Incident Response | Analyzes Datadog alerts, correlates with recent changes, provides root cause analysis | [datadog/agents.md](../../datadog/agents.md) |
| Dashboard Builder Agent | Visualization | Autonomously creates and optimizes Datadog dashboards based on service topology | [datadog/agents.md](../../datadog/agents.md) |
| SLO Manager Agent | Reliability | Manages SLO definitions, tracks burn rates, and alerts on budget consumption | [datadog/agents.md](../../datadog/agents.md) |

---

## Atlassian Agents

| Agent Name | Category | Description | Source File |
|------------|----------|-------------|------------|
| Backlog Manager Agent | Project Management | Triages issues, maintains priorities, identifies duplicates, keeps backlog healthy | [atlassian/agents.md](../../atlassian/agents.md) |
| Sprint Planner Agent | Project Management | Plans sprints based on team velocity, priorities, and dependencies | [atlassian/agents.md](../../atlassian/agents.md) |
| Documentation Agent | Documentation | Maintains Confluence documentation aligned with code changes | [atlassian/agents.md](../../atlassian/agents.md) |

---

## Slack Agents

| Agent Name | Category | Description | Source File |
|------------|----------|-------------|------------|
| Incident Commander Bot | Incident Response | Creates incident channels, pages responders, tracks timelines, generates post-mortems | [slack/agents.md](../../slack/agents.md) |
| Channel Curator Agent | Communication | Manages Slack channels -- archives stale ones, enforces naming, maintains topics | [slack/agents.md](../../slack/agents.md) |
| Notification Router Agent | Communication | Routes notifications to appropriate channels based on content and urgency | [slack/agents.md](../../slack/agents.md) |

---

## Cloudflare Agents

| Agent Name | Category | Description | Source File |
|------------|----------|-------------|------------|
| Edge Deployment Agent | DevOps | Full lifecycle Cloudflare Workers and Pages deployments with staging and rollback | [cloudflare/agents.md](../../cloudflare/agents.md) |
| Security Agent | Security | Monitors and manages WAF, DDoS protection, bot management, and SSL configuration | [cloudflare/agents.md](../../cloudflare/agents.md) |
| Performance Optimization Agent | Performance | Analyzes and optimizes edge caching, routing, and Worker performance | [cloudflare/agents.md](../../cloudflare/agents.md) |

---

## ChatOps / AI SRE Agents

| Agent Name | Category | Description | Source File |
|------------|----------|-------------|------------|
| Incident Commander Agent | Incident Response | Coordinates full incident lifecycle -- declaration, investigation, mitigation, resolution | [chatops/agents.md](../../chatops/agents.md) |
| Root Cause Analyzer Agent | Investigation | Autonomous root cause analysis using metrics, logs, traces, and change correlation | [chatops/agents.md](../../chatops/agents.md) |
| Post-Mortem Writer Agent | Documentation | Generates blameless post-mortems with timeline, root cause, and action items | [chatops/agents.md](../../chatops/agents.md) |
| Alert Correlator Agent | Investigation | Correlates alerts across monitoring platforms to identify related incidents | [chatops/agents.md](../../chatops/agents.md) |
| Runbook Executor Agent | Remediation | Executes runbooks with safety checks, approval gates, and rollback capabilities | [chatops/agents.md](../../chatops/agents.md) |
| Health Monitor Agent | Monitoring | Continuous health monitoring across all platforms with anomaly detection | [chatops/agents.md](../../chatops/agents.md) |

---

## Multi-Tool Integration Agents

| Agent Name | Category | Description | Source File |
|------------|----------|-------------|------------|
| The Review Board | Code Review | Multi-model review agent getting opinions from Claude, Gemini, and Codex | [working_nicely/example_agents.md](../../working_nicely/example_agents.md) |
| Research-Implement-Verify Loop | Development | Uses Gemini for research, Claude for implementation, Codex for verification | [working_nicely/example_agents.md](../../working_nicely/example_agents.md) |
| Continuous Improvement Agent | Quality | Monitors codebase and suggests improvements using multiple AI tools | [working_nicely/example_agents.md](../../working_nicely/example_agents.md) |
| Multi-Agent Task Decomposer | Orchestration | Decomposes complex tasks and distributes across specialized AI tools | [working_nicely/example_agents.md](../../working_nicely/example_agents.md) |
| Cross-Agent Memory Coordinator | Context | Manages shared context and memory across multiple AI tool sessions | [working_nicely/example_agents.md](../../working_nicely/example_agents.md) |
| Claude Code Agent Teams with External Specialists | Orchestration | Claude Code agent teams augmented with external AI specialist tools | [working_nicely/example_agents.md](../../working_nicely/example_agents.md) |

---

## Advanced Orchestration Patterns

These are meta-patterns for composing agents, documented in [advanced/multi_agent_orchestration.md](../../advanced/multi_agent_orchestration.md).

| Pattern Name | Category | Description | Source File |
|--------------|----------|-------------|------------|
| Supervisor (Orchestrator-Worker) | Orchestration | Central orchestrator decomposes tasks, assigns to workers, merges results | [advanced/multi_agent_orchestration.md](../../advanced/multi_agent_orchestration.md) |
| Pipeline (Sequential) | Orchestration | Each agent's output feeds into the next (e.g., Spec > Test > Implement > Review) | [advanced/multi_agent_orchestration.md](../../advanced/multi_agent_orchestration.md) |
| Swarm | Orchestration | Multiple agents work on independent subtasks in parallel | [advanced/multi_agent_orchestration.md](../../advanced/multi_agent_orchestration.md) |
| Debate (Evaluator-Optimizer) | Orchestration | Two agents argue opposing positions, a judge selects the best outcome | [advanced/multi_agent_orchestration.md](../../advanced/multi_agent_orchestration.md) |

---

## Agent Framework Patterns

Higher-level patterns for building multi-agent systems, documented in [agent_frameworks/multi_agent_patterns.md](../../agent_frameworks/multi_agent_patterns.md).

| Pattern Name | Category | Description | Source File |
|--------------|----------|-------------|------------|
| Supervisor / Manager | Framework | Central supervisor receives, decomposes, delegates, monitors, and synthesizes | [agent_frameworks/multi_agent_patterns.md](../../agent_frameworks/multi_agent_patterns.md) |
| Pipeline / Sequential Chain | Framework | Agents execute in sequence, each transforming the output of the previous | [agent_frameworks/multi_agent_patterns.md](../../agent_frameworks/multi_agent_patterns.md) |
| Debate / Adversarial Review | Framework | Agents present competing solutions; a judge evaluates and selects | [agent_frameworks/multi_agent_patterns.md](../../agent_frameworks/multi_agent_patterns.md) |
| Map-Reduce / Fan-Out | Framework | Split task into independent subtasks, process in parallel, aggregate results | [agent_frameworks/multi_agent_patterns.md](../../agent_frameworks/multi_agent_patterns.md) |
| Hierarchical Teams | Framework | Multi-level agent hierarchy with team leads and specialists | [agent_frameworks/multi_agent_patterns.md](../../agent_frameworks/multi_agent_patterns.md) |
| Blackboard / Shared Workspace | Framework | Agents read/write to a shared workspace, self-selecting tasks | [agent_frameworks/multi_agent_patterns.md](../../agent_frameworks/multi_agent_patterns.md) |

---

## Advanced Specialist Agents

Standalone agent guides in the [advanced/](../../advanced/) directory.

| Agent Name | Category | Description | Source File |
|------------|----------|-------------|------------|
| Performance Agent | Performance | Profiling, bottleneck detection, optimization suggestions | [advanced/performance_agent.md](../../advanced/performance_agent.md) |
| Database Agent | Data | Schema design, migration generation, query optimization | [advanced/database_agent.md](../../advanced/database_agent.md) |
| API Design Agent | Architecture | OpenAPI spec generation, backward compatibility checks | [advanced/api_design_agent.md](../../advanced/api_design_agent.md) |

---

## How to Use This Index

1. **Find an agent** by scanning the tables above or searching by category
2. **Read the source file** linked in the last column for full implementation details (behavioral rules, tool permissions, example workflows)
3. **Copy the agent definition** into your project's `.claude/skills/` directory with `agent: true` in the frontmatter
4. See [How_to_use.md](../../How_to_use.md) for detailed installation instructions
5. For orchestration patterns (composing multiple agents), see [advanced/multi_agent_orchestration.md](../../advanced/multi_agent_orchestration.md) and [agent_frameworks/multi_agent_patterns.md](../../agent_frameworks/multi_agent_patterns.md)

---

*Part of the [Claude Framework](../../README.md). See [index.md](../../index.md) for the full file listing.*
