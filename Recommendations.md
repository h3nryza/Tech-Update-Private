# Recommendations

> Living document — updated as research progresses.

---

## Top-Level Recommendations

### 1. Start Simple, Evolve the Framework
Don't try to set up everything at once. The most successful vibe coding practitioners start with:
1. A solid CLAUDE.md file
2. 2-3 core skills (commit, review, test)
3. 1-2 MCP servers (GitHub + one platform)
Then iterate based on actual usage patterns.

### 2. CLAUDE.md is Your Most Important File
This is the single highest-leverage thing you can configure. A well-crafted CLAUDE.md transforms Claude Code from a generic assistant into a domain expert. Invest time here first.

### 3. Don't Over-Engineer Skills and Agents
The community consensus is clear: fewer, well-tested skills beat a library of untested ones. Start with the built-in slash commands (`/commit`, `/review-pr`) and only create custom ones when you hit a repeated workflow.

### 4. MCP Servers: Quality Over Quantity
Most productive setups use 3-5 MCP servers, not 20. The most common stack:
- GitHub (built-in or via MCP)
- Slack (for ChatOps)
- One cloud provider (AWS/GCP/Azure)
- One monitoring tool (Datadog/Grafana)
- One project management tool (Jira/Linear/Notion)

### 5. Repository Structure: Framework in the Repo
The emerging best practice is to keep your Claude Code configuration IN the project repo (`.claude/` directory), not as a separate library. This ensures:
- Context travels with the code
- Different projects can have different configs
- Team members get the config automatically

For SHARED skills/agents that work across projects, use `~/.claude/` (user-level config).

### 6. Remote Access: SSH + tmux is King
The most reliable way to use Claude Code remotely:
```
ssh your-server -t "tmux attach -t claude || tmux new -s claude"
```
Then use any SSH client on your phone. Claude Code sessions persist across disconnects.

### 7. Multi-Tool Approach
Don't be dogmatic about one AI tool. The best practitioners use:
- Claude Code for complex reasoning, architecture, multi-file changes
- Copilot for inline completions while typing
- Codex/Gemini CLI for quick one-off questions or second opinions

### 8. ChatOps is the Future of Vibe Ops
The most mature vibe ops setups route everything through Slack:
- Alerts trigger AI analysis automatically
- Incident channels get AI-powered runbooks
- Post-mortems are drafted by AI, reviewed by humans

---

## Framework-Specific Recommendations

### For This Repo
1. **Keep the Prime Directive** — it's solid logging/governance
2. **Add a `/setup` slash command** — one command to bootstrap new users
3. **Create platform-specific CLAUDE.md includes** — so you can mix and match
4. **Build the AI SRE as a separate agent** — it should be composable

### For Your Workflow
1. **Use tmux + SSH** for phone access immediately
2. **Set up GitHub MCP server** first — highest ROI
3. **Create a `/incident` slash command** — ties together all platforms
4. **Log everything** — your Prime Directive already does this well

---

*Last updated: 2026-03-22T00:05:00Z*
