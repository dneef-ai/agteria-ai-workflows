# AI Workflows at Agteria

How we use Claude Code as a daily operating system for R&D work — managing email, Slack, experiments, data, and knowledge from a single terminal session.

*Daniel Neef, March 2026*

---

## The Big Picture

The core principle: **AI should reduce context-switching and busywork, not add another tool to check.**

The approach that works best is consolidating multiple daily operations into a single AI-powered session, rather than using AI as a one-off assistant for isolated tasks. We call this the "Command Center" — open it in the morning, run through your briefing, keep it open through the day, close it at night. Each session picks up where the last left off via persistent memory.

### The Stack
- **Claude Code** (CLI) as the primary interface — runs in a terminal, stays open all day
- **MCP servers** connecting Claude to Slack, Gmail, Google Calendar, Google Drive, Notion, and Airtable
- **Persistent memory** so each session picks up where the last one left off
- **Custom skills** (slash commands) for recurring workflows
- **Local markdown files** for notes, todos, and logs that AI reads and updates throughout the day

---

## What's In This Repo

```
agteria-ai-workflows/
├── README.md                          # This file
├── CLAUDE.md.example                  # Template project instructions for Claude Code
├── skills/
│   ├── morning/SKILL.md               # /morning — daily briefing
│   ├── closeout/SKILL.md              # /closeout — end-of-day wrap-up
│   ├── weekly/SKILL.md                # /weekly — weekly summary + reporting
│   └── experiment/SKILL.md            # /experiment — experiment planning + CSV generation
└── examples/
    ├── memory-template.md             # How to set up persistent memory
    ├── mcp-setup-guide.md             # How to connect integrations
    └── airtable-workflow.md           # How we use Airtable for experiment data
```

---

## 1. Daily Operations — The "Command Center"

### What It Replaces
Opening Slack, Gmail, Google Calendar, and Notion separately every morning. Manually scanning channels, triaging emails, checking the calendar, and writing down what needs doing.

### What It Does
One command — `/morning` — triggers a sweep of all platforms:

- **Google Calendar**: Pulls today's schedule, converts timezones, flags meetings needing prep
- **Gmail**: Checks unread emails, identifies spam/junk and archives it, flags urgent items
- **Google Scholar Alerts**: Triages research papers for relevance, logs relevant ones to a Notion database, archives the alert emails
- **Slack**: Scans key channels and DMs for unread messages, mentions, and threads needing response
- **Notion**: Checks for meeting notes, extracts action items
- **Todo list**: Surfaces current priorities, adds new items from the sweep

The whole process takes about 2 minutes and produces a prioritized action list for the day.

### End of Day (`/closeout`)
Reviews what was accomplished, writes a daily log entry, checks for missed items, logs decisions to Notion, and updates the todo list.

### End of Week (`/weekly`)
Reads all daily logs, synthesizes into project-level weekly summaries, posts to a shared Notion database, backfills any missing daily logs, and reviews project memory for stale information.

### What This Looks Like in Practice
```
Me: /morning

AI: [Scans Calendar, Gmail, Slack, Notion in parallel — ~2 min]

## Calendar (today)
- 10:00 GMT — Team standup
- 14:00 GMT — Experiment review with lab team

## Email
- Henrik confirmed grant submission. Results in June. No action needed.
- Martin replied to experiment protocol — 3 suggestions. Needs response.

## Scholar Alerts
- Added 3 papers to Literature Tracker (2× AB-03, 1× AB-04)
- Skipped 17 irrelevant papers
- All 4 Scholar alert emails archived.

## Junk Mail (archived)
- (none today)

## Slack
- #ab04-nitrogen: Lab team reports protozoa viability issues in batch culture
- No unread DMs or mentions

## Today's Focus
1. Reply to Martin on experiment protocol
2. Address protozoa viability finding — changes experiment design
3. Provide feedback on Scispot dashboards (overdue)

Updated todos/work.md with 2 new items.
```

---

## 2. Email Management

### Inbox Cleanup
The `/morning` skill automatically:
- Identifies junk mail, cold outreach, and spam
- Reports what it found (so you know what was cleared)
- Archives it out of your inbox

Over time, you build a list of known spam senders in the skill config. Result: your inbox only contains things that matter.

### Scholar Alert Triage
We get 5-10 Google Scholar alerts per day across multiple research topics. AI reads all alerts, evaluates relevance to active projects, and produces a ranked summary. Relevant papers are automatically logged to a Notion Literature Tracker database with project tags, key findings, and relevance ratings. All alert emails are then archived.

**Result**: 39 Scholar alerts triaged in one session. 16 relevant papers identified and catalogued. Would have taken 2+ hours manually — took about 5 minutes.

### Email Drafting
All outbound email goes through a draft-review-send workflow:
1. AI drafts the email based on context (it has access to the full conversation thread)
2. You review and approve (or edit)
3. AI sends and logs the action

This has been used for collaboration responses, funding applications, meeting scheduling, and supplier communications.

---

## 3. Experiment Data in Airtable

### What We Track
Airtable serves as our structured data layer for experiment results. While daily notes and logs live in markdown files, structured quantitative data goes into Airtable where it can be filtered, sorted, and cross-referenced.

We use it for:
- **Experiment records**: Each in-vitro run gets a record with date, conditions, compounds tested, and status
- **Treatment results**: VFA concentrations, gas measurements (H₂, CH₄), pH, redox potential — all linked to the parent experiment
- **Treatment summaries**: Calculated means, standard deviations, and normalized ratios across replicates
- **Compound tracking**: What we've tested, at what concentrations, and what the outcomes were

### How Claude Interacts with Airtable
Claude can read and write to Airtable via MCP. The typical workflow:

1. **After an experiment**: Raw data (e.g., from GC analysis) is processed by Claude
2. **Validation**: Claude parses the data, checks for outliers or missing values
3. **Calculation**: Means, standard deviations, treatment-vs-control comparisons
4. **Upload**: Structured results are written to Airtable as linked records
5. **Analysis report**: Claude generates a written summary referencing the Airtable data

### Why Airtable (Not Just Markdown)
Markdown is great for narrative notes and logs. But when you need to ask "which compound gave the best VFA recovery across all experiments?" or "show me dose-response data for compound X" — you need structured, queryable data. Airtable gives us:
- Linked records (experiment → treatments → results)
- Filtering and sorting across experiments
- A shared interface the whole team can browse
- API access so Claude can read/write programmatically

### Scale
To date: 9 experiments, 143 VFA result records, 94 unique treatments processed through this workflow. Each new experiment takes ~10 minutes to fully process and upload.

---

## 4. Scientific Analysis

### Multi-Agent Safety Sweeps
When we needed to assess the safety profile of ~40 molecules, AI ran a systematic 20-agent sweep:
- Each agent evaluated a subset of molecules against toxicological databases, regulatory status, and published safety data
- Results were consolidated into a single report: 11 Favorable, 5 Conditional, 18 Disqualified
- Key insight discovered: a reactivity-toxicity paradox (best performance correlated with highest genotoxicity due to redox cycling)

This analysis would have taken days of literature searching. It was completed in one session.

### Literature Deep Dives
AI reads full papers, cross-references findings against existing experimental data, identifies untested pathways, and produces tiered experiment priority lists. One analysis of a metagenomics paper identified 6 untested pathways that are now driving our next experiments.

### Cross-Experiment Pattern Analysis
After processing VFA data from multiple experiments, AI identified two distinct mechanism classes that weren't obvious from individual experiment results — they emerged from cross-experiment pattern analysis across the Airtable data.

### Candidate Ranking
AI produces comprehensive rankings of candidate molecules by combining:
- Efficacy data (from Airtable)
- Safety classification
- Dose-response patterns
- Mechanistic fit

---

## 5. Experiment Planning (`/experiment`)

### In-Vitro Template Generation
AI generates experiment setup files from a description of what you want to test:
- Calculates stock solution concentrations and pipetting volumes
- Handles unit conversions
- Produces CSV files that can be directly imported into LIMS (Scispot)
- Drafts a Slack message for the lab team with instructions

### The Weekly Cadence
- **Friday**: Review this week's results, design next week's experiment
- **Monday 12:00**: Deadline — send CSV files + instructions to lab
- **Tuesday**: Lab runs experiment
- **Wednesday**: Analyses + data upload
- **Thursday**: Review meeting
- **Friday**: VFA data arrives, plan next week → cycle repeats

The `/experiment` skill handles the Friday planning and Monday deliverable.

---

## 6. Knowledge Management

### Persistent Memory
Claude Code's memory system means you explain things once. Project context, collaborator details, preferences, and decisions persist across sessions. See `examples/memory-template.md` for how to set this up.

### Decision Logging
Every significant decision made during a session is automatically logged to a Notion database: what was decided, why, which project it affects, and where it was discussed. When someone asks "why did we choose approach X?" there's a searchable record with context.

### Notion Organization
AI can analyze and reorganize a Notion workspace — identifying structural problems, creating logical hierarchies, and moving pages into appropriate locations. We reorganized ~80 pages from a flat dump into project-based folders in one session.

### Weekly Notion Updates
The `/weekly` skill reads all daily logs, synthesizes project-level summaries, and posts them to a shared Notion database. It also checks for missing entries from previous weeks and offers to backfill.

---

## 7. Communication

### Slack Integration
- Read and summarize channel activity across multiple channels
- Draft and send messages (always with confirmation before sending)
- Search across public and private channels
- Post formatted updates with data summaries

### Meeting Management
- Check calendar and flag meetings needing prep
- Auto-import meeting notes (Gemini, Fireflies, Notion)
- Extract action items and add to todo list
- Convert between timezones automatically

---

## What Works Best

1. **Consolidation over fragmentation**: One AI session touching multiple tools beats multiple AI tools each doing one thing.
2. **Persistent memory**: AI that remembers your projects, preferences, and decisions across sessions is dramatically more useful than starting fresh each time.
3. **Draft-then-confirm for outbound communication**: Never let AI send emails or Slack messages without review. This builds trust and catches tone issues.
4. **Cross-referencing**: AI's biggest strength is connecting information across sources — a paper finding + experimental data + safety data + regulatory status = actionable insight.
5. **Structured data where it matters**: Use markdown for narrative, Airtable for quantitative data that needs querying.
6. **Let AI do the boring parts**: Data reformatting, timezone conversion, supplier searching, inbox scanning. Save human attention for decisions and interpretation.

## What Doesn't Work

1. **AI as a replacement for domain expertise**: AI accelerates analysis but doesn't replace understanding the science. You still need to know what questions to ask.
2. **Fully autonomous operation**: The draft-review-confirm pattern exists for a reason. AI makes mistakes, especially with names, numbers, and nuance.
3. **Over-engineering**: The best AI workflows are simple. "Read my email and tell me what matters" beats a complex automation pipeline every time.

---

## Getting Started

1. **Install Claude Code** and get it running in your terminal
2. **Set up MCP servers** for the tools you use daily (see `examples/mcp-setup-guide.md`)
3. **Copy `CLAUDE.md.example`** into your project folder and customize it
4. **Copy the skills** you want into `.claude/skills/` and customize the marked sections
5. **Set up persistent memory** (see `examples/memory-template.md`)
6. **Start with `/morning`** and build from there

The skills in this repo are templates — they're meant to be adapted to your role and workflow. The `<!-- CUSTOMIZE -->` comments mark the sections you need to change.

---

*This document was written collaboratively with AI based on two months of daily session logs, experiment data, and persistent memory — a meta-example of the workflow in action.*
