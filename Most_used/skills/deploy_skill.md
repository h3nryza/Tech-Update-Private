# Deploy Skill -- Deployment Orchestration

> Orchestrates build, test, and deployment pipelines with safety gates. Intentionally requires explicit user invocation to prevent accidental deployments.

---

## Why This Skill Exists

Deployment is a high-stakes operation that should never happen accidentally. This skill provides a structured deployment checklist, ensures all safety gates pass before pushing to production, and uses `disable-model-invocation: true` to prevent Claude from triggering it autonomously.

---

## SKILL.md Configuration

```yaml
---
name: deploy
description: >
  Orchestrates the full deployment pipeline: test, build, deploy, verify.
  Only triggered by explicit user request -- never auto-invoked.
disable-model-invocation: true
---

# Deploy Skill

## IMPORTANT: This skill requires explicit user invocation.
Claude must NEVER trigger a deployment autonomously.

## Pre-Deploy Checklist

Before deploying, ALL of these must pass:

1. [ ] All tests pass (`npm test` / `pytest` / `go test ./...`)
2. [ ] Linter passes with no errors
3. [ ] Build completes successfully
4. [ ] No uncommitted changes in working directory
5. [ ] Current branch is up to date with remote
6. [ ] Environment variables are configured
7. [ ] Database migrations are ready (if applicable)
8. [ ] No critical security vulnerabilities in dependencies
9. [ ] User has confirmed the deployment target

## Deployment Targets

### Staging
```bash
# Typical staging deployment flow
git checkout staging
git merge main
npm run build
npm run deploy:staging
# Verify
curl -s https://staging.example.com/health | jq .
```

### Production
```bash
# Typical production deployment flow
# 1. Tag the release
git tag -a v$(cat package.json | jq -r .version) -m "Release $(cat package.json | jq -r .version)"
git push origin --tags

# 2. Build
npm run build

# 3. Deploy
npm run deploy:production

# 4. Verify
curl -s https://api.example.com/health | jq .

# 5. Monitor (first 5 minutes)
# Check error rates, response times, key metrics
```

## Platform-Specific Patterns

### Vercel
```bash
# Preview deployment
vercel

# Production deployment
vercel --prod

# Check deployment status
vercel ls --limit 5
```

### AWS (CDK / CloudFormation)
```bash
# Diff changes
cdk diff

# Deploy
cdk deploy --require-approval broadening

# Verify
aws cloudformation describe-stacks --stack-name MyStack --query 'Stacks[0].StackStatus'
```

### Docker / Kubernetes
```bash
# Build image
docker build -t myapp:$(git rev-parse --short HEAD) .

# Push to registry
docker push registry.example.com/myapp:$(git rev-parse --short HEAD)

# Deploy to k8s
kubectl set image deployment/myapp myapp=registry.example.com/myapp:$(git rev-parse --short HEAD)

# Verify rollout
kubectl rollout status deployment/myapp --timeout=300s
```

### GitHub Actions (trigger workflow)
```bash
# Trigger deployment workflow
gh workflow run deploy.yml --ref main

# Monitor
gh run list --workflow=deploy.yml --limit 3
gh run watch
```

### Cloudflare Workers / Pages
```bash
# Deploy worker
wrangler deploy

# Deploy pages
wrangler pages deploy ./dist
```

## Rollback Procedure

If deployment fails or causes issues:

```bash
# Kubernetes
kubectl rollout undo deployment/myapp

# Vercel
vercel rollback

# AWS CDK
cdk deploy --previous

# Docker
docker tag myapp:previous myapp:latest
docker push registry.example.com/myapp:latest

# Git-based (revert and redeploy)
git revert HEAD
git push origin main
```

## Post-Deploy Verification

After every deployment, check:

1. [ ] Health endpoint returns 200
2. [ ] Key user flows work (smoke test)
3. [ ] Error rates are not elevated
4. [ ] Response times are within normal range
5. [ ] Logs show no unexpected errors

## Output Format

```markdown
### Deployment Report

**Target**: [staging/production]
**Version**: [tag or commit hash]
**Timestamp**: [ISO 8601]

#### Pre-Deploy Checks
- Tests: PASS (X/Y)
- Lint: PASS
- Build: PASS
- Clean working dir: YES

#### Deployment
- Status: SUCCESS / FAILED
- Duration: Xs
- Method: [vercel/k8s/cdk/etc.]

#### Post-Deploy Verification
- Health check: PASS
- Smoke test: PASS
- Error rate: X% (normal: Y%)

#### Rollback Required: NO / YES (reason)
```
```

---

## CLAUDE.md Integration

```markdown
## Deployment Rules

- NEVER deploy without explicit user confirmation
- All tests and lint must pass before any deployment
- Always verify health endpoint after deployment
- Log every deployment in the changelog with version, target, and status
- If deployment fails, execute rollback procedure immediately
- Production deploys require a git tag
```

---

## Hook Integration

Block accidental deployments with a PreToolUse hook:

```json
// .claude/settings.json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "command": "if echo '$ARGUMENTS' | jq -r '.command' | grep -qiE '(deploy|wrangler deploy|cdk deploy|kubectl apply|vercel --prod)'; then echo 'BLOCKED: Deployment commands require explicit user approval. Use the deploy skill.' >&2; exit 2; fi"
      }
    ]
  }
}
```

---

## Sources

- [Claude Code Skills Documentation](https://code.claude.com/docs/en/skills) -- `disable-model-invocation` flag
- [Claude Code Hooks Guide](https://code.claude.com/docs/en/hooks-guide) -- PreToolUse blocking
- [levnikolaevich/claude-code-skills](https://github.com/levnikolaevich/claude-code-skills) -- Full delivery lifecycle plugins
