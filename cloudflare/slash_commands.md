# Cloudflare Slash Commands

## Command Reference

| Command | Description | Example |
|---------|-------------|---------|
| `/cf-deploy` | Deploy to Cloudflare | `/cf-deploy Deploy my-worker to production` |
| `/cf-dns` | Manage DNS records | `/cf-dns List all records for example.com` |
| `/cf-security` | Security audit/config | `/cf-security Audit example.com security` |
| `/cf-cache` | Cache management | `/cf-cache Purge cache for api.example.com` |
| `/cf-worker` | Create/manage Workers | `/cf-worker Create an API proxy worker` |
| `/cf-pages` | Deploy to Pages | `/cf-pages Deploy the frontend to Pages` |
| `/cf-status` | Zone status overview | `/cf-status` |

---

## `/cf-deploy`

```yaml
---
name: cf-deploy
description: Deploy Workers, Pages, or infrastructure to Cloudflare
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Write
  - mcp__cloudflare__*
---
```

### Usage

```
# Deploy a Worker
/cf-deploy Deploy my-api-worker to production

# Deploy to Pages
/cf-deploy Deploy the dist/ folder to Cloudflare Pages

# Deploy with preview
/cf-deploy Deploy my-worker to staging and test it

# Rollback
/cf-deploy Rollback my-worker to the previous version
```

---

## `/cf-dns`

```yaml
---
name: cf-dns
description: Manage Cloudflare DNS records
user-invocable: true
allowed-tools:
  - Bash
  - mcp__cloudflare__*
---
```

### Usage

```
# List records
/cf-dns Show all DNS records for example.com

# Create record
/cf-dns Add an A record for api.example.com pointing to 192.0.2.1

# Update record
/cf-dns Change api.example.com to point to 192.0.2.2

# Audit
/cf-dns Check for stale or misconfigured DNS records

# Email setup
/cf-dns Set up SPF, DKIM, and DMARC for example.com
```

### Output Example

```
## DNS Records: example.com

| Type | Name | Content | Proxy | TTL |
|------|------|---------|-------|-----|
| A | example.com | 192.0.2.1 | On | Auto |
| A | api | 192.0.2.2 | On | Auto |
| CNAME | www | example.com | On | Auto |
| MX | example.com | mail.example.com (10) | Off | 3600 |
| TXT | example.com | v=spf1 include:_spf.google.com ~all | Off | 3600 |

### Issues Found
- Missing DMARC record (recommended: v=DMARC1; p=quarantine)
- DKIM not configured for Google Workspace
```

---

## `/cf-security`

### Usage

```
# Full audit
/cf-security Run a security audit on example.com

# Specific checks
/cf-security Check WAF rules for api.example.com

# Add rate limiting
/cf-security Add rate limiting to /api/login - 10 requests per minute

# Block IPs
/cf-security Block IP 203.0.113.0/24 - automated scanning detected

# Review firewall events
/cf-security Show firewall events from the last 24 hours
```

---

## `/cf-cache`

### Usage

```
# Purge all
/cf-cache Purge all cache for example.com

# Purge specific URLs
/cf-cache Purge cache for https://example.com/api/config

# Purge by tag
/cf-cache Purge all cached assets tagged with v2.3.0

# Check hit rate
/cf-cache What's the cache hit rate for example.com?

# Optimize
/cf-cache Suggest cache optimizations for example.com
```

---

## `/cf-worker`

```yaml
---
name: cf-worker
description: Create and manage Cloudflare Workers
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - mcp__cloudflare__*
---
```

### Usage

```
# Create a new Worker
/cf-worker Create an API gateway worker that routes to different backends

# Create an MCP server
/cf-worker Create an MCP server worker for our internal API

# Debug a Worker
/cf-worker Why is my-worker returning 500 errors?

# Analyze performance
/cf-worker How is my-worker performing? Check CPU time and request volume
```

---

## `/cf-pages`

### Usage

```
# Deploy
/cf-pages Deploy the build output to production

# Preview
/cf-pages Create a preview deployment for this PR

# List deployments
/cf-pages Show recent deployments for my-site

# Rollback
/cf-pages Rollback my-site to the previous deployment
```

---

## `/cf-status`

```yaml
---
name: cf-status
description: Quick status overview of Cloudflare zones and services
user-invocable: true
allowed-tools:
  - Bash
  - mcp__cloudflare__*
---
```

### Usage

```
/cf-status
/cf-status Check example.com
```

### Output Example

```
## Cloudflare Status: example.com

### Overview
- Plan: Pro
- Status: Active
- SSL: Full (Strict)
- Proxy: Enabled

### Traffic (last 24h)
- Total requests: 2.4M
- Cached: 2.1M (87.5%)
- Threats blocked: 1,234
- Bandwidth: 45 GB (38 GB cached)

### Workers
| Worker | Requests | Errors | CPU (p50) |
|--------|----------|--------|-----------|
| api-gateway | 145K | 23 (0.02%) | 3.2ms |
| auth-proxy | 89K | 0 | 1.1ms |

### Security Events (last 24h)
- WAF blocks: 456
- Rate limited: 89
- Bot challenges: 234
- DDoS mitigated: 2 events

### DNS
- Records: 24
- DNSSEC: Enabled
- Last change: 3 days ago
```
