# Project Memory Template

This file lives at `~/.claude/projects/<project-path>/memory/MEMORY.md` and persists across Claude Code conversations. It's automatically loaded into context at the start of each session.

## The 200-Line Rule

**Only the first ~200 lines of MEMORY.md are loaded at session start.** Content beyond that may not be read, which means end-of-file instructions get silently ignored. If your MEMORY.md is growing past 150 lines, it's time to split.

## Recommended Structure: Index + Topic Files

Keep MEMORY.md as a **short index** (~50-80 lines) with only what's needed every session. Move everything else into topic-specific files that Claude reads on demand.

```
memory/
├── MEMORY.md              # Index — loaded every session (~50-80 lines)
├── collaborators.md       # Team and external contacts
├── setup.md               # Technical config, MCP servers, tools
├── notion-databases.md    # Database IDs, field schemas, API quirks
├── grants-and-strategy.md # Funding, strategic decisions
└── daniel-profile.md      # Personal context (rarely needed)
```

### What stays in MEMORY.md (loaded every session)
- Essential context (timezone, communication preferences)
- Todo list location
- Daily workflow rules (what to scan, what to skip, known bugs)
- Auto-fill database IDs (Literature Tracker, Decision Log)
- Active project summary (1-2 lines each)
- Weekly cadence / schedule

### What goes in topic files (loaded on demand)
- Collaborator details and contact rules
- Technical setup (MCP config, GitHub access, Obsidian)
- Notion database schemas and field requirements
- Grant details and strategic notes
- Personal background and profile
- Completed/historical items

### How to reference topic files from MEMORY.md
Add a note at the top:
```markdown
> Topic files (loaded on demand): `collaborators.md`, `setup.md`, `notion-databases.md`, `grants-and-strategy.md`
```

Claude will read these files when it needs the information — e.g., when creating a Notion entry, it reads `notion-databases.md` for the field schema.

## Example MEMORY.md (Index)

```markdown
# Project Memory

> Topic files: `collaborators.md`, `setup.md`, `notion-databases.md`

## Essential Context
- Based in London (GMT). Calendar is CET — always convert.
- Email tone: professional but friendly. Always draft before sending.

## Todo List
- Primary: `~/Projects/Command/todos/work.md`
- Read on session start, update during conversation

## Daily Workflows
- Morning: Calendar → Email → Slack → Todos
- Channels to scan: #project-alpha, #team-general
- Skip in triage: promotions, cold outreach

## Auto-Fill Databases
- Literature Tracker: `collection://xxx` — add papers during Scholar triage
- Decision Log: `collection://yyy` — log decisions proactively

## Active Projects
- **Alpha**: Phase 2 testing, 3 compounds in pipeline
- **Beta**: Protocol design, awaiting lab supplies
```

## Maintenance

### Every 2 weeks
- Audit MEMORY.md for stale content
- Move completed items to topic files or delete them
- Check topic files for outdated info

### Signs your MEMORY.md is too big
- File is over 150 lines
- You see duplicate or contradictory entries
- Historical details (completed grants, old decisions) are still in the index
- Setup instructions that never change are taking up space

### What to delete vs. archive
- **Delete**: Outdated facts, superseded decisions, completed one-off tasks
- **Move to topic file**: Stable reference info (database schemas, collaborator details)
- **Keep in MEMORY.md**: Anything Claude needs to know at the start of *every* session

## Tips

- **Let Claude update it** — Say "remember that X" and Claude will add it to memory
- **Don't duplicate CLAUDE.md** — Memory is for dynamic state; CLAUDE.md is for stable instructions
- **Topic files are free** — They don't cost context until Claude reads them
- **The goal is ~50-80 lines** in MEMORY.md, not zero — some context every session is valuable
