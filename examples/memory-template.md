# Project Memory Template

This file lives at `~/.claude/projects/<project-path>/memory/MEMORY.md` and persists across Claude Code conversations. It's automatically loaded into context at the start of each session.

## How It Works

Claude Code's auto-memory system lets you save persistent context that carries across sessions. Instead of re-explaining your project, team, and preferences every time, you maintain a living memory file that Claude reads automatically.

## What to Include

### Project Context
- Active projects and their current status
- Key collaborators (names, roles, contact info)
- Important dates and deadlines

### Workflow Preferences
- Communication style preferences
- Tool preferences (e.g., "always use bun instead of npm")
- Approval workflows (e.g., "always draft before sending")

### Integration Details
- Database IDs (Notion, Airtable)
- Channel IDs (Slack)
- API endpoints or service configurations

### Decisions & Status
- Key decisions made and their rationale
- Blockers and their current status
- Items waiting on external input

## Example Structure

```markdown
# Project Memory

## Active Projects
- **Project Alpha**: [status, key details]
- **Project Beta**: [status, key details]

## Key Collaborators
- **Alice** (alice@company.com) — Project Alpha lead
- **Bob** (bob@company.com) — Lab manager

## Workflow Preferences
- Draft all emails before sending
- Use GMT for all time references
- Skip promotional emails in triage

## Integration IDs
- Notion Literature DB: `collection://xxx`
- Notion Decision Log: `collection://yyy`
- Slack key channels: #project-alpha (C12345), #team (C67890)

## Session Patterns
- Morning briefing covers: Calendar, Email, Slack, Notion → Todos
- End of day: update daily log, update todos
- End of week: weekly summary, backfill missing logs
```

## Tips

- **Keep it concise** — MEMORY.md is loaded every session, so lines after ~200 get truncated
- **Use separate files for detail** — Link from MEMORY.md to topic-specific files (e.g., `debugging.md`, `patterns.md`)
- **Update regularly** — Correct outdated info, remove completed items
- **Let Claude update it** — Say "remember that X" and Claude will add it to memory
- **Don't duplicate CLAUDE.md** — Memory is for dynamic state; CLAUDE.md is for stable instructions
