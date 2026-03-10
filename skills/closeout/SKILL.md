---
name: closeout
description: "Daily closeout — run at end of day. Writes daily log, checks for missed items, imports meeting notes, logs decisions, updates todos and project memory."
disable-model-invocation: false
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, Agent
---

# Daily Closeout

Run `date` first to establish current day and time.

## Step 1: Write Daily Log

- Create/update `~/Projects/Command/daily-log/YYYY-MM-DD.md` for today  <!-- CUSTOMIZE path -->
- Pull context from:
  - Calendar: meetings held today
  - Gmail: sent + received emails today
  - Slack: user's messages and key channel activity
  - Session context: any work done during this conversation
- Standard sections:

```markdown
# YYYY-MM-DD — Day of Week

## Meetings
- [meeting name] — [key outcomes/decisions]

## Email & Comms
- [sent/received summary]

## Scientific / Project Work
- [any project work, analysis, planning]

## Slack
- [key conversations, decisions]

## Decisions Made
- [significant decisions with rationale]

## Todo Status
- Completed: [items done]
- Added: [new items]
- Blocked: [items waiting on something]
```

## Step 2: Last Check — Email & Slack

- Scan Gmail for anything new/unread since morning briefing
- **Scholar alert triage (MANDATORY):** Search for any untriaged Google Scholar alerts (from: scholaralerts-noreply@google.com) still in inbox. No Scholar alert should ever go untriaged. For each:
  - Filter papers for relevance to active projects
  - Add relevant papers to your literature tracking database
  - Archive ALL Scholar alert emails (remove INBOX label)
  - **IMPORTANT: The archive step MUST be executed directly, NOT delegated to subagents.** Subagents may report success without actually archiving.
- Scan Slack for unread DMs, mentions, important channel activity
- Flag anything urgent or needing response — add to todos if needed

## Step 3: Meeting Notes

- Check today's calendar for meetings that occurred
- For each meeting:
  - Search Gmail for auto-generated meeting notes (Gemini, Otter, Fireflies, etc.)
  - Check Notion or other note-taking tools for matching notes
- Extract action items from any meeting notes found
- Add action items to todos

## Step 4: Decision Log (optional)

- Review the session for significant decisions:
  - Approved compounds, experiment designs, purchases
  - Changed project direction or approach
  - Parked tasks or deferred decisions
  - Chose between approaches
- Log each decision to a tracking database (e.g., Notion Decision Log)
  - Fields: Decision (title), Project, Rationale, Date, Status, Made By, Context

## Step 5: Update Project Memory

- Scan project memory files for anything that needs updating based on today's work:
  - `~/.claude/projects/*/memory/`  <!-- CUSTOMIZE path -->
  - Check for stale, outdated, or incorrect information
  - Update any facts, status, or context that changed today
  - Remove or correct any outdated information

## Step 6: Update Todo List

- Read your todo list file
- Mark completed items from today
- Add new items surfaced from email, Slack, meeting notes, or session work
- Reorder/reprioritize as needed
- Present updated todo list

## Parallelization

Steps 2 and 3 can run in parallel. Steps 4 and 5 can run in parallel after Steps 1-3 complete. Step 6 runs last.
