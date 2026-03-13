# Known Issues & Operational Lessons

Hard-won lessons from running Claude Code as a daily operating system. These aren't in the docs and cost hours to figure out.

---

## Gmail / Google Workspace MCP

### `from:` search is unreliable

The `from:` Gmail search operator doesn't work reliably through workspace-mcp. Use `subject:` patterns instead.

```
# Broken — returns inconsistent results
subject:"new results" from:scholaralerts-noreply@google.com is:unread

# Works — match on subject pattern
subject:"new results" is:unread
```

### Archiving requires removing TWO labels

Removing only the `INBOX` label does not fully archive an email. It remains visible in Gmail's Unread view. You must remove both:

```
labels_to_remove: ["INBOX", "UNREAD"]
```

If you only remove `INBOX`, the email disappears from the inbox but still shows up under "Unread" in Gmail, which defeats the purpose of triage.

### Pagination defaults are too low

The default `page_size` is 10. For any bulk operation (triage, search, scanning), always set `page_size: 50` and paginate using `page_token` from the response. Otherwise you'll miss emails and wonder why your triage feels incomplete.

### Port 8000 conflict on restart

After an unclean session termination (crash, force-quit, laptop sleep), stale workspace-mcp processes hold port 8000. Symptoms: MCP connection fails silently or throws a port-in-use error.

Fix:
```bash
ps aux | grep workspace-mcp
kill <stale_pid>
# Then restart your Claude Code session
```

### No official Google MCP

As of early 2026, Google has not released an official MCP server. The most complete community option is [taylorwilsdon/workspace-mcp](https://github.com/taylorwilsdon/workspace-mcp), which covers Gmail, Drive, Docs, Sheets, Slides, and Calendar through a single OAuth flow.

---

## Subagent Reliability

### Write operations should stay in the main context

Subagents (parallel agents spawned for concurrent work) are unreliable for write operations to external services. They may report success on Gmail label modifications, Slack sends, or Notion updates without actually executing them.

**Rule of thumb:**
- Subagents are great for **read** operations: searching emails, scanning Slack channels, fetching Notion pages, gathering data in parallel
- **Write** operations (archiving emails, sending messages, updating databases) should always run in the main conversation context

This bit us repeatedly with Scholar alert archiving — subagents would report "archived 12 emails" but Gmail showed them all still unread.

---

## Memory System

### MEMORY.md has a soft size limit

Only the first ~200 lines of MEMORY.md are loaded into context. Content beyond that is silently truncated — Claude won't know it exists and won't tell you it's missing.

**Mitigations:**
- Keep MEMORY.md lean. Move detailed context into topic files (e.g., `collaborators.md`, `setup.md`, `notion-databases.md`)
- Topic files are free — they consume zero context until Claude explicitly reads them on demand
- Put the most important, frequently-needed context at the top of MEMORY.md

### Don't duplicate between MEMORY.md and CLAUDE.md

CLAUDE.md is loaded every session, guaranteed. MEMORY.md is also loaded but has the size limit above. If something is in CLAUDE.md, putting it in MEMORY.md too wastes precious memory space and creates drift when one copy gets updated but the other doesn't.

- **CLAUDE.md**: Stable instructions, workflows, project structure, safety rules
- **MEMORY.md**: Dynamic state — current blockers, recent decisions, active context that changes week to week

### Stale dates go unnoticed

Dates, version numbers, and "current status" lines in memory files go stale without anyone noticing. Claude won't flag that a date is three weeks old. Don't store temporal information in memory unless it's genuinely actionable (e.g., a deadline, a blocker with a resolution date).

---

## Notion MCP

### Multi-select fields take single values

When writing to multi-select properties, pass each value as a standalone string. Do not comma-separate them.

```
# Wrong
"Workstreams": "AB-03, AB-04"

# Right — pass as array of individual values
"Workstreams": ["AB-03", "AB-04"]
```

### Database writes need collection IDs

You can't write to a Notion database using its page URL. You need the collection/data source ID (format: `collection://uuid`). Get it by fetching the database page first, then extracting the collection ID from the response.

### Person fields need user IDs

Name and person properties require Notion user UUIDs, not display names. Store these in your memory files — you'll need them repeatedly.

```
# Wrong
"Made By": "Daniel"

# Right
"Made By": "86b5aa2c-d58e-4e65-8ed3-85182bcd6eb2"
```

---

## Airtable MCP

### Some field types can't be created via API

The Airtable MCP cannot create count, rollup, or formula field types. If you need computed fields:
- Create regular number or text fields via API
- Compute values externally (in Claude Code or a script) and write the results
- Or create the formula fields manually in the Airtable UI first

### Batch create limit is 10 records

Each `create_record` call accepts a maximum of 10 records. For larger uploads, chunk your data into batches of 10 and call sequentially.

### Watch for invisible whitespace in field names

Fields created through the Airtable UI sometimes have leading or trailing spaces in their names. The API requires exact matches, so `"Treatment"` and `"Treatment "` are different fields. If a write fails with a field-not-found error, fetch the table schema and check for whitespace.

---

## General Claude Code

### Always run `date` at session start

Claude can drift on day-of-week, especially in long sessions. Run `date` at the start of every session to establish the current day and time. Re-run it before any time-sensitive operation (scheduling, deadline checks, meeting prep).

### Don't try to read skill files directly

When using custom skills (e.g., `/upload-vfa`), the skill content is loaded into the conversation context automatically. Don't try to use the Read tool on skill files — it won't find them where you expect, and the content is already available.

### Session hygiene matters

- Start each session by reading your todo list and running `date`
- End each session by updating todos and writing a daily log entry
- If a session crashes, check for stale MCP processes before restarting (see port 8000 issue above)
- Long sessions accumulate context — if Claude starts forgetting earlier instructions, it may be time for a fresh session
