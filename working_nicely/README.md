# Multi-Tool AI Coding Integration Guide

Using multiple AI coding tools together from the terminal to leverage each tool's strengths.

## The Landscape (March 2026)

| Tool | Install | Auth | Free Tier | Best For |
|------|---------|------|-----------|----------|
| **Claude Code** | `npm i -g @anthropic-ai/claude-code` | Anthropic API key or Claude subscription | Claude Pro/Max subscription includes usage | Deep reasoning, refactoring, multi-file edits, agent teams |
| **Gemini CLI** | `npx @google/gemini-cli` | Google account login (browser) | **1,000 req/day free** with personal Google account | Large context (1M tokens), Google Search grounding, zero-cost usage |
| **Codex CLI** | `npm i -g @openai/codex` | ChatGPT account or OpenAI API key | Free with ChatGPT Free/Plus (limited time) | Autonomous task execution, full-auto mode, sandbox execution |
| **Aider** | `pip install aider-chat` | API key for chosen provider | Tool is free; pay for LLM API calls | Git-native pair programming, auto-commits, multi-model support |
| **OpenCode** | Binary download or `brew install opencode` | API key for chosen provider | Tool is free; pay for LLM API calls (or use Ollama for free) | TUI interface, multi-provider support, LSP integration |

## Truly Free Options (No API Key Required)

1. **Gemini CLI** -- Login with any personal Google account. 1,000 requests/day of Gemini 2.5 Pro free. No credit card, no API key.
2. **Codex CLI** -- Login with ChatGPT account. Free tier includes Codex usage (limited time offer as of early 2026).
3. **Aider + Ollama** -- Run local models (e.g., DeepSeek, Llama) via Ollama. Zero API cost. Quality depends on your hardware.
4. **OpenCode + Ollama** -- Same local model approach. Good TUI experience.

## Integration Bridges

Several tools exist to make these work together:

| Bridge Tool | What It Does | GitHub |
|-------------|-------------|--------|
| **Claude Code Bridge (CCB)** | Split-pane terminal with Claude, Codex, and Gemini side-by-side | [bfly123/claude_code_bridge](https://github.com/bfly123/claude_code_bridge) |
| **PAL MCP Server** | MCP server connecting Claude Code/Gemini CLI/Codex CLI to multiple AI providers | [BeehiveInnovations/pal-mcp-server](https://github.com/BeehiveInnovations/pal-mcp-server) |
| **Memorix** | Cross-agent persistent memory shared across 10 IDEs/tools via MCP | [AVIDS2/memorix](https://github.com/AVIDS2/memorix) |
| **MCP Agent Bridge** | Bidirectional MCP communication between Codex, Claude, and Gemini CLIs | [vizi2000/mcp-agent-bridge](https://github.com/vizi2000/mcp-agent-bridge) |
| **Gemini MCP** | MCP server letting Claude Code call Gemini models | [RLabs-Inc/gemini-mcp](https://github.com/RLabs-Inc/gemini-mcp) |

## File Index

- [codex_integration.md](codex_integration.md) -- Claude Code + OpenAI Codex CLI
- [gemini_integration.md](gemini_integration.md) -- Claude Code + Google Gemini CLI
- [multi_tool_workflow.md](multi_tool_workflow.md) -- Combining multiple tools in real workflows
- [example_skills.md](example_skills.md) -- Claude Code skills that bridge tools
- [example_agents.md](example_agents.md) -- Agents coordinating between tools
- [example_commands.md](example_commands.md) -- Terminal commands, aliases, and shell functions

## Sources

- [OpenAI Codex CLI GitHub](https://github.com/openai/codex)
- [Google Gemini CLI GitHub](https://github.com/google-gemini/gemini-cli)
- [Aider GitHub](https://github.com/Aider-AI/aider)
- [OpenCode GitHub](https://github.com/opencode-ai/opencode)
- [Claude Code Bridge GitHub](https://github.com/bfly123/claude_code_bridge)
- [PAL MCP Server GitHub](https://github.com/BeehiveInnovations/pal-mcp-server)
- [Memorix GitHub](https://github.com/AVIDS2/memorix)
- [Claude Code Skills Docs](https://code.claude.com/docs/en/skills)
- [Codex CLI Docs](https://developers.openai.com/codex/cli)
- [Gemini CLI Docs](https://google-gemini.github.io/gemini-cli/)
- [Top 5 Terminal AI Coding Agents in 2026](https://dev.to/nebulagg/top-5-terminal-ai-coding-agents-in-2026-272)
- [9 AI Coding Alternatives for Terminal Development: Complete 2026 Guide](https://www.pasqualepillitteri.it/en/news/386/ai-coding-cli-alternatives-2026)
