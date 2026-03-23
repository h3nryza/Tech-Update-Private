# Setting Up MCP Servers for Slack

## Overview

The Model Context Protocol (MCP) connects Claude Code to your Slack workspace, enabling AI-powered message search, channel management, ChatOps workflows, and team communication directly from your terminal. Slack provides an official MCP server with OAuth support.

## Architecture

```mermaid
graph TB
    subgraph "Claude Code"
        CLI[CLI Interface]
        ENGINE[Skill Engine]
    end

    subgraph "MCP Transport"
        STDIO[stdio transport]
        HTTP[HTTP/OAuth]
    end

    subgraph "MCP Servers"
        OFFICIAL[Official Slack MCP Server<br/>@anthropic-ai/slack-mcp-server]
        COMMUNITY[Community Server<br/>jtalk22/slack-mcp-server]
        COMPOSIO[Composio Slack<br/>Integration]
    end

    subgraph "Slack Workspace"
        CHANNELS[Channels & DMs]
        THREADS[Threads]
        SEARCH[Message Search]
        USERS[Users & Profiles]
        CANVAS[Canvas Documents]
        SCHEDULE[Scheduled Messages]
    end

    CLI --> ENGINE
    ENGINE --> STDIO
    ENGINE --> HTTP
    STDIO --> OFFICIAL
    STDIO --> COMMUNITY
    STDIO --> COMPOSIO
    OFFICIAL --> CHANNELS
    OFFICIAL --> THREADS
    OFFICIAL --> SEARCH
    OFFICIAL --> USERS
    OFFICIAL --> CANVAS
    OFFICIAL --> SCHEDULE
```

## Prerequisites

```bash
# Verify requirements
node --version    # 18+ (for npx-based servers)
claude --version  # Latest
```

You also need:
- A Slack workspace where you have permission to install apps
- A Slack Bot Token (xoxb-) and/or App Token (xapp-)

## Option 1: Official Slack MCP Server (Recommended)

### Install via CLI

```bash
claude mcp add slack \
  --transport stdio \
  -- npx -y @anthropic-ai/slack-mcp-server
```

The server prompts for OAuth authentication on first use.

### Token-Based Configuration

```json
// .claude/mcp.json
{
  "mcpServers": {
    "slack": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/slack-mcp-server"],
      "env": {
        "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}",
        "SLACK_APP_TOKEN": "${SLACK_APP_TOKEN}"
      }
    }
  }
}
```

## Option 2: Community MCP Server

For session-based local-first workflows:

```bash
claude mcp add slack \
  --transport stdio \
  -- npx -y slack-mcp-server
```

## Option 3: Composio Integration

```bash
pip install composio-claude
composio add slack
```

---

## Creating a Slack App

### Step 1: Create the App

1. Go to https://api.slack.com/apps
2. Click "Create New App" > "From scratch"
3. Name it (e.g., "Claude Code MCP") and select your workspace
4. Click "Create App"

### Step 2: Configure Bot Token Scopes

Go to **OAuth & Permissions** > **Bot Token Scopes** and add:

| Scope | Purpose |
|-------|---------|
| `channels:read` | List public channels |
| `channels:history` | Read public channel messages |
| `groups:read` | List private channels you belong to |
| `groups:history` | Read private channel messages |
| `im:read` | List direct message channels |
| `im:history` | Read direct messages |
| `im:write` | Send direct messages |
| `chat:write` | Send messages to channels |
| `chat:write.customize` | Send messages with custom username/icon |
| `search:read` | Search messages and files |
| `users:read` | View user profiles |
| `users:read.email` | View user email addresses |
| `app_mentions:read` | Read @mentions of the bot |
| `commands` | Register slash commands |

### Step 3: Enable Socket Mode (for Real-Time Events)

1. Go to **Socket Mode** in the sidebar
2. Enable Socket Mode
3. Create an App-Level Token with `connections:write` scope
4. Copy the token (starts with `xapp-`)

### Step 4: Subscribe to Events

Go to **Event Subscriptions** > Enable Events > Subscribe to bot events:
- `app_mention` -- When someone @mentions the bot
- `message.im` -- Direct messages to the bot
- `message.channels` -- Messages in public channels (optional)

### Step 5: Install to Workspace

1. Go to **Install App** in the sidebar
2. Click "Install to Workspace"
3. Authorize the requested permissions
4. Copy the **Bot User OAuth Token** (starts with `xoxb-`)

---

## Authentication Setup

### Store Tokens Securely

```bash
# Shell profile
echo 'export SLACK_BOT_TOKEN="xoxb-your-bot-token"' >> ~/.zshrc
echo 'export SLACK_APP_TOKEN="xapp-your-app-token"' >> ~/.zshrc

# macOS Keychain
security add-generic-password -s "slack-bot-token" -a "$USER" -w "xoxb-your-bot-token"
security add-generic-password -s "slack-app-token" -a "$USER" -w "xapp-your-app-token"

# Environment variable retrieval
export SLACK_BOT_TOKEN=$(security find-generic-password -s "slack-bot-token" -w)
export SLACK_APP_TOKEN=$(security find-generic-password -s "slack-app-token" -w)
```

### Minimal vs. Full Permissions

**Read-only setup** (safer, for monitoring and search):
- `channels:read`, `channels:history`, `groups:read`, `groups:history`
- `search:read`, `users:read`

**Full ChatOps setup** (read + write):
- All read scopes plus `chat:write`, `im:write`, `chat:write.customize`
- `commands` for slash command registration

## MCP Server Tools

| Tool | Description |
|------|-------------|
| `send_message` | Send a message to a channel or DM |
| `read_channel` | Read recent messages from a channel |
| `read_thread` | Read all replies in a specific thread |
| `search_public` | Search public channel messages |
| `search_public_and_private` | Search all accessible messages |
| `search_channels` | Find channels by name or topic |
| `search_users` | Find users by name or email |
| `read_user_profile` | Get user profile information |
| `create_canvas` | Create a Slack Canvas document |
| `read_canvas` | Read a Canvas document |
| `update_canvas` | Update a Canvas document |
| `schedule_message` | Schedule a message for later delivery |
| `send_message_draft` | Create a message draft |

---

## Verification

After setup, verify the MCP server is connected:

```bash
# List configured MCP servers
claude mcp list

# Test the connection
claude "Search Slack for messages about the latest deployment"

# Check server health
/mcp
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `invalid_auth` | Verify bot token starts with `xoxb-`; reinstall app if needed |
| `missing_scope` | Add the required scope in **OAuth & Permissions** and reinstall |
| `channel_not_found` | Invite the bot to the channel with `/invite @YourBot` |
| `not_in_channel` | Bot must be a member of the channel to read/write messages |
| `MCP server won't start` | Check Node.js 18+; run `npx -y @anthropic-ai/slack-mcp-server` standalone |
| `Socket mode not working` | Ensure app-level token (xapp-) has `connections:write` scope |
| `Search returns no results` | `search:read` scope required; only indexes messages the bot can see |
| `Rate limited` | Slack API rate limits apply; back off and retry |

## Sources

- [Slack MCP Server - Slack Developer Docs](https://docs.slack.dev/ai/slack-mcp-server/connect-to-claude/)
- [Claude Code in Slack - Official Docs](https://code.claude.com/docs/en/slack)
- [Slack MCP Server Setup Guide (Digidog)](https://digidog.org/blog/slack-mcp-server-setup-guide)
- [Composio Slack MCP Integration](https://composio.dev/toolkits/slack/framework/claude-code)
- [Claude Code MCP Documentation](https://code.claude.com/docs/en/mcp)
