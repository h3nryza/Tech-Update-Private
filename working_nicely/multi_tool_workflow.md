# Multi-Tool AI Coding Workflows

Practical workflows for combining multiple AI coding tools from the terminal.

## Workflow 1: The Free Stack

**Cost: $0 (or near-zero)**

Tools: Gemini CLI (free) + Aider with Ollama (free) + Memorix (free)

```bash
# Setup
npx @google/gemini-cli          # Login with Google account
pip install aider-chat           # Install Aider
brew install ollama              # Install Ollama
ollama pull deepseek-coder-v2   # Pull a free local model
npm install -g memorix           # Shared memory

# Workflow
# 1. Gemini researches and plans (free, 1000 req/day)
gemini "Analyze this codebase structure and create an implementation plan for adding WebSocket support. Write to PLAN.md"

# 2. Aider implements using local model (free, runs on your hardware)
aider --model ollama/deepseek-coder-v2 --message "Read PLAN.md and implement the WebSocket server"

# 3. Gemini reviews (free)
gemini "Review the git log for recent commits. Check the WebSocket implementation for correctness and security"
```

## Workflow 2: The Power Stack

**Cost: Claude subscription + ChatGPT subscription**

Tools: Claude Code + Codex CLI + Gemini CLI (free)

```bash
# Setup: All three installed, authenticated
# Claude Code: Anthropic subscription
# Codex CLI: ChatGPT subscription
# Gemini CLI: Free Google account

# Architecture phase (Claude -- best at reasoning)
claude "Design a microservices architecture for this monolith. Document in ARCHITECTURE.md with mermaid diagrams"

# Implementation phase (Codex -- full-auto mode)
codex --full-auto "Read ARCHITECTURE.md and implement the user-service microservice"

# Research phase (Gemini -- free, has Google Search)
gemini "Search for current best practices for gRPC inter-service communication in Node.js and document findings"

# Review phase (Claude -- thorough reasoning)
claude "Review all changes since the last tagged release. Identify issues and suggest improvements"
```

## Workflow 3: The Review Pipeline

Use multiple tools to review the same code for higher confidence:

```bash
#!/bin/bash
# review-pipeline.sh -- Multi-AI code review

DIFF=$(git diff HEAD~1)
echo "$DIFF" > /tmp/review_diff.txt

echo "=== Claude Code Review ==="
claude "Review this git diff for bugs, security issues, and code quality: $(cat /tmp/review_diff.txt)" | tee /tmp/review_claude.txt

echo "=== Gemini Review (free) ==="
gemini "Review this git diff for bugs, security issues, and code quality: $(cat /tmp/review_diff.txt)" | tee /tmp/review_gemini.txt

echo "=== Codex Review ==="
codex "Review this git diff for bugs, security issues, and code quality: $(cat /tmp/review_diff.txt)" | tee /tmp/review_codex.txt

echo "Reviews written to /tmp/review_*.txt"
```

## Workflow 4: Specialist Routing

Route tasks to the tool best suited for them:

```bash
# Complex refactoring --> Claude Code (best reasoning)
claude "Refactor the event system to use an event bus pattern with proper decoupling"

# Test generation --> Codex CLI (good at autonomous execution)
codex --full-auto "Generate comprehensive unit tests for all files in src/services/"

# Documentation with research --> Gemini CLI (free + Google Search)
gemini "Research current API documentation best practices and generate OpenAPI docs for our REST endpoints"

# Quick fixes with git integration --> Aider (auto-commits)
aider --message "Fix the TypeScript type errors in src/types.ts"
```

## Workflow 5: The tmux Multi-Pane Setup

Run all tools simultaneously in a tmux session:

```bash
#!/bin/bash
# multi-ai-session.sh

# Create tmux session with named panes
tmux new-session -d -s ai -n main

# Pane 0: Claude Code (top-left)
tmux send-keys -t ai:main "claude" Enter

# Pane 1: Gemini CLI (top-right)
tmux split-window -h -t ai:main
tmux send-keys -t ai:main.1 "gemini" Enter

# Pane 2: Codex CLI (bottom-left)
tmux split-window -v -t ai:main.0
tmux send-keys -t ai:main.2 "codex" Enter

# Pane 3: Your shell (bottom-right)
tmux split-window -v -t ai:main.1

# Attach to the session
tmux attach -t ai
```

## Workflow 6: Claude Code Agent Teams + External Tools

Claude Code's built-in Agent Teams feature can coordinate with external tools:

```bash
# Start a tmux session (required for split-pane agent teams)
tmux new-session -s dev

# In Claude Code, use agent teams for internal coordination
claude

# Then inside Claude Code:
# > /agent-teams start --teammates 3
# > Teammate 1: Implement the API endpoints
# > Teammate 2: Write tests for the endpoints
# > Teammate 3: Update the documentation

# In a separate pane, use Gemini for free research
# tmux split-window
gemini "Search for security best practices for the authentication patterns we're implementing"
```

## Workflow 7: The Aider + Claude Code Combo

Aider's git-native workflow complements Claude Code:

```bash
# Aider for rapid iteration with auto-commits
aider --model claude-3.7-sonnet --message "Add input validation to UserController"
# Aider automatically commits the change with a descriptive message

# Claude Code for reviewing the accumulated changes
claude "Review the last 5 git commits made by aider. Are there any issues or inconsistencies?"

# Aider for fixing issues Claude found
aider --message "Fix the validation issues Claude identified: missing email format check and XSS sanitization"
```

## Workflow 8: Cross-Tool Shared Memory with Memorix

```bash
# Install Memorix
npm install -g memorix

# Add to Claude Code MCP config (~/.claude/settings.json)
# Add to Gemini CLI MCP config (~/.gemini/settings.json)
# Both point to the same Memorix instance

# In Claude Code session:
# "Store a memory: We decided to use JWT with 15-min expiry and refresh tokens"

# Later, in Gemini CLI session:
# "What authentication approach did we decide on?"
# --> Gemini retrieves the decision stored by Claude Code

# All memory stored locally in ~/.memorix/data/
# No API keys, no cloud, no cost
```

## Decision Matrix: Which Tool for What

| Task | Best Tool | Why | Cost |
|------|-----------|-----|------|
| Architecture design | Claude Code | Deepest reasoning | Subscription |
| Large codebase analysis | Gemini CLI | 1M context, free | Free |
| Autonomous implementation | Codex CLI | Full-auto mode | Subscription |
| Quick fixes with git | Aider | Auto-commit workflow | API costs |
| Research current docs | Gemini CLI | Google Search grounding | Free |
| Security review | Claude Code | Thorough analysis | Subscription |
| Test generation | Codex CLI | Good at autonomous tasks | Subscription |
| Multi-file refactoring | Claude Code | Best at complex changes | Subscription |
| Second opinion (free) | Gemini CLI | High quality, zero cost | Free |
| Local/private code | OpenCode + Ollama | Fully local, no data leaves machine | Free |

## Sources

- [Claude Code Agent Teams](https://code.claude.com/docs/en/agent-teams)
- [Codex CLI Features](https://developers.openai.com/codex/cli/features)
- [Gemini CLI GitHub](https://github.com/google-gemini/gemini-cli)
- [Aider](https://aider.chat/)
- [OpenCode](https://opencode.ai/)
- [Memorix](https://github.com/AVIDS2/memorix)
- [Top 5 Terminal AI Coding Agents in 2026](https://dev.to/nebulagg/top-5-terminal-ai-coding-agents-in-2026-272)
- [Codex-Workflow GitHub](https://github.com/manastm/codex-workflow)
