# MCP Server Setup Guide for Claude Code

MCP (Model Context Protocol) servers connect Claude Code to external tools and services. This guide documents a working production setup that powers daily operations — email triage, calendar management, Slack monitoring, note-taking, literature search, and data analysis — all from a single terminal session.

There are two types of integrations:
- **Standalone MCP servers**: Configured in `~/.mcp.json`, launched as subprocesses by Claude Code
- **Claude Code first-party plugins**: Configured via Claude Code's built-in integration settings (not in `~/.mcp.json`), use OAuth managed by Claude Code itself

## Architecture

```
Claude Code (terminal)
  │
  ├── Standalone MCP servers (configured in ~/.mcp.json)
  │   ├── google-workspace (uvx)  → Gmail, Calendar, Drive, Docs, Sheets, Slides
  │   ├── airtable (npx)          → Bases, Tables, Records
  │   ├── paper-search (uv run)   → PubMed, arXiv, bioRxiv, Google Scholar
  │   ├── pubchem (npx)           → Compound lookup (name/CID/SMILES)
  │   ├── mcp-pandas (uvx)        → CSV/Excel/JSON analysis with pandas
  │   ├── obsidian (uvx)          → Markdown file read/browse
  │   ├── playwright (npx)        → Headless browser automation
  │   ├── rss (npx)               → RSS/Atom feed fetching
  │   └── apple-health (uv run)   → Health data via DuckDB
  │
  └── Claude Code first-party plugins (configured in Claude Code settings)
      ├── Slack                    → Channels, DMs, Search, Send
      └── Notion                   → Pages, Databases, Comments
```

## Prerequisites

1. **Claude Code** installed and working
2. **Python tooling**: `uv` and `uvx` installed (`curl -LsSf https://astral.sh/uv/install.sh | sh`)
3. **Node.js**: `npx` available (comes with Node.js)
4. **API credentials** for each service (see per-server sections below)

---

## First-Party Plugins (Slack, Notion)

These are **not** configured in `~/.mcp.json`. They are managed through Claude Code's built-in integration system.

### Slack

Configured via Claude Code's built-in integrations. Uses OAuth — no API keys to manage manually.

- **Setup**: In Claude Code, go to integrations/plugins and enable Slack. Follow the OAuth flow to authorize your workspace.
- **Tool prefix**: `mcp__claude_ai_Slack__`
- **Capabilities**: Read channels, DMs, threads; search messages (public + private); send messages

### Notion

Configured via Claude Code's built-in integrations. Uses OAuth — no API keys to manage manually.

- **Setup**: In Claude Code, go to integrations/plugins and enable Notion. Follow the OAuth flow.
- **Tool prefix**: `mcp__notion__`
- **Capabilities**: Search, read/create/update pages, query databases, manage comments
- **Important**: Share relevant pages and databases with the Notion integration for access.

---

## Standalone MCP Servers (`~/.mcp.json`)

All standalone servers are configured in `~/.mcp.json`. Below is a complete working configuration.

### Full `~/.mcp.json`

```json
{
  "mcpServers": {
    "google-workspace": {
      "command": "uvx",
      "args": [
        "workspace-mcp",
        "--single-user",
        "--tools", "gmail", "drive", "docs", "sheets", "slides", "calendar"
      ],
      "env": {
        "GOOGLE_OAUTH_CLIENT_ID": "CUSTOMIZE: your-google-client-id.apps.googleusercontent.com",
        "GOOGLE_OAUTH_CLIENT_SECRET": "CUSTOMIZE: your-google-client-secret",
        "USER_GOOGLE_EMAIL": "CUSTOMIZE: you@example.com"
      }
    },
    "airtable": {
      "command": "npx",
      "args": ["-y", "airtable-mcp-server"],
      "env": {
        "AIRTABLE_API_KEY": "CUSTOMIZE: your-airtable-personal-access-token"
      }
    },
    "paper-search": {
      "command": "uv",
      "args": [
        "run",
        "--directory", "CUSTOMIZE: /absolute/path/to/paper-search-mcp"
      ]
    },
    "pubchem": {
      "command": "npx",
      "args": ["-y", "@cyanheads/pubchem-mcp-server"]
    },
    "mcp-pandas": {
      "command": "uvx",
      "args": ["mcp-pandas"]
    },
    "obsidian": {
      "command": "uvx",
      "args": ["mcp-obsidian"],
      "env": {
        "OBSIDIAN_API_KEY": "CUSTOMIZE: your-obsidian-local-rest-api-key",
        "OBSIDIAN_HOST": "CUSTOMIZE: e.g. https://127.0.0.1",
        "OBSIDIAN_PORT": "CUSTOMIZE: e.g. 27124"
      }
    },
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest", "--headless"]
    },
    "rss": {
      "command": "npx",
      "args": ["-y", "rss-mcp"]
    },
    "apple-health": {
      "command": "uv",
      "args": [
        "run",
        "--frozen",
        "--directory", "CUSTOMIZE: /absolute/path/to/apple-health-mcp-server"
      ],
      "env": {
        "RAW_XML_PATH": "CUSTOMIZE: /path/to/apple-health-export.xml",
        "DUCKDB_FILENAME": "CUSTOMIZE: /path/to/health.duckdb",
        "CHUNK_SIZE": "500000"
      }
    }
  }
}
```

> **All `CUSTOMIZE:` values must be replaced with your actual credentials/paths before use.**

---

## Per-Server Setup

### 1. Google Workspace

**Package**: [`workspace-mcp`](https://github.com/taylorwilsdon/workspace-mcp) by taylorwilsdon

1. Create a Google Cloud project at https://console.cloud.google.com
2. Enable APIs: Gmail, Calendar, Drive, Docs, Sheets, Slides
3. Create OAuth 2.0 credentials — choose **Web Application** type
4. Set authorized redirect URI to `http://localhost:8000/oauth2callback`
5. Copy the Client ID and Client Secret into `~/.mcp.json`
6. Set `USER_GOOGLE_EMAIL` to your Google account email
7. On first use, a browser window will open for OAuth consent

**Tool prefix**: `mcp__google-workspace__`

**Required env vars**:
| Variable | Description |
|---|---|
| `GOOGLE_OAUTH_CLIENT_ID` | OAuth 2.0 Client ID (`*.apps.googleusercontent.com`) |
| `GOOGLE_OAUTH_CLIENT_SECRET` | OAuth 2.0 Client Secret |
| `USER_GOOGLE_EMAIL` | Your Google email (required on every API call) |

### 2. Airtable

**Package**: `airtable-mcp-server` (npm)

1. Go to https://airtable.com/create/tokens
2. Create a personal access token with scopes: `data.records:read`, `data.records:write`, `schema.bases:read`
3. Grant access to the specific bases you want Claude to use
4. Copy the token into `~/.mcp.json`

**Tool prefix**: `mcp__airtable__`

### 3. Paper Search

**Package**: `paper-search-mcp` (local install via git clone)

1. Clone the repository to a local directory
2. Set the `--directory` arg in `~/.mcp.json` to the absolute path of that directory
3. No API keys required — searches PubMed, arXiv, bioRxiv, Google Scholar, Semantic Scholar, IACR, CrossRef, medRxiv

**Tool prefix**: `mcp__paper-search__`

### 4. PubChem

**Package**: `@cyanheads/pubchem-mcp-server` (npm)

No configuration needed. Uses the public PubChem API.

**Tool prefix**: `mcp__pubchem__`

### 5. mcp-pandas

**Package**: `mcp-pandas` (PyPI, run via `uvx`)

No configuration needed. Point it at local CSV, Excel, or JSON files during use.

**Tool prefix**: `mcp__mcp-pandas__`

### 6. Obsidian

**Package**: `mcp-obsidian` (PyPI, run via `uvx`)

Requires the [Obsidian Local REST API](https://github.com/coddingtonbear/obsidian-local-rest-api) plugin to be installed and running in Obsidian.

1. Install the Local REST API plugin in Obsidian
2. Copy the API key from the plugin settings
3. Set the host and port in `~/.mcp.json`

**Tool prefix**: `mcp__obsidian__`

### 7. Playwright

**Package**: `@playwright/mcp` (npm)

No configuration needed. Runs in headless mode for browser automation (useful for JS-heavy pages that cannot be fetched with simple HTTP requests).

**Tool prefix**: `mcp__playwright__`

### 8. RSS

**Package**: `rss-mcp` (npm)

No configuration needed. Fetches and parses RSS/Atom feeds on demand.

**Tool prefix**: `mcp__rss__`

### 9. Apple Health

**Package**: Local install (git clone)

1. Export health data from Apple Health on your iPhone
2. Set `RAW_XML_PATH` to the exported XML file
3. Set `DUCKDB_FILENAME` to where you want the DuckDB database stored
4. `CHUNK_SIZE` controls import batching (default `500000` works well)

**Tool prefix**: `mcp__apple-health__`

---

## Verification

After configuring, restart Claude Code and test each integration:

| Test prompt | What it verifies |
|---|---|
| "What's on my calendar today?" | Google Calendar |
| "Check my unread emails" | Gmail |
| "Read #general in Slack" | Slack (first-party plugin) |
| "Search Notion for meeting notes" | Notion (first-party plugin) |
| "List Airtable bases" | Airtable |
| "Search arXiv for rumen microbiome" | Paper Search |
| "Look up caffeine on PubChem" | PubChem |
| "List files in my vault" | Obsidian |

If a tool returns "tool not found", the server has not started. Check `~/.mcp.json` syntax and restart Claude Code.

---

## Known Issues and Workarounds

### Google Workspace

- **`from:` search is broken**: Gmail search using `from:` does not work reliably with workspace-mcp. Use `subject:` queries instead. For example, search Scholar alerts with `subject:"new results" is:unread` rather than `from:scholaralerts-noreply@google.com`.

- **Archive requires removing BOTH labels**: To properly archive a Gmail message, you must remove both the `INBOX` and `UNREAD` labels. Removing only `INBOX` leaves the message marked unread and it may resurface.

- **Always use `page_size: 50` for Gmail search**: Smaller page sizes can miss results; larger ones may time out.

- **Port 8000 can get stuck**: The Google Workspace MCP server uses port 8000 for OAuth callbacks. If a Claude Code session does not terminate cleanly, a stale process may hold the port. Fix with:
  ```bash
  lsof -ti:8000 | xargs kill -9
  ```
  Then restart Claude Code.

### Subagent Reliability

- **Subagents may silently fail on Gmail label changes**: A subagent (spawned via the Agent tool) may report success on Gmail label modifications without the change actually taking effect. Always execute archive/label steps in the main conversation, never delegate them to subagents.

---

## Security Notes

- **Never commit `~/.mcp.json` to git** — it contains API keys and secrets. Add it to your global `.gitignore`.
- **Use minimal scopes** — only grant the permissions each integration actually needs.
- **Review before sending** — always have Claude draft emails and Slack messages for your approval before sending.
- **Prompt injection risk** — email and Slack content could contain instructions aimed at manipulating Claude. Be aware of this when triaging messages from unknown senders.
- **OAuth tokens are cached locally** — the Google Workspace server stores tokens on disk after the initial OAuth flow. Treat these as sensitive credentials.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| "Tool not found" | MCP server failed to start | Check `~/.mcp.json` syntax (valid JSON?). Restart Claude Code. |
| Authentication error on Google | OAuth token expired or revoked | Delete cached token and re-run OAuth flow. |
| Notion/Slack tools missing | First-party plugin not enabled | Check Claude Code settings/integrations, re-authorize. |
| Notion returns empty results | Pages not shared with integration | Share target pages/databases with the Notion integration. |
| `npx` command not found | Node.js not installed | Install Node.js (includes npm/npx). |
| `uvx` command not found | `uv` not installed | Install with `curl -LsSf https://astral.sh/uv/install.sh \| sh` |
| Port 8000 already in use | Stale Google Workspace OAuth listener | Run `lsof -ti:8000 \| xargs kill -9` |
| Paper search returns nothing | Local repo path is wrong | Verify the `--directory` path in `~/.mcp.json` points to the cloned repo |
| Playwright times out | Browser dependencies missing | Run `npx playwright install chromium` to install browser binaries |
