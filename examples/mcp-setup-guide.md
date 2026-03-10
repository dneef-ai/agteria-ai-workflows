# MCP Server Setup Guide

MCP (Model Context Protocol) servers connect Claude Code to external tools and services. This is what makes the "Command Center" possible — Claude can read your email, check your calendar, post to Slack, and query databases all from the terminal.

## What You Need

1. **Claude Code** installed and working
2. **MCP server packages** for each integration
3. **API credentials** for each service

## Architecture

```
Claude Code (terminal)
  ├── Google Workspace MCP → Gmail, Calendar, Drive, Docs, Sheets
  ├── Slack MCP → Channels, DMs, Search
  ├── Notion MCP → Pages, Databases, Comments
  ├── Airtable MCP → Bases, Tables, Records
  └── (other MCPs as needed)
```

## Configuration

MCP servers are configured in `~/.mcp.json`. Each entry specifies:
- The server command (usually `npx` or `uvx`)
- Environment variables (API keys, tokens)

### Example `~/.mcp.json` structure

```json
{
  "mcpServers": {
    "google-workspace": {
      "command": "npx",
      "args": ["-y", "@anthropic/google-workspace-mcp"],
      "env": {
        "GOOGLE_CLIENT_ID": "your-client-id",
        "GOOGLE_CLIENT_SECRET": "your-client-secret",
        "GOOGLE_REDIRECT_URI": "http://localhost:8000/oauth2callback"
      }
    },
    "slack": {
      "command": "npx",
      "args": ["-y", "@anthropic/slack-mcp"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-your-token"
      }
    },
    "notion": {
      "command": "npx",
      "args": ["-y", "@anthropic/notion-mcp"],
      "env": {
        "NOTION_API_KEY": "your-notion-api-key"
      }
    },
    "airtable": {
      "command": "npx",
      "args": ["-y", "@anthropic/airtable-mcp"],
      "env": {
        "AIRTABLE_API_KEY": "your-airtable-key"
      }
    }
  }
}
```

> **Note**: The exact package names and configuration may vary. Check the official MCP server registry for current packages: https://github.com/modelcontextprotocol/servers

## Setup Steps

### 1. Google Workspace
- Create a Google Cloud project
- Enable Gmail, Calendar, and Drive APIs
- Create OAuth 2.0 credentials (Web Application type)
- Set redirect URI to `http://localhost:8000/oauth2callback`
- First use will prompt browser-based OAuth flow

### 2. Slack
- Create a Slack app at api.slack.com
- Add required scopes (channels:read, chat:write, search:read, users:read, etc.)
- Install to workspace and get bot token

### 3. Notion
- Create a Notion integration at notion.so/my-integrations
- Share relevant pages/databases with the integration
- Copy the API key

### 4. Airtable
- Generate a personal access token at airtable.com/account
- Grant access to the bases you want Claude to use
- Scopes needed: data.records:read, data.records:write, schema.bases:read

## Verification

After configuring, restart Claude Code and test each integration:
- "What's on my calendar today?" → Tests Google Calendar
- "Check my unread emails" → Tests Gmail
- "Read #general in Slack" → Tests Slack
- "Search Notion for meeting notes" → Tests Notion
- "List Airtable bases" → Tests Airtable

## Security Notes

- **Never commit API keys** to git. Use environment variables or a `.env` file
- **Use minimal scopes** — only grant the permissions each integration needs
- **Review before sending** — always confirm before Claude sends emails or Slack messages
- **Be aware of prompt injection** — email and Slack content could contain instructions aimed at Claude. Stay alert.

## Troubleshooting

- **"Tool not found"**: MCP server may not have started. Check `~/.mcp.json` syntax.
- **Authentication errors**: Token may have expired. Re-run OAuth flow or regenerate API key.
- **Missing data**: Ensure Notion pages/Airtable bases are shared with the integration.
