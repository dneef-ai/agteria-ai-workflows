---
name: morning
description: "Morning briefing — run when user says 'morning briefing', 'morning start', 'good morning', or similar. Checks Calendar, Email, Slack, Notion, and updates todos."
disable-model-invocation: false
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, Agent
argument-hint: "[--quick]"
---

# Morning Briefing

Run `date` first to establish current day and time. All times presented in GMT.

## Quick Mode

If `$ARGUMENTS` contains `--quick`, skip Steps 2-4 and only do Calendar + Todos (Steps 1 and 5).

## Step 1: Calendar

- Fetch today's Google Calendar events using `mcp__google-workspace__get_events`
- Use `user_google_email`: `YOUR_EMAIL@company.com`  <!-- CUSTOMIZE -->
- Convert all times to user's local timezone
- Flag meetings needing prep
- Note any scheduling conflicts
- Check if any meetings have already passed — if so, trigger post-meeting auto-check in Step 4

## Step 2: Email

- Search Gmail for unread/new emails since last session using `mcp__google-workspace__search_gmail_messages`
- Fetch full content of all unread emails using `mcp__google-workspace__get_gmail_messages_content_batch`
- Triage into three categories:

### 2a: Junk / Spam Identification
- Identify junk mail, spam, newsletters user didn't subscribe to, cold outreach, and irrelevant marketing
- **Always junk**: [add known spam senders here]  <!-- CUSTOMIZE -->
- **Not junk**: emails from team members, collaborators, known contacts, active tool vendors
- List identified junk in the briefing summary (sender + subject, one line each)
- Archive all junk emails using `mcp__google-workspace__batch_modify_gmail_message_labels` with `remove_label_ids: ["INBOX"]`

### 2b: Google Scholar Alerts (optional — for research teams)
- **Scholar alerts must ALWAYS be triaged. No alert should ever go untriaged.**
- Search for Scholar alerts using `subject:"new results" is:unread` (do NOT use `from:` — it doesn't work reliably with some MCP servers)
- Filter papers for relevance to active projects
- For each relevant paper, add to a tracking database (e.g., Notion Literature Tracker) with:
  - Paper: title
  - Project: tag
  - Key Finding: one-line summary
  - Source: `Scholar Alert`
  - Relevance: `High`, `Medium`, or `Low`
  - Notes: author, year, journal, brief context
- List relevant papers in the briefing summary
- Archive AND mark as read ALL Scholar alert emails using `mcp__google-workspace__batch_modify_gmail_message_labels` with `remove_label_ids: ["INBOX", "UNREAD"]`
- **IMPORTANT: The archive step MUST be executed directly in the main context, NOT delegated to subagents.** Subagents may report success without actually performing the archive. Triage and relevance filtering can be parallelized, but the final `batch_modify_gmail_message_labels` call must happen in the main conversation.

### 2c: Actionable Email
- Flag urgent emails, identify responses needed
- Present actionable emails in the briefing summary

## Step 3: Slack

- Check DMs and mentions (last 24h) using Slack MCP tools
- Scan key channels: [list your channels here]  <!-- CUSTOMIZE -->
- Flag items needing response

## Step 4: Notion (optional)

- Check for new meeting notes in Meetings folder
- Check for updates/comments on pages user owns
- For any meetings that have already passed (from Step 1):
  - Search Gmail for auto-generated meeting notes (e.g., Gemini, Otter, Fireflies)
  - Check Notion for matching notes
  - Extract action items

### 4b: Meeting Notes Cleanup  <!-- CUSTOMIZE: set your Meetings folder ID -->
- Search Notion for pages containing "meeting notes", "notes:", or meeting-related content that are NOT in the Meetings folder
- If any orphaned meeting notes are found, move them using `mcp__notion__notion-move-pages` to the Meetings folder
- Report any moved pages in the briefing summary

## Step 5: Todos

- Read your todo list file (e.g., `~/Projects/Command/todos/work.md`)  <!-- CUSTOMIZE path -->
- Add new items from email, Slack, calendar, meeting notes
- Surface current priorities
- Present today's focus items

## Parallelization

Steps 1-4 are independent — run them in parallel using subagents where possible, then synthesize results in Step 5.

## Output Format

Present a structured summary:

```
## Calendar (today)
- [events with times in local timezone]

## Email
- [urgent/actionable items]

## Scholar Alerts (if applicable)
- [relevant papers added to tracker — title, project, relevance]
- [count of skipped papers]
- "All Scholar alert emails archived."

## Junk Mail (archived)
- [sender — subject, one line each]
- "N emails archived."

## Slack
- [DMs and mentions needing response]
- [channel highlights]

## Meeting Notes
- [any imported notes + action items]

## Today's Focus
- [top priority items from todos]
```
