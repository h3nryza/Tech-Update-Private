# Slack Integration with Claude Code

## Overview

Claude Code integrates with Slack through the official Slack MCP server, the Claude Code Slack bot, and native Claude in Slack features. This enables ChatOps workflows, incident management, team notifications, and AI-powered coding assistance directly in Slack channels.

## Architecture

```mermaid
graph TB
    subgraph "Slack Workspace"
        CHANNELS[Channels]
        DMS[Direct Messages]
        THREADS[Threads]
        CANVAS[Canvas Documents]
        WORKFLOWS[Workflow Builder]
    end

    subgraph "Claude Code"
        CLI[Claude Code CLI]
        SDK[Claude Agent SDK]
        SKILLS[Skills & Agents]
    end

    subgraph "Integration Methods"
        MCP[Slack MCP Server]
        BOT[Claude Code Slack Bot]
        NATIVE[Claude in Slack]
        WEBHOOK[Incoming Webhooks]
    end

    subgraph "ChatOps Capabilities"
        INCIDENTS[Incident Management]
        DEPLOYS[Deploy Notifications]
        ALERTS[Alert Routing]
        CODE[Code Assistance]
    end

    CLI --> SKILLS
    SKILLS --> MCP
    MCP --> CHANNELS
    MCP --> DMS
    MCP --> THREADS
    MCP --> CANVAS
    BOT --> CHANNELS
    BOT --> CODE
    NATIVE --> CHANNELS
    NATIVE --> DMS
    WEBHOOK --> CHANNELS
    CHANNELS --> INCIDENTS
    CHANNELS --> DEPLOYS
    CHANNELS --> ALERTS
```

## Quick Start

### Option 1: Official Slack MCP Server

```bash
# Add the Slack MCP server (OAuth-based)
claude mcp add slack \
  --transport stdio \
  -- npx -y @anthropic-ai/slack-mcp-server
```

The server prompts for OAuth authentication on first use.

### Option 2: Token-Based Configuration

```json
// .claude/mcp.json
{
  "mcpServers": {
    "slack": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/slack-mcp-server"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-your-bot-token",
        "SLACK_APP_TOKEN": "xapp-your-app-token"
      }
    }
  }
}
```

### Option 3: Claude Code Slack Bot

For a persistent Slack bot that runs Claude Code sessions:

```bash
# Clone and configure the slack bot
git clone https://github.com/mpociot/claude-code-slack-bot.git
cd claude-code-slack-bot
cp .env.example .env
# Edit .env with your Slack and Anthropic tokens
npm install && npm start
```

### Option 4: Claude in Slack (Native)

Claude is available natively in Slack for Pro/Max/Team/Enterprise subscribers:
1. Go to [claude.ai/directory](https://claude.ai/directory)
2. Enable the Slack integration
3. Mention @Claude in any channel

## MCP Server Capabilities

| Tool | Description |
|------|-------------|
| `send_message` | Send a message to a channel or DM |
| `read_channel` | Read recent messages from a channel |
| `read_thread` | Read a specific thread |
| `search_public` | Search public channel messages |
| `search_public_and_private` | Search all accessible messages |
| `search_channels` | Find channels by name or topic |
| `search_users` | Find users by name or email |
| `read_user_profile` | Get user profile information |
| `create_canvas` | Create a Slack canvas document |
| `read_canvas` | Read a canvas document |
| `update_canvas` | Update a canvas document |
| `schedule_message` | Schedule a message for later |
| `send_message_draft` | Create a message draft |

## Creating a Slack Bot for Claude Code

### Step 1: Create Slack App

1. Go to https://api.slack.com/apps
2. Click "Create New App" > "From scratch"
3. Set scopes:
   - `channels:history`, `channels:read`, `channels:write`
   - `chat:write`, `chat:write.customize`
   - `groups:read`, `groups:history`
   - `im:read`, `im:write`, `im:history`
   - `users:read`
   - `app_mentions:read`
   - `commands`
4. Enable Socket Mode for real-time events
5. Subscribe to events: `app_mention`, `message.im`

### Step 2: Configure Bot

```bash
# Environment variables
export SLACK_BOT_TOKEN="xoxb-..."
export SLACK_APP_TOKEN="xapp-..."
export ANTHROPIC_API_KEY="sk-ant-..."
```

### Step 3: Set Up Event Handling

When someone mentions @Claude in Slack:
1. Bot receives the `app_mention` event
2. Creates a Claude Code session with the message as prompt
3. Claude processes the request
4. Bot posts the response in a thread

## File Index

- [skills.md](skills.md) - Slack integration skills
- [agents.md](agents.md) - Slack agents
- [slash_commands.md](slash_commands.md) - Slack slash commands
- [mcp_setup.md](mcp_setup.md) - Detailed MCP server setup guide

## Sources

- [Claude Code in Slack Docs](https://code.claude.com/docs/en/slack)
- [Slack MCP Server - Slack Developer Docs](https://docs.slack.dev/ai/slack-mcp-server/connect-to-claude/)
- [Claude Code Slack Bot](https://github.com/mpociot/claude-code-slack-bot)
- [Composio Slack MCP](https://composio.dev/toolkits/slack/framework/claude-code)
- [Claude Cowork Slack Integration](https://www.eesel.ai/blog/claude-cowork-slack-integration)
