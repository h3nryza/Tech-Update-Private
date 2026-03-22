# Cloudflare Skills for Claude Code

---

## Skill: Workers Development

### `.claude/skills/cf-workers/SKILL.md`

```yaml
---
name: cf-workers
description: Create, deploy, and manage Cloudflare Workers
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - mcp__cloudflare__*
---
```

```markdown
# Cloudflare Workers Skill

## Create a Worker

### From Prompt
```bash
# Use wrangler to scaffold
npx wrangler init my-worker --type javascript

# Or use Claude Code with Cloudflare docs MCP
# Claude generates the worker code based on requirements
```

### Worker Templates

#### Basic API Handler
```typescript
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const url = new URL(request.url);

    if (url.pathname === "/api/health") {
      return Response.json({ status: "ok", timestamp: Date.now() });
    }

    if (url.pathname.startsWith("/api/")) {
      return handleAPI(request, env);
    }

    return new Response("Not Found", { status: 404 });
  }
};
```

#### Worker with KV
```typescript
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const key = new URL(request.url).pathname.slice(1);

    if (request.method === "GET") {
      const value = await env.MY_KV.get(key);
      return value
        ? Response.json({ key, value: JSON.parse(value) })
        : new Response("Not Found", { status: 404 });
    }

    if (request.method === "PUT") {
      const body = await request.json();
      await env.MY_KV.put(key, JSON.stringify(body), { expirationTtl: 86400 });
      return Response.json({ key, status: "saved" });
    }

    return new Response("Method Not Allowed", { status: 405 });
  }
};
```

#### Worker with D1 Database
```typescript
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const { results } = await env.DB.prepare(
      "SELECT * FROM users WHERE active = ?"
    ).bind(1).all();

    return Response.json(results);
  }
};
```

#### MCP Server Worker
```typescript
import { McpAgent } from "agents/mcp";
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { z } from "zod";

export class MyMCP extends McpAgent {
  server = new McpServer({ name: "my-api", version: "1.0.0" });

  async init() {
    this.server.tool("search", "Search my data", {
      query: z.string().describe("Search query")
    }, async ({ query }) => {
      // Implementation
      return { content: [{ type: "text", text: "results" }] };
    });
  }
}
```

## Deploy

```bash
# Deploy to production
npx wrangler deploy

# Deploy to staging
npx wrangler deploy --env staging

# Deploy with specific route
npx wrangler deploy --route "api.example.com/*"
```

## Wrangler Configuration

```toml
# wrangler.toml
name = "my-worker"
main = "src/index.ts"
compatibility_date = "2026-03-01"

[vars]
ENVIRONMENT = "production"

[[kv_namespaces]]
binding = "MY_KV"
id = "abc123"

[[d1_databases]]
binding = "DB"
database_name = "my-db"
database_id = "def456"

[[r2_buckets]]
binding = "STORAGE"
bucket_name = "my-bucket"
```
```

---

## Skill: DNS Management

### `.claude/skills/cf-dns/SKILL.md`

```yaml
---
name: cf-dns
description: Manage Cloudflare DNS records - create, update, delete, audit
allowed-tools:
  - Bash
  - Read
  - mcp__cloudflare__*
---
```

```markdown
# DNS Management Skill

## Common Operations

### List DNS Records
```bash
curl -s -X GET "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records" \
  -H "Authorization: Bearer $CF_API_TOKEN" \
  -H "Content-Type: application/json" | jq '.result[] | {name, type, content, proxied, ttl}'
```

### Create Record
```bash
curl -s -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records" \
  -H "Authorization: Bearer $CF_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "A",
    "name": "api.example.com",
    "content": "192.0.2.1",
    "ttl": 1,
    "proxied": true
  }'
```

### Update Record
```bash
curl -s -X PUT "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records/$RECORD_ID" \
  -H "Authorization: Bearer $CF_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "A",
    "name": "api.example.com",
    "content": "192.0.2.2",
    "ttl": 1,
    "proxied": true
  }'
```

## DNS Audit

Check for:
- Records pointing to decommissioned IPs
- Unproxied records that should be proxied
- Missing SPF/DKIM/DMARC records for email
- Dangling CNAME records
- TTL inconsistencies

## Output Format

| Type | Name | Content | Proxied | TTL |
|------|------|---------|---------|-----|
| A | api.example.com | 192.0.2.1 | Yes | Auto |
| CNAME | www | example.com | Yes | Auto |
| MX | example.com | mail.example.com | No | 3600 |
| TXT | example.com | v=spf1 ... | No | 3600 |
```

---

## Skill: Security Configuration

### `.claude/skills/cf-security/SKILL.md`

```yaml
---
name: cf-security
description: Configure Cloudflare security features - WAF, firewall rules, rate limiting, Zero Trust
allowed-tools:
  - Bash
  - Read
  - Write
  - mcp__cloudflare__*
---
```

```markdown
# Cloudflare Security Skill

## WAF Configuration

### Custom Rules
```json
{
  "description": "Block suspicious user agents",
  "expression": "(http.user_agent contains \"sqlmap\") or (http.user_agent contains \"nikto\") or (http.user_agent contains \"scanner\")",
  "action": "block"
}
```

### Rate Limiting
```json
{
  "description": "API rate limit - 100 requests per minute",
  "expression": "http.request.uri.path matches \"^/api/\"",
  "action": "block",
  "ratelimit": {
    "characteristics": ["ip.src"],
    "period": 60,
    "requests_per_period": 100,
    "mitigation_timeout": 600
  }
}
```

### Bot Management
```json
{
  "description": "Challenge likely bots on sensitive endpoints",
  "expression": "(cf.bot_management.score lt 30) and (http.request.uri.path matches \"^/(login|signup|checkout)\")",
  "action": "managed_challenge"
}
```

## Security Audit Checklist

- [ ] SSL/TLS mode set to "Full (Strict)"
- [ ] HSTS enabled with long max-age
- [ ] Minimum TLS version set to 1.2
- [ ] Always Use HTTPS enabled
- [ ] WAF managed rules enabled
- [ ] Rate limiting on API endpoints
- [ ] Bot management configured
- [ ] DDoS protection settings reviewed
- [ ] Page Shield enabled (if applicable)
- [ ] Email security (SPF, DKIM, DMARC) configured

## Zero Trust Configuration

### Access Application
```json
{
  "name": "Internal Dashboard",
  "domain": "dashboard.example.com",
  "type": "self_hosted",
  "policies": [
    {
      "name": "Allow Engineering",
      "decision": "allow",
      "include": [
        {"email_domain": {"domain": "company.com"}},
        {"group": {"id": "engineering-group-id"}}
      ],
      "require": [
        {"login_method": {"id": "google-oauth-id"}}
      ]
    }
  ]
}
```
```

---

## Skill: Cache Management

### `.claude/skills/cf-cache/SKILL.md`

```yaml
---
name: cf-cache
description: Manage Cloudflare cache - purge, configure rules, analyze hit rates
allowed-tools:
  - Bash
  - mcp__cloudflare__*
---
```

```markdown
# Cache Management Skill

## Operations

### Purge Cache
```bash
# Purge everything
curl -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/purge_cache" \
  -H "Authorization: Bearer $CF_API_TOKEN" \
  -d '{"purge_everything": true}'

# Purge specific URLs
curl -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/purge_cache" \
  -H "Authorization: Bearer $CF_API_TOKEN" \
  -d '{"files": ["https://example.com/api/config", "https://example.com/static/app.js"]}'

# Purge by tag
curl -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/purge_cache" \
  -H "Authorization: Bearer $CF_API_TOKEN" \
  -d '{"tags": ["product-page", "v2.3.1"]}'
```

### Cache Rules
```json
{
  "description": "Cache API responses for 5 minutes",
  "expression": "http.request.uri.path matches \"^/api/public/\"",
  "action": "set_cache_settings",
  "action_parameters": {
    "browser_ttl": { "mode": "override_origin", "default": 60 },
    "edge_ttl": { "mode": "override_origin", "default": 300 },
    "cache": true
  }
}
```

## Cache Analysis

Monitor cache performance:
- Hit ratio (target: >90% for static assets)
- Bandwidth saved
- Origin load reduction
- Cache eviction rates
```

---

## Skill: Pages Deployment

### `.claude/skills/cf-pages/SKILL.md`

```yaml
---
name: cf-pages
description: Deploy and manage Cloudflare Pages projects
allowed-tools:
  - Bash
  - Read
  - Write
  - mcp__cloudflare__*
---
```

```markdown
# Cloudflare Pages Skill

## Deploy a Static Site

```bash
# Direct upload
npx wrangler pages deploy ./dist --project-name my-site

# Connect to GitHub (one-time setup)
npx wrangler pages project create my-site --production-branch main
```

## Build Configuration

| Framework | Build Command | Output Dir |
|-----------|--------------|------------|
| Next.js | `npm run build` | `.next` |
| React (CRA) | `npm run build` | `build` |
| Vue | `npm run build` | `dist` |
| Astro | `npm run build` | `dist` |
| SvelteKit | `npm run build` | `.svelte-kit/cloudflare` |

## Pages Functions

Add serverless functions to Pages:

```typescript
// functions/api/hello.ts
export const onRequestGet: PagesFunction = async (context) => {
  return Response.json({ message: "Hello from Pages Functions!" });
};
```

## Environment Variables

```bash
# Set production variable
npx wrangler pages secret put API_KEY --project-name my-site

# Set preview variable
npx wrangler pages secret put API_KEY --project-name my-site --env preview
```
```
