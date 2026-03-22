# Example Agents: Coordinating Between AI Tools

Agents are autonomous or semi-autonomous processes that coordinate work between multiple AI coding tools.

## Agent 1: The Review Board

A multi-model review agent that gets opinions from three AI tools before approving changes.

### Shell Script Implementation

```bash
#!/bin/bash
# agents/review-board.sh
# Usage: ./review-board.sh [commit-range]

RANGE="${1:-HEAD~1..HEAD}"
DIFF=$(git diff "$RANGE")

if [ -z "$DIFF" ]; then
    echo "No changes in range $RANGE"
    exit 0
fi

REVIEW_DIR="/tmp/review-board-$(date +%s)"
mkdir -p "$REVIEW_DIR"

echo "$DIFF" > "$REVIEW_DIR/diff.txt"

PROMPT="You are a code reviewer. Review this diff for:
1. Bugs and logic errors
2. Security vulnerabilities
3. Performance issues
4. Code style and maintainability
Be specific. Reference line numbers.

Diff:
$(cat "$REVIEW_DIR/diff.txt")"

echo "=== Requesting Claude Code review... ==="
echo "$PROMPT" | claude --print 2>/dev/null > "$REVIEW_DIR/claude.txt" &
PID_CLAUDE=$!

echo "=== Requesting Gemini review (free)... ==="
echo "$PROMPT" | gemini 2>/dev/null > "$REVIEW_DIR/gemini.txt" &
PID_GEMINI=$!

echo "=== Requesting Codex review... ==="
codex "$PROMPT" 2>/dev/null > "$REVIEW_DIR/codex.txt" &
PID_CODEX=$!

wait $PID_CLAUDE $PID_GEMINI $PID_CODEX

echo ""
echo "========================================="
echo "  REVIEW BOARD RESULTS"
echo "========================================="
echo ""
echo "--- Claude Code ---"
cat "$REVIEW_DIR/claude.txt"
echo ""
echo "--- Gemini CLI ---"
cat "$REVIEW_DIR/gemini.txt"
echo ""
echo "--- Codex CLI ---"
cat "$REVIEW_DIR/codex.txt"
echo ""
echo "Reviews saved to $REVIEW_DIR/"
```

### Claude Code Skill Wrapper

`.claude/skills/review-board/SKILL.md`:

```yaml
---
description: Run a multi-AI review board on current changes
disable-model-invocation: true
---
```

```markdown
# Review Board Agent

1. Run `bash agents/review-board.sh` to get reviews from Claude, Gemini, and Codex
2. Read the output files from /tmp/review-board-*/
3. Synthesize the three reviews into a single report:
   - Unanimous concerns (act on these immediately)
   - Majority concerns (likely worth fixing)
   - Single-tool concerns (evaluate case by case)
4. Propose fixes for all unanimous and majority concerns
```

## Agent 2: The Research-Implement-Verify Loop

An agent that uses Gemini for research, Claude for implementation, and Codex for verification.

### Shell Script

```bash
#!/bin/bash
# agents/research-implement-verify.sh
# Usage: ./research-implement-verify.sh "Add OAuth2 with PKCE to the Express app"

TASK="$1"
WORKSPACE="/tmp/riv-$(date +%s)"
mkdir -p "$WORKSPACE"

echo "=== Phase 1: Research (Gemini -- free) ==="
echo "Research current best practices and implementation patterns for: $TASK
Include specific library recommendations, code patterns, and security considerations.
Write a detailed implementation guide." | gemini > "$WORKSPACE/research.md"

echo "Research complete. See $WORKSPACE/research.md"

echo "=== Phase 2: Plan (Claude Code) ==="
echo "Based on this research, create a step-by-step implementation plan for our codebase:

Research findings:
$(cat "$WORKSPACE/research.md")

Task: $TASK

Write a detailed plan with specific file changes needed." | claude --print > "$WORKSPACE/plan.md"

echo "Plan complete. See $WORKSPACE/plan.md"

echo "=== Phase 3: Implement (Codex -- full-auto) ==="
cp "$WORKSPACE/plan.md" ./.implementation-plan.md
codex --full-auto "Read .implementation-plan.md and implement all changes described. Make atomic commits for each logical change."
rm ./.implementation-plan.md

echo "=== Phase 4: Verify (Claude Code) ==="
echo "Review all recent git commits. Verify the implementation against:
1. The original task: $TASK
2. The research findings in $WORKSPACE/research.md
3. The plan in $WORKSPACE/plan.md

Report any gaps, issues, or improvements needed." | claude --print > "$WORKSPACE/verification.md"

echo "Verification complete. See $WORKSPACE/verification.md"
echo "All artifacts saved to $WORKSPACE/"
```

## Agent 3: The Continuous Improvement Agent

Runs in the background, periodically analyzing the codebase with free tools.

```bash
#!/bin/bash
# agents/continuous-improvement.sh
# Run in background: nohup ./agents/continuous-improvement.sh &

INTERVAL=3600  # Run every hour
LOG_DIR="./agents/logs"
mkdir -p "$LOG_DIR"

while true; do
    TIMESTAMP=$(date +%Y%m%d_%H%M%S)
    LOG="$LOG_DIR/improvement_$TIMESTAMP.md"

    echo "# Improvement Scan: $TIMESTAMP" > "$LOG"

    # Use Gemini (free) for analysis
    echo "Analyze this codebase for:
1. Code duplication that could be refactored
2. Outdated dependencies
3. Missing error handling
4. Test coverage gaps
Be specific with file paths and line numbers." | gemini >> "$LOG" 2>/dev/null

    echo "Scan complete: $LOG"
    sleep $INTERVAL
done
```

## Agent 4: The Multi-Agent Task Decomposer

Breaks a large task into subtasks and assigns each to the best tool.

### Claude Code Skill

`.claude/skills/task-decomposer/SKILL.md`:

```yaml
---
description: Decompose a large task and route subtasks to the best AI tool
disable-model-invocation: true
---
```

```markdown
# Task Decomposer Agent

When invoked with a complex task:

1. Analyze the task and break it into subtasks
2. For each subtask, determine the best tool:
   - **Research/docs** --> Gemini CLI (free, has Google Search)
   - **Complex logic/architecture** --> Claude Code (you, handle directly)
   - **Autonomous implementation** --> Codex CLI (full-auto mode)
   - **Quick fixes with commits** --> Aider
   - **Private/sensitive code** --> OpenCode + Ollama (local)

3. Create a task file:
   ```
   ## Task: <original task>

   ### Subtask 1: <description> [Tool: Gemini]
   Status: pending

   ### Subtask 2: <description> [Tool: Claude]
   Status: pending
   ```

4. Execute subtasks in dependency order:
   - Run Gemini tasks first (research informs implementation)
   - Handle Claude tasks yourself
   - Delegate Codex tasks via: `codex --full-auto "<subtask>"`
   - Track status in the task file

5. After all subtasks complete, verify the integrated result
```

## Agent 5: The Cross-Agent Memory Coordinator

Uses Memorix to maintain shared context across all tools.

### Setup

```bash
# Install Memorix
npm install -g memorix

# Add to Claude Code (.mcp.json in project root)
cat > .mcp.json << 'EOF'
{
  "mcpServers": {
    "memorix": {
      "command": "npx",
      "args": ["-y", "memorix"],
      "env": {}
    }
  }
}
EOF

# Add to Gemini CLI (~/.gemini/settings.json)
# Add the same Memorix MCP configuration
```

### Agent Behavior

```markdown
# Memory Coordinator

At the start of every session:
1. Retrieve all memories for this workspace from Memorix
2. Surface key decisions, gotchas, and context

During the session:
3. After every significant decision, store it:
   - Architecture choices
   - Library selections
   - Security decisions
   - Known issues and workarounds

At session end:
4. Summarize what was accomplished
5. Store the summary as a session memory
6. Any tool (Claude Code, Gemini CLI, Codex CLI) that starts
   a new session will have access to this context
```

## Agent 6: Claude Code Agent Teams with External Specialists

Leverage Claude Code's built-in Agent Teams alongside external tools.

```bash
# Start tmux (required for split-pane agent teams)
tmux new-session -s dev-team

# In the main pane, start Claude Code
claude

# Inside Claude Code, configure agent teams:
# > /agent-teams start

# Claude Code spawns teammate subagents in split panes.
# Meanwhile, in another tmux window:
tmux split-window -h
gemini  # Free Gemini CLI for research support

# The workflow:
# - Claude Code main agent: orchestrator
# - Claude Code teammates: implementation specialists
# - Gemini CLI (separate pane): free research and second opinions
```

## Directory Structure for Agents

```
project-root/
  agents/
    review-board.sh
    research-implement-verify.sh
    continuous-improvement.sh
    logs/
  .claude/
    skills/
      review-board/SKILL.md
      task-decomposer/SKILL.md
  .mcp.json           # Memorix and other MCP servers
```

## Notes

- All Gemini-based agent components are free (1,000 req/day)
- Codex-based components require ChatGPT subscription or API key
- Aider is free software but LLM API calls cost money (unless using Ollama)
- Memorix is fully local and free -- no API keys, no cloud
- Claude Code Agent Teams requires tmux or iTerm2 for split-pane mode
- Shell script agents can be run from cron, CI/CD, or manually

## Sources

- [Claude Code Agent Teams](https://code.claude.com/docs/en/agent-teams)
- [Claude Code Bridge](https://github.com/bfly123/claude_code_bridge)
- [Memorix](https://github.com/AVIDS2/memorix)
- [PAL MCP Server](https://github.com/BeehiveInnovations/pal-mcp-server)
- [MCP Agent Bridge](https://github.com/vizi2000/mcp-agent-bridge)
