---
name: weekly
description: "Weekly closeout — run at end of week. Backfills daily logs, creates weekly summary entries, reviews project memory."
disable-model-invocation: false
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, Agent
---

# Weekly Closeout

Run `date` first to establish current day, time, and ISO week number.

## Step 0: Recurring Deliverables Check

- Confirm any recurring weekly deliverables are ready
- Flag anything overdue or at risk as HIGH PRIORITY
- Examples: experiment plans, weekly reports, client deliverables  <!-- CUSTOMIZE -->

## Step 1: Backfill Daily Logs

1. Check your daily log directory for missing days since last log entry
2. For each missing day, gather context from:
   - Calendar: events/meetings that day
   - Gmail: sent + received emails
   - Slack: user's messages and key activity
   - Project memory files
3. Create daily log files in standard format (see `/closeout` skill for template)
4. Present backfilled logs for review

## Step 2: Weekly Summary

1. Read ALL daily logs for the week
2. Read ALL project memory files
3. Synthesize into ~5 bullets "this week" + ~5 bullets "next week" per project/workstream
4. Present drafts for review before pushing to any external system

### Posting to a shared database (optional)

If your team uses a shared weekly update database (e.g., Notion):
- Create one entry per project/workstream
- Common fields: Week number, Name, Date range, Status, Summary, Next Week's Focus
- Use your team's format and conventions  <!-- CUSTOMIZE -->

## Step 3: Backfill Check

- After creating entries, check the previous week for missing entries
- Flag any gaps and offer to backfill from daily logs and memory

## Step 4: Decision Log Review

- Review the week's daily logs and session history for significant decisions
- Cross-reference with your decision tracking system
- Log any decisions that weren't captured during daily closeouts

## Step 5: Update Project Memory

- Thorough review of ALL project memory files:
  - Check for stale, outdated, or incorrect information
  - Update status of ongoing items (blockers resolved, tasks completed, etc.)
  - Remove completed items that no longer need tracking
  - Add any new persistent context from the week
- Flag any memory entries that seem wrong or contradictory for user to confirm

## Step 6: Present Summary

Present:
- Weekly highlights (what was accomplished)
- Outstanding items carrying into next week
- Any blockers or concerns
- Next week's priorities
- Status of recurring deliverables
