---
description: Trigger an incident management workflow — investigate, diagnose, mitigate, and document production issues
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
model: opus
argument-hint: "[severity:SEV1|SEV2|SEV3] [description of the incident]"
---

# Incident Management Workflow

You are running an AI-assisted incident management workflow. This is a structured process for investigating, mitigating, and documenting production incidents.

## Parse Arguments

Parse `$ARGUMENTS` for:
- **Severity**: Look for `SEV1`, `SEV2`, `SEV3`, `sev1`, `sev2`, `sev3`, `P1`, `P2`, `P3`. Default to `SEV2` if not specified.
- **Description**: Everything else in the arguments is the incident description.

If `$ARGUMENTS` is empty, ask the user: "Describe the incident. Include severity if known (SEV1/SEV2/SEV3) and what symptoms you are seeing."

## Severity Definitions

| Level | Meaning | Response Time | Examples |
|-------|---------|---------------|----------|
| **SEV1** | Critical — service down, data loss, security breach | Immediate, all hands | Full outage, data corruption, active exploit |
| **SEV2** | Major — degraded service, significant user impact | Within 30 min | Partial outage, high error rates, performance degradation |
| **SEV3** | Minor — limited impact, workaround exists | Within 2 hours | Intermittent errors, non-critical feature broken, cosmetic issues |

## Phase 1: Triage (Do This First)

1. **Create incident log**: Create `claudefiles/incidents/{YYYY-MM-DD}_{short-slug}.md` with:
   - Incident ID: `INC-{YYYYMMDD}-{HHMM}`
   - Severity level
   - Description
   - Reporter (user)
   - Start time (now)
   - Status: INVESTIGATING

2. **Announce**: Print a clear incident header:
   ```
   === INCIDENT: INC-{id} | {SEVERITY} | {short description} ===
   Status: INVESTIGATING
   Started: {timestamp}
   ```

3. **Initial assessment**: Based on the description, identify:
   - What service/component is affected?
   - What is the user-facing impact?
   - What changed recently? (check `git log --oneline -20`)
   - Are there any recent deployments? (check CI/CD if available)

## Phase 2: Investigate

Run these diagnostic steps based on what is available in the project:

### Code Investigation
1. Check recent commits: `git log --oneline --since="24 hours ago"`
2. Check for recent changes to critical files: `git diff HEAD~5 --name-only`
3. Search for error patterns mentioned in the incident description
4. Check configuration files for recent changes

### Log Investigation (if accessible)
1. Search for error patterns in any log files present in the project
2. Look for stack traces, error codes, or exception patterns
3. Check for patterns: sudden spikes, repeated errors, timeout patterns

### Infrastructure Investigation (if MCP servers available)
1. If GitHub MCP is available: check recent PRs, CI status, deployment status
2. If database MCP is available: check connection pool, slow queries, locks
3. If monitoring MCP is available: check error rates, latency, resource usage

### Document Findings
Update the incident log with each finding as you discover it. Use timestamps for each entry.

## Phase 3: Identify Root Cause

Based on investigation, determine:
1. **Root cause hypothesis**: What do you believe caused this?
2. **Contributing factors**: What made it worse or harder to detect?
3. **Evidence**: What data supports this hypothesis?
4. **Confidence level**: High / Medium / Low

Update the incident log with the root cause analysis.

## Phase 4: Mitigate

Propose mitigation options ranked by speed and risk:

| Option | Speed | Risk | Reversible? | Description |
|--------|-------|------|-------------|-------------|
| 1 | ... | ... | ... | ... |
| 2 | ... | ... | ... | ... |

**Ask the user** which mitigation to proceed with. Do NOT auto-apply fixes for SEV1/SEV2 without confirmation.

For SEV3 with an obvious fix, you may propose and implement with user confirmation.

After mitigation:
1. Verify the fix resolves the symptoms
2. Update incident status to MITIGATED
3. Log the mitigation action with timestamp

## Phase 5: Document & Learn

1. **Update incident log** with:
   - Resolution time
   - Root cause (confirmed)
   - Mitigation applied
   - Status: RESOLVED

2. **Create postmortem stub** in `claudefiles/incidents/{date}_{slug}_postmortem.md`:
   - Timeline of events
   - Root cause analysis
   - What went well
   - What could be improved
   - Action items (with owners if known)

3. **Update changelog**: Log the incident and resolution in `claudefiles/changelog.md`

4. **Check for skill extraction**: Could any part of this investigation become a reusable diagnostic skill? If so, note it for the skills master.

5. **Update CLAUDE.md if needed**: If the incident revealed a gotcha or non-obvious behavior, add it to the project's CLAUDE.md or `.claude/rules/`.

## Communication Templates

### Status Update (use during investigation)
```
=== INCIDENT UPDATE: INC-{id} | {SEVERITY} ===
Status: {INVESTIGATING | IDENTIFIED | MITIGATING | MITIGATED | RESOLVED}
Duration: {time since start}
Impact: {current user impact}
Next steps: {what we are doing now}
```

### Resolution Notice
```
=== INCIDENT RESOLVED: INC-{id} ===
Duration: {total time}
Root cause: {one-line summary}
Resolution: {what fixed it}
Postmortem: {link to postmortem file}
```

## Rules

- **Always create the incident log first** before any investigation.
- **Timestamp every action** in the incident log.
- **Never auto-deploy fixes** without user confirmation.
- **Bias toward reversible mitigations** (rollback > hotfix > config change > code change).
- **If SEV1**: Emphasize speed. Skip non-essential documentation during active mitigation; backfill after resolution.
- **If the incident is outside your capability** (requires cloud console access, database admin, etc.), clearly state what you need the user to do manually.
