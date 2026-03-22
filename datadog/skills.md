# Datadog Monitoring Skills for Claude Code

---

## Skill: Monitor Health Check

### `.claude/skills/dd-health/SKILL.md`

```yaml
---
name: dd-health
description: Check overall system health from Datadog - monitors, alerts, service status
allowed-tools:
  - Bash
  - Read
  - mcp__datadog__*
---
```

```markdown
# Datadog Health Check Skill

## Workflow

1. Query all monitors in ALERT or WARN state
2. Check critical service SLOs
3. Review recent events for anomalies
4. Summarize overall health status

## Health Check Queries

### Active Alerts
```
GET /api/v1/monitor?monitor_tags=env:production&states=Alert,Warn
```

### Service SLOs
```
GET /api/v1/slo?tags=env:production
```

### Recent Error Events
```
GET /api/v1/events?start=<1h_ago>&end=<now>&tags=status:error
```

## Output Format

```
## System Health: [HEALTHY|DEGRADED|CRITICAL]

### Active Alerts (N)
| Monitor | Status | Duration | Service |
|---------|--------|----------|---------|

### SLO Status
| SLO | Target | Current | Budget Remaining |
|-----|--------|---------|-----------------|

### Recent Events
- [timestamp] event description
```
```

---

## Skill: Log Investigation

### `.claude/skills/dd-logs/SKILL.md`

```yaml
---
name: dd-logs
description: Search and analyze Datadog logs for debugging and investigation
allowed-tools:
  - Bash
  - Read
  - mcp__datadog__*
---
```

```markdown
# Log Investigation Skill

## Query Syntax

Datadog log queries use a specific syntax:

```
service:my-app status:error @http.status_code:500
```

### Common Patterns

| Pattern | Query |
|---------|-------|
| Errors in a service | `service:api-gateway status:error` |
| Specific HTTP errors | `@http.status_code:>=500` |
| By trace ID | `@trace_id:abc123` |
| Text search | `*timeout*` |
| Combined | `service:auth status:error @http.method:POST` |

## Investigation Workflow

1. Start broad: query errors in the affected service
2. Identify patterns: group by error type, endpoint, user
3. Trace individual requests: follow trace_id across services
4. Correlate with metrics: check if error spike aligns with metric changes
5. Identify root cause: deployment, config change, dependency failure

## Time Range Shortcuts

| Range | API Parameter |
|-------|--------------|
| Last 15 min | `from=now-15m&to=now` |
| Last 1 hour | `from=now-1h&to=now` |
| Last 4 hours | `from=now-4h&to=now` |
| Last 24 hours | `from=now-24h&to=now` |
| Custom | `from=2026-03-22T10:00:00Z&to=2026-03-22T14:00:00Z` |

## Output Format

Present log findings as:
1. **Pattern**: What error pattern was found
2. **Volume**: How many occurrences
3. **Timing**: When it started, trending up or down
4. **Sample**: 3-5 representative log entries
5. **Correlation**: Related metrics or events
```

---

## Skill: Metric Analysis

### `.claude/skills/dd-metrics/SKILL.md`

```yaml
---
name: dd-metrics
description: Query and analyze Datadog metrics - time series, anomalies, forecasts
allowed-tools:
  - Bash
  - Read
  - mcp__datadog__*
---
```

```markdown
# Metric Analysis Skill

## Common Metric Queries

### Application Metrics
```
avg:trace.http.request.duration{service:api-gateway,env:production} by {resource_name}
sum:trace.http.request.errors{service:api-gateway,env:production}.as_count()
```

### Infrastructure Metrics
```
avg:system.cpu.user{env:production} by {host}
avg:system.mem.used{env:production} / avg:system.mem.total{env:production} * 100
```

### Custom Business Metrics
```
sum:orders.completed{env:production}.as_count()
avg:checkout.duration{env:production}
```

## Analysis Techniques

### Anomaly Detection
Compare current values against historical baselines:
- Is the current value outside 2 standard deviations?
- Has the trend changed direction?
- Are there periodic patterns being broken?

### Correlation
When investigating an issue, check:
1. Did a deployment happen? (events overlay)
2. Did traffic pattern change? (request volume)
3. Did a dependency degrade? (downstream latency)
4. Did resource limits hit? (CPU, memory, connections)

## Output Format

Present metric analysis as:
1. **Current State**: Latest values and trend direction
2. **Comparison**: vs 1h ago, 24h ago, 7d ago
3. **Anomalies**: Values outside normal range
4. **Correlations**: Related metrics that moved together
5. **Recommendation**: What to investigate or action to take
```

---

## Skill: Dashboard Builder

### `.claude/skills/dd-dashboard/SKILL.md`

```yaml
---
name: dd-dashboard
description: Create and manage Datadog dashboards programmatically
allowed-tools:
  - Bash
  - Read
  - Write
  - mcp__datadog__*
---
```

```markdown
# Dashboard Builder Skill

## Dashboard Types

### Service Overview Dashboard
Standard dashboard for any microservice:
- Request rate, error rate, latency (RED metrics)
- Resource utilization (CPU, memory)
- Dependencies health
- Deployment markers

### SRE Dashboard
- SLO burn rate
- Error budget remaining
- Alert volume over time
- MTTR trend

### Business Dashboard
- Key business metrics (orders, revenue, users)
- Conversion funnel
- Feature adoption rates

## Widget Library

### Timeseries
```json
{
  "type": "timeseries",
  "requests": [{
    "q": "avg:trace.http.request.duration{service:api} by {resource_name}",
    "display_type": "line"
  }],
  "title": "Request Latency by Endpoint"
}
```

### Query Value (Single Stat)
```json
{
  "type": "query_value",
  "requests": [{
    "q": "sum:trace.http.request.errors{service:api}.as_count()",
    "aggregator": "sum"
  }],
  "title": "Total Errors (last 1h)",
  "precision": 0
}
```

### Top List
```json
{
  "type": "toplist",
  "requests": [{
    "q": "top(avg:trace.http.request.duration{service:api} by {resource_name}, 10, 'mean', 'desc')"
  }],
  "title": "Slowest Endpoints"
}
```

### Heatmap
```json
{
  "type": "heatmap",
  "requests": [{
    "q": "avg:trace.http.request.duration{service:api}"
  }],
  "title": "Latency Distribution"
}
```

## Standard Template

When creating a service dashboard, always include:
1. Header group: Service name, environment, links
2. RED metrics group: Request rate, Error rate, Duration
3. Infrastructure group: CPU, Memory, Disk, Network
4. Dependencies group: Downstream service latency/errors
5. Deployments: Event overlay with deploy markers
```

---

## Skill: Monitor Management

### `.claude/skills/dd-monitors/SKILL.md`

```yaml
---
name: dd-monitors
description: Create, update, and manage Datadog monitors and alerts
allowed-tools:
  - Bash
  - Read
  - Write
  - mcp__datadog__*
---
```

```markdown
# Monitor Management Skill

## Monitor Types

| Type | Use Case | Example |
|------|----------|---------|
| Metric | Threshold on a metric | CPU > 80% for 5 min |
| Log | Log volume or pattern | > 100 errors/min |
| APM | Trace metric threshold | p99 latency > 2s |
| Composite | Combine multiple monitors | High CPU AND high errors |
| Anomaly | Deviation from normal | Traffic 3x below baseline |
| Forecast | Projected future value | Disk full in 48h |
| SLO | SLO burn rate alert | Error budget burning 10x |

## Standard Monitor Templates

### High Error Rate
```json
{
  "name": "[{{service}}] High Error Rate in {{env}}",
  "type": "metric alert",
  "query": "sum(last_5m):sum:trace.http.request.errors{service:{{service}},env:{{env}}}.as_count() / sum:trace.http.request.hits{service:{{service}},env:{{env}}}.as_count() * 100 > 5",
  "message": "Error rate is {{value}}% (threshold: 5%)\n\nService: {{service}}\nEnvironment: {{env}}\n\n@slack-incidents @pagerduty-sre",
  "options": {
    "thresholds": {"critical": 5, "warning": 2},
    "notify_no_data": false,
    "renotify_interval": 30
  },
  "tags": ["service:{{service}}", "env:{{env}}", "team:{{team}}"]
}
```

### High Latency
```json
{
  "name": "[{{service}}] High p99 Latency in {{env}}",
  "type": "metric alert",
  "query": "percentile(last_5m):p99:trace.http.request.duration{service:{{service}},env:{{env}}} > 2",
  "message": "p99 latency is {{value}}s (threshold: 2s)\n\n@slack-incidents",
  "options": {
    "thresholds": {"critical": 2, "warning": 1}
  }
}
```

### Anomaly Detection
```json
{
  "name": "[{{service}}] Traffic Anomaly in {{env}}",
  "type": "query alert",
  "query": "avg(last_1h):anomalies(sum:trace.http.request.hits{service:{{service}},env:{{env}}}.as_count(), 'agile', 3) >= 1",
  "message": "Traffic pattern is anomalous for {{service}}\n\n@slack-incidents"
}
```

## Notification Channels

Configure monitors to notify via:
- `@slack-channel-name` - Slack channel
- `@pagerduty-service` - PagerDuty
- `@opsgenie-team` - OpsGenie
- `@webhook-name` - Custom webhook
- `@email-address` - Email
```
