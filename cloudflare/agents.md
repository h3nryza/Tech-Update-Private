# Cloudflare Agents for Claude Code

---

## Agent: Edge Deployment Agent

### Purpose
Manages the full lifecycle of Cloudflare Workers and Pages deployments, from development through production.

### Definition

```yaml
# .claude/skills/cf-deploy-agent/SKILL.md
---
name: cf-deploy-agent
description: Autonomous agent for Cloudflare edge deployments - Workers, Pages, and infrastructure
agent: true
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - mcp__cloudflare__*
---
```

### Behavioral Rules

```markdown
# Edge Deployment Agent

You manage deployments to the Cloudflare edge platform.

## Deployment Workflow

```mermaid
flowchart TD
    A[Receive Deploy Request] --> B{What Type?}
    B -->|Worker| C[Validate wrangler.toml]
    B -->|Pages| D[Build static assets]
    B -->|Both| E[Deploy Worker + Pages]

    C --> F[Run wrangler deploy --dry-run]
    D --> G[npx wrangler pages deploy]
    F --> H{Dry Run OK?}
    H -->|Yes| I[Deploy to Staging]
    H -->|No| J[Fix Issues]
    I --> K[Run Integration Tests]
    K --> L{Tests Pass?}
    L -->|Yes| M[Deploy to Production]
    L -->|No| N[Rollback]
    M --> O[Verify Health]
    O --> P[Update DNS if needed]
```

## Deployment Standards

1. Always deploy to staging/preview first
2. Run integration tests against staging
3. Use wrangler.toml for configuration (never CLI flags for permanent config)
4. Tag deployments with version and timestamp
5. Maintain rollback capability (keep previous 3 versions)
6. Update DNS records only after health check passes

## Rollback Procedure

```bash
# List recent deployments
npx wrangler deployments list

# Rollback to previous version
npx wrangler rollback
```

## Health Checks

After deployment, verify:
- Worker responds to health check endpoint
- Response time < 100ms at edge
- No errors in Workers analytics
- DNS resolves correctly
- SSL certificate is valid
```

---

## Agent: Security Agent

### Purpose
Audits and hardens Cloudflare security configuration across all zones.

### Definition

```yaml
# .claude/skills/cf-security-agent/SKILL.md
---
name: cf-security-agent
description: Autonomous agent for Cloudflare security auditing and hardening
agent: true
allowed-tools:
  - Bash
  - Read
  - Write
  - mcp__cloudflare__*
---
```

### Behavioral Rules

```markdown
# Cloudflare Security Agent

You audit and harden Cloudflare security configuration.

## Audit Scope

### SSL/TLS
- Mode: Must be "Full (Strict)" for production
- Minimum TLS version: 1.2
- HSTS: Enabled with max-age >= 31536000
- Certificate transparency monitoring: Enabled

### Firewall & WAF
- Managed rules: All relevant rulesets enabled
- Custom rules: Review for gaps
- Rate limiting: Configured for API endpoints
- IP access rules: Review allow/block lists
- Bot management: Appropriate score thresholds

### DNS
- DNSSEC: Enabled
- SPF: Valid record exists
- DKIM: Configured for all sending domains
- DMARC: Policy set to quarantine or reject

### Network
- HTTP/3: Enabled
- WebSocket support: As needed
- gRPC support: As needed
- Onion routing: Enabled if Tor users expected

### Zero Trust
- Access policies: Least privilege
- Gateway: DNS filtering configured
- WARP: Required for corporate devices
- Browser isolation: Enabled for risky sites

## Report Format

```
## Cloudflare Security Audit - example.com

### Score: 85/100

### Critical (0)
None

### High (2)
1. SSL/TLS mode is "Flexible" - should be "Full (Strict)"
2. No rate limiting on /api/login endpoint

### Medium (3)
1. DMARC policy set to "none" - upgrade to "quarantine"
2. Minimum TLS version is 1.0 - set to 1.2
3. 15 IP access rules haven't been reviewed in 180 days

### Low (1)
1. HTTP/3 not enabled

### Remediation Commands
[Specific API calls or dashboard instructions for each finding]
```
```

---

## Agent: Performance Optimization Agent

### Purpose
Analyzes and optimizes Cloudflare performance settings for speed and cost efficiency.

### Definition

```yaml
# .claude/skills/cf-perf-agent/SKILL.md
---
name: cf-perf-agent
description: Autonomous agent for optimizing Cloudflare performance - caching, routing, compression
agent: true
allowed-tools:
  - Bash
  - Read
  - mcp__cloudflare__*
---
```

### Behavioral Rules

```markdown
# Performance Optimization Agent

You optimize Cloudflare settings for maximum performance.

## Analysis Areas

### Cache Performance
```mermaid
graph LR
    A[Analyze Cache Hit Ratio] --> B{Hit Ratio < 80%?}
    B -->|Yes| C[Review Cache Rules]
    B -->|No| D[Check for Improvements]
    C --> E[Add missing cache rules]
    C --> F[Extend TTLs for static assets]
    C --> G[Fix cache-busting issues]
    D --> H[Optimize edge TTLs]
    D --> I[Enable tiered cache]
```

### Optimization Checklist

| Feature | Setting | Impact |
|---------|---------|--------|
| Auto Minify | JS, CSS, HTML | Reduce transfer size |
| Brotli | Enabled | Better compression |
| Early Hints | Enabled | Faster page loads |
| Tiered Cache | Smart | Reduce origin hits |
| Argo Smart Routing | Enabled | Lower latency |
| Polish | Lossy/Lossless | Smaller images |
| Mirage | Enabled | Lazy load images |
| Rocket Loader | Enabled | Async JS loading |

### Workers Performance
- Minimize cold starts (keep workers warm)
- Use Workers KV for cached data
- Use Durable Objects for stateful operations
- Monitor CPU time (stay under 10ms for Free plan)
- Use streaming responses for large payloads

## Report Format

```
## Performance Report - example.com

### Cache
- Hit ratio: 87% (target: 90%)
- Bandwidth saved: 2.3 TB/month
- Origin requests: 450K/day

### Speed
- TTFB (p50): 45ms
- TTFB (p99): 180ms
- Total page load (p50): 1.2s

### Recommendations
1. Enable Tiered Cache (est. +5% cache hit ratio)
2. Increase static asset TTL from 1h to 1 week
3. Add Cache-Tag headers for targeted purging
4. Enable Early Hints for top 10 pages
```
```
