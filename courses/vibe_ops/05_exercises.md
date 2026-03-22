# Module 05: Interactive Exercises

---

## How to Use These Exercises

Each exercise presents an operations scenario. Work through them using Claude Code in your vibe-ops-lab directory from Module 02. Focus on the natural language interactions rather than writing scripts by hand.

**Difficulty levels:**
- **Starter** -- Single-workflow exercises
- **Intermediate** -- Multi-step scenarios with decision points
- **Advanced** -- Full incident simulations

---

## Exercise 1: First Deploy (Starter)

### Scenario

You've just finished building a feature on branch `feature/user-profiles`. It's been tested locally and you want to deploy it to staging.

### Task

Using natural language in Claude Code, walk through:
1. Pre-deployment checks (what should we verify?)
2. The deployment itself
3. Post-deployment verification
4. Promoting to production (if staging looks good)

### What to Practice

- Clear, specific deployment requests
- Asking for status checks before and after
- Requesting confirmation flows for production

<details>
<summary>Sample Interaction</summary>

```
You:   I want to deploy the feature/user-profiles branch to staging.
       What checks should we do first?

AI:    Before deploying feature/user-profiles to staging, let's verify:

       Pre-deploy checklist:
       [ok] Branch exists and is up-to-date with main
       [ok] CI pipeline passed (all 142 tests green)
       [ok] No merge conflicts with main
       [ok] Staging environment is healthy and not in use
       [ok] Last staging deploy was 3 days ago (stable)

       Ready to deploy. Shall I proceed?

You:   Yes, deploy it.

AI:    Deploying feature/user-profiles to staging...
       [14:30:01] Building Docker image...
       [14:30:15] Build complete. Running smoke tests...
       [14:30:22] Smoke tests passed. Deploying to staging...
       [14:30:45] Deployment complete.

       Post-deploy verification:
       - Health check: PASSING
       - New endpoint /api/users/profile: RESPONDING (200 OK)
       - Error rate: 0%
       - Response time: 45ms (normal)

       Preview: https://staging.example.com/profiles

       Would you like to run the full integration test suite
       against staging, or are you ready to promote to production?

You:   Run integration tests first.

AI:    Running integration test suite against staging...
       [14:31:00] Auth tests: 24/24 passed
       [14:31:15] User tests: 31/31 passed
       [14:31:20] Profile tests (new): 12/12 passed
       [14:31:30] Payment tests: 18/18 passed
       [14:31:35] Total: 85/85 passed

       All tests passing. Ready for production promotion.
       Production deploy requires confirmation. Type 'confirm' to proceed.
```

</details>

---

## Exercise 2: Status Investigation (Starter)

### Scenario

It's Monday morning. You've just opened your laptop and want to know how things are running before starting work.

### Task

Ask the AI for:
1. Overall system health
2. Any incidents over the weekend
3. Resource utilization trends
4. Anything that needs attention

### What to Practice

- Broad, open-ended questions about system state
- Following up on warnings or anomalies
- Asking for trend analysis (not just current state)

<details>
<summary>Sample Interaction</summary>

```
You:   Good morning. How's everything running? Anything happen over the weekend?

AI:    Good morning! Here's your Monday morning briefing:

       System Health: All services healthy
       api-service     HEALTHY  3/3 pods   CPU: 35%   Mem: 52%
       web-frontend    HEALTHY  2/2 pods   CPU: 12%   Mem: 28%
       worker-service  HEALTHY  2/2 pods   CPU: 44%   Mem: 61%
       auth-service    HEALTHY  2/2 pods   CPU: 18%   Mem: 39%

       Weekend Summary:
       - No incidents
       - 1 auto-scaling event: Saturday 22:00 - worker-service scaled
         2 -> 3 pods for 4 hours due to batch job processing
       - 2 deployment attempts (both to staging, both successful)

       Trends to Watch:
       - worker-service memory usage has been trending up over the past
         week (was 48% last Monday, now 61%). Not urgent but worth
         monitoring for a potential memory leak.
       - Database storage at 72% (was 68% last week). At current growth
         rate, you have ~45 days before hitting 90%.

       Action Items:
       1. Consider investigating worker-service memory trend
       2. Plan database storage expansion or cleanup within 30 days

       Anything you'd like me to dig into?

You:   Tell me more about the worker-service memory trend. Show me the
       daily averages for the past week.

AI:    Worker-service memory usage (daily average, past 7 days):

       Mon 3/15: 48%  ||||||||||||||||||||||||
       Tue 3/16: 50%  |||||||||||||||||||||||||
       Wed 3/17: 51%  ||||||||||||||||||||||||||
       Thu 3/18: 54%  |||||||||||||||||||||||||||
       Fri 3/19: 56%  ||||||||||||||||||||||||||||
       Sat 3/20: 58%  |||||||||||||||||||||||||||||
       Sun 3/21: 61%  ||||||||||||||||||||||||||||||

       This is a steady ~2% daily increase with no recovery on restarts,
       which is consistent with a memory leak pattern.

       The last deployment to worker-service was March 14 (commit def789:
       "Add batch report generation"). This aligns with when the trend
       started.

       Recommended: Review the batch report generation code for objects
       that aren't being garbage collected. Most likely cause: large
       report data being held in memory instead of streamed to disk.
```

</details>

---

## Exercise 3: Scaling for a Product Launch (Intermediate)

### Scenario

Your marketing team is launching a campaign tomorrow at 10am that's expected to drive 5-10x normal traffic. You need to prepare the infrastructure today.

### Task

Work with the AI to:
1. Assess current capacity
2. Plan scaling for each service
3. Estimate costs
4. Implement the scaling changes
5. Set up monitoring for the event
6. Plan automatic scale-down after the event

### What to Practice

- Collaborative planning with the AI
- Cost awareness
- Setting up temporary operational changes
- Pre-event checklists

<details>
<summary>Key prompts to try</summary>

- "We have a product launch tomorrow at 10am expecting 5-10x traffic. Help me prepare."
- "What's our current capacity and what do we need for 10x?"
- "Show me the cost breakdown for the scaling plan."
- "Set up enhanced monitoring during the event -- I want alerts at lower thresholds."
- "Create a scale-down plan for 24 hours after launch."
- "Generate a launch day runbook with everything we've discussed."

</details>

---

## Exercise 4: Full Incident Response (Advanced)

### Scenario

It's 3pm on a Tuesday. You receive this alert:

> CRITICAL: api-service error rate at 25%. 500 errors on /api/checkout. Started 2 minutes ago.

### Task

Walk through the complete incident lifecycle:
1. Acknowledge the alert and begin triage
2. Gather context (recent changes, dependent services, etc.)
3. Diagnose the root cause
4. Implement a fix
5. Verify recovery
6. Generate a post-mortem

### What to Practice

- Staying calm and systematic under pressure
- Using the AI to correlate data quickly
- Making decisions about mitigation vs. root cause fix
- Documenting as you go

<details>
<summary>Guided walkthrough</summary>

**Step 1: Acknowledge and triage**
```
You:   We have a critical alert: 25% error rate on /api/checkout.
       Triage this immediately.
```

**Step 2: Gather context**
```
You:   What changed in the last hour? Any deployments, config changes,
       or dependent service issues?
```

**Step 3: Narrow down the cause**
```
You:   The errors started right after that deployment. Show me what
       changed in that commit and which endpoints are affected.
```

**Step 4: Decide on action**
```
You:   Should we rollback or can we hotfix this? What's faster?
```

**Step 5: Execute and verify**
```
You:   Roll it back. Monitor the error rate and tell me when it's
       back to normal.
```

**Step 6: Post-mortem**
```
You:   Generate a post-mortem for this incident. Include timeline,
       root cause, impact, and action items.
```

</details>

---

## Exercise 5: Design a Complete ChatOps System (Advanced)

### Scenario

A startup with 5 developers has asked you to design their ChatOps system from scratch. They use:
- AWS (ECS for containers)
- GitHub for code
- Slack for communication
- Datadog for monitoring
- PostgreSQL on RDS

### Task

Design and document:
1. Slack channel structure
2. All slash commands they need
3. Alert routing rules
4. Deployment workflow (staging -> production)
5. On-call rotation integration
6. Self-healing rules for common scenarios
7. Security guardrails

### Deliverable

Create a complete ChatOps design document using Claude Code. It should include Mermaid diagrams for workflows and be something the team could actually implement.

<details>
<summary>Starting prompt</summary>

> Design a complete ChatOps system for a 5-person startup using AWS ECS, GitHub, Slack, Datadog, and PostgreSQL. I need Slack channel structure, slash commands, alert routing, deployment workflows, on-call integration, self-healing rules, and security guardrails. Create a comprehensive design document with Mermaid diagrams. Make it practical -- something we could actually implement in a week.

</details>

---

## Exercise 6: Incident Game Day (Advanced)

### Scenario

Run a "game day" where you simulate 3 incidents in sequence, each building on the previous one's resolution.

### The Incidents

**Incident 1 (Easy):** Worker service runs out of memory and crashes.

**Incident 2 (Medium):** After fixing Incident 1, you notice the fix (increasing memory limits) has increased your AWS bill. A cost alert fires showing you're 40% over budget for the month.

**Incident 3 (Hard):** While investigating costs, you discover that a misconfigured auto-scaling rule has been spinning up instances in an unused region for the past 2 weeks.

### Task

For each incident:
1. Detect and triage
2. Diagnose
3. Fix
4. Verify
5. Update the system to prevent recurrence

Then: Generate a combined post-mortem covering all three incidents and how they're related.

<details>
<summary>Starting the game day</summary>

```
You:   Let's run an incident game day. I'll work through 3 simulated
       incidents. Start with Incident 1: the worker service has crashed
       with an OOM error. Walk me through detection, triage, and diagnosis.
       Generate realistic data.
```

After resolving each incident, say:
```
You:   Incident 1 resolved. Trigger Incident 2: a cost alert fires
       showing we're 40% over budget for the month.
```

</details>

---

## Self-Assessment Rubric

Rate yourself on each exercise:

| Criteria | Points |
|----------|--------|
| **Communication clarity** -- Were your prompts specific and actionable? | /25 |
| **Systematic approach** -- Did you follow a logical process? | /25 |
| **Decision quality** -- Were your choices appropriate for the scenario? | /25 |
| **Learning extraction** -- Did you identify prevention measures? | /25 |

### Score Guide

| Score | Level |
|-------|-------|
| 90-100 | Ready to implement vibe ops in production |
| 70-89 | Strong understanding, practice the advanced exercises more |
| 50-69 | Revisit Modules 03 and 04 before proceeding |
| Below 50 | Start from Module 01 again with fresh eyes |

---

## What's Next?

Congratulations on completing the Vibe Ops course. Continue your learning:

1. **Implement ChatOps for a real project** -- even a personal one
2. **Explore the [Claude Code course](../claude_code/README.md)** for deep CLI expertise
3. **Join SRE and DevOps communities** to see how others implement AI ops
4. **Read incident post-mortems** from companies like Google, Cloudflare, and GitHub to learn patterns
5. **Practice incident response** regularly with game days
