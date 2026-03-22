# Example Terminal Commands and Aliases

Practical shell configuration for working with multiple AI coding tools.

## Shell Aliases (add to ~/.zshrc or ~/.bashrc)

```bash
# ============================================================
# AI Tool Aliases
# ============================================================

# --- Claude Code ---
alias cc="claude"
alias ccp="claude --print"                    # Print mode (no interactive session)
alias ccr="claude 'Review the git diff for the last commit'"
alias cca="claude 'Analyze this codebase and summarize the architecture'"

# --- Gemini CLI (FREE - 1000 req/day) ---
alias gem="gemini"
alias gemr="gemini 'Review the git diff for the last commit'"
alias gems="gemini 'Search the web for'"      # Leverage Google Search grounding

# --- Codex CLI ---
alias cx="codex"
alias cxf="codex --full-auto"                 # Full autonomous mode
alias cxa="codex --approval-mode auto-edit"   # Auto-edit, confirm commands
alias cxs="codex --approval-mode suggest"     # Safest mode

# --- Aider ---
alias aid="aider"
alias aidl="aider --model ollama/deepseek-coder-v2"  # Free local model
alias aidc="aider --model claude-3.7-sonnet"          # Claude backend

# --- OpenCode ---
alias oc="opencode"
```

## Shell Functions

```bash
# ============================================================
# Multi-Tool Functions
# ============================================================

# Quick review from Gemini (free)
gemini-review() {
    local diff=$(git diff "${1:-HEAD~1..HEAD}")
    if [ -z "$diff" ]; then
        echo "No changes to review"
        return 1
    fi
    echo "Review this diff for bugs and security issues:\n$diff" | gemini
}

# Get a second opinion from Gemini on Claude's suggestion
second-opinion() {
    echo "$1" | gemini
}

# Research a topic using Gemini's Google Search (free)
ai-research() {
    gemini "Search the web and provide current information about: $*"
}

# Multi-model review: run Claude and Gemini in parallel
multi-review() {
    local range="${1:-HEAD~1..HEAD}"
    local diff=$(git diff "$range")
    local tmpdir=$(mktemp -d)

    echo "Running parallel reviews..."

    echo "Review this diff:\n$diff" | claude --print > "$tmpdir/claude.txt" 2>/dev/null &
    echo "Review this diff:\n$diff" | gemini > "$tmpdir/gemini.txt" 2>/dev/null &
    wait

    echo "=== Claude Code ==="
    cat "$tmpdir/claude.txt"
    echo ""
    echo "=== Gemini CLI (free) ==="
    cat "$tmpdir/gemini.txt"
    echo ""
    rm -rf "$tmpdir"
}

# Route to the right tool based on task type
ai() {
    case "$1" in
        research|search)
            shift
            gemini "Search the web for: $*"
            ;;
        implement|build)
            shift
            codex --auto-edit "$*"
            ;;
        review)
            shift
            claude "Review: $*"
            ;;
        fix)
            shift
            aider --message "$*"
            ;;
        test)
            shift
            codex --full-auto "Write tests for: $*"
            ;;
        analyze)
            shift
            claude "Analyze: $*"
            ;;
        *)
            claude "$*"
            ;;
    esac
}
# Usage:
#   ai research "latest Express.js security advisories"
#   ai implement "add rate limiting middleware"
#   ai review "the authentication module"
#   ai fix "TypeScript errors in src/types.ts"
#   ai test "src/services/auth.ts"
#   ai analyze "the database connection handling"
```

## tmux Session Scripts

### Multi-AI Development Session

```bash
# Save as ~/bin/ai-session
#!/bin/bash
# Start a tmux session with all AI tools ready

SESSION="ai-dev"

tmux has-session -t "$SESSION" 2>/dev/null
if [ $? -eq 0 ]; then
    echo "Session $SESSION already exists. Attaching..."
    tmux attach -t "$SESSION"
    exit 0
fi

# Create session with Claude Code
tmux new-session -d -s "$SESSION" -n "claude"
tmux send-keys -t "$SESSION:claude" "claude" Enter

# Add Gemini pane (split right)
tmux split-window -h -t "$SESSION:claude"
tmux send-keys -t "$SESSION:claude.1" "gemini" Enter

# Add Codex pane (split bottom-left)
tmux split-window -v -t "$SESSION:claude.0"
tmux send-keys -t "$SESSION:claude.2" "codex" Enter

# Add shell pane (split bottom-right)
tmux split-window -v -t "$SESSION:claude.1"
tmux send-keys -t "$SESSION:claude.3" "echo 'Shell ready. git status:' && git status" Enter

# Create a second window for monitoring
tmux new-window -t "$SESSION" -n "monitor"
tmux send-keys -t "$SESSION:monitor" "watch -n 5 'git log --oneline -10'" Enter

tmux select-window -t "$SESSION:claude"
tmux attach -t "$SESSION"
```

```bash
chmod +x ~/bin/ai-session
```

## Git Hooks with AI Tools

### Pre-commit: Free Gemini Review

```bash
# .git/hooks/pre-commit
#!/bin/bash
# Free AI review on every commit using Gemini CLI

DIFF=$(git diff --cached)
if [ -z "$DIFF" ]; then
    exit 0
fi

echo "Running free Gemini review on staged changes..."

RESULT=$(echo "Briefly review this diff for critical bugs or security issues only. If everything looks fine, just say 'LGTM'. If there are critical issues, describe them concisely.

$DIFF" | gemini 2>/dev/null)

if echo "$RESULT" | grep -qi "LGTM"; then
    echo "Gemini: LGTM"
    exit 0
else
    echo "Gemini flagged potential issues:"
    echo "$RESULT"
    echo ""
    read -p "Continue with commit? (y/n) " -n 1 -r
    echo
    if [[ $REPLY =~ ^[Yy]$ ]]; then
        exit 0
    else
        exit 1
    fi
fi
```

```bash
chmod +x .git/hooks/pre-commit
```

## One-Liner Commands

```bash
# Ask all three tools the same question and compare
ask-all() {
    local q="$*"
    echo "=== Claude ===" && echo "$q" | claude --print
    echo "=== Gemini ===" && echo "$q" | gemini
    echo "=== Codex ===" && codex "$q"
}

# Generate a commit message with Gemini (free)
ai-commit-msg() {
    local msg=$(git diff --cached | gemini "Write a concise git commit message for this diff. Only output the commit message, nothing else." 2>/dev/null)
    echo "Suggested: $msg"
    read -p "Use this message? (y/n/e for edit) " -n 1 -r
    echo
    case $REPLY in
        y|Y) git commit -m "$msg" ;;
        e|E) git commit -e -m "$msg" ;;
        *) echo "Aborted" ;;
    esac
}

# Explain an error using Gemini (free)
explain-error() {
    echo "Explain this error and suggest a fix: $*" | gemini
}

# Quick security scan with Claude (most thorough)
security-scan() {
    claude "Perform a security audit of ${1:-.}. Focus on: injection vulnerabilities, authentication issues, exposed secrets, insecure defaults."
}

# Generate tests with Codex in full-auto mode
auto-test() {
    codex --full-auto "Write comprehensive unit tests for $1. Use the existing test framework and patterns in the project."
}
```

## MCP Configuration Commands

### Add Memorix to Claude Code (shared memory across tools)

```bash
# Project-level MCP config
cat > .mcp.json << 'MCPEOF'
{
  "mcpServers": {
    "memorix": {
      "command": "npx",
      "args": ["-y", "memorix"],
      "env": {}
    }
  }
}
MCPEOF
```

### Add PAL MCP Server (multi-model access from Claude Code)

```bash
# Clone and setup PAL
git clone https://github.com/BeehiveInnovations/pal-mcp-server.git ~/tools/pal-mcp-server
cd ~/tools/pal-mcp-server && ./run-server.sh

# Add to global Claude Code config
# Edit ~/.claude/settings.json and add under mcpServers:
# "pal": {
#   "command": "node",
#   "args": ["~/tools/pal-mcp-server/dist/index.js"],
#   "env": {
#     "GEMINI_API_KEY": "your-key",
#     "OPENAI_API_KEY": "your-key"
#   }
# }
```

## Environment Setup Checklist

```bash
#!/bin/bash
# check-ai-tools.sh -- Verify all AI tools are installed and authenticated

echo "Checking AI coding tools..."
echo ""

# Claude Code
if command -v claude &>/dev/null; then
    echo "[OK] Claude Code: $(claude --version 2>/dev/null || echo 'installed')"
else
    echo "[MISSING] Claude Code: npm i -g @anthropic-ai/claude-code"
fi

# Gemini CLI
if command -v gemini &>/dev/null; then
    echo "[OK] Gemini CLI: installed (FREE - 1000 req/day)"
else
    echo "[MISSING] Gemini CLI: npm i -g @google/gemini-cli"
fi

# Codex CLI
if command -v codex &>/dev/null; then
    echo "[OK] Codex CLI: installed"
else
    echo "[MISSING] Codex CLI: npm i -g @openai/codex"
fi

# Aider
if command -v aider &>/dev/null; then
    echo "[OK] Aider: $(aider --version 2>/dev/null || echo 'installed')"
else
    echo "[MISSING] Aider: pip install aider-chat"
fi

# OpenCode
if command -v opencode &>/dev/null; then
    echo "[OK] OpenCode: installed"
else
    echo "[MISSING] OpenCode: brew install opencode (or download from opencode.ai)"
fi

# Ollama (for free local models)
if command -v ollama &>/dev/null; then
    echo "[OK] Ollama: installed (free local models)"
    ollama list 2>/dev/null | head -5
else
    echo "[OPTIONAL] Ollama: brew install ollama (for free local models)"
fi

# tmux (for multi-pane sessions)
if command -v tmux &>/dev/null; then
    echo "[OK] tmux: installed (needed for multi-pane AI sessions)"
else
    echo "[RECOMMENDED] tmux: brew install tmux"
fi

echo ""
echo "Free tools (no API key): Gemini CLI, Ollama + Aider/OpenCode"
echo "Subscription tools: Claude Code, Codex CLI"
```

```bash
chmod +x check-ai-tools.sh
```

## Sources

- [Claude Code CLI Reference](https://code.claude.com/docs/en/cli)
- [Codex CLI Reference](https://developers.openai.com/codex/cli/reference)
- [Gemini CLI GitHub](https://github.com/google-gemini/gemini-cli)
- [Aider Usage](https://aider.chat/)
- [OpenCode Docs](https://opencode.ai/docs/)
