# Multi-Project Setup: Hub-and-Spoke Architecture

How to organize 6+ projects under Claude Code so they stay independent but coordinated.

---

## The Problem

When you use Claude Code across multiple projects, you quickly hit coordination issues:
- Where do todos live?
- How do you avoid duplicating context across CLAUDE.md files?
- What happens when two projects overlap (e.g., experiment design feeds into data analysis)?
- How do you keep one "source of truth" for your day?

The solution is a hub-and-spoke architecture with one coordination hub and independent domain projects.

---

## Architecture Overview

```
Command Center (~/Projects/Command/)          <-- Daily ops hub
  |-- Morning briefing, email, Slack, todos
  |-- Daily/weekly logs
  +-- Coordinates across all projects

Fermentation-Analysis (~/Projects/Fermentation-Analysis/)  <-- Data processing
  |-- Gas + VFA analysis (merged from separate projects)
  |-- Airtable upload workflows
  |-- Treatment summaries and cross-experiment analysis
  +-- Custom skills: /upload-vfa, /upload-gas, /analyze-experiment

In Vitro Planning (~/Projects/In vitro Planning/)  <-- Experiment design
  |-- Python CSV generator (generate.py + compounds.yaml)
  |-- YAML experiment configs
  |-- Scispot integration
  +-- Custom skill: /experiment

AB04 (~/Projects/AB04/)                       <-- Project knowledge base
  |-- Full project context in CLAUDE.md
  |-- Discovery pipeline outputs
  |-- qPCR quantification planning
  +-- Compound screening results

Jarvis-Review (~/Projects/Jarvis-Review/)     <-- Pipeline validation
  |-- Review Jarvis v8 discovery outputs
  |-- Zoetis mastitis + Elanco liver abscess
  +-- Scientific review prompts in repos

Running Training (~/Projects/running-training/) <-- Personal project
  |-- Half marathon training plan
  |-- TCX file parsing, run logging
  +-- Apple Health MCP integration
```

---

## Key Principles

### 1. Each project owns its own CLAUDE.md

Every project folder has a CLAUDE.md that contains domain-specific instructions: what the project is, what workflows it supports, what conventions to follow. When you open a Claude Code session in that folder, the CLAUDE.md loads automatically and Claude has full context for that domain.

Example: AB04's CLAUDE.md knows about rumen protein metabolism, compound screening criteria, and qPCR primer design. Command Center's CLAUDE.md doesn't need any of that — it just needs to know AB04 exists and what its current status is.

### 2. Each project has its own memory directory

Claude Code stores auto-saved memory in `~/.claude/projects/<project-path>/memory/`. This means each project accumulates its own dynamic state without polluting other projects.

Never duplicate memory content across projects. If AB04 discovers a blocker, it goes in AB04's memory. If that blocker affects the weekly plan, it gets noted in Command Center's todo list — not copied into Command Center's memory verbatim.

### 3. Command Center is the hub

One project serves as the coordination point. It holds:
- **Todo lists** (`todos/work.md`, `todos/lab.md`) — the single source of truth for what needs doing
- **Daily logs** (`daily-log/YYYY-MM-DD.md`) — a record of what happened each day, across all projects
- **Cross-project coordination** — morning briefings, weekly updates, email/Slack triage

You open Command Center first thing in the morning and return to it throughout the day. Domain projects are opened when you need to do focused work in that area.

### 4. MCP servers are shared, not per-project

MCP server configuration lives in `~/.mcp.json` (global) or `~/.claude/mcp.json`. All projects access the same integrations — Gmail, Slack, Notion, Airtable — without per-project MCP setup.

This means any project can read email or post to Slack if needed, but in practice, most external communication flows through Command Center.

### 5. Merging projects is normal

Projects evolve. When two projects converge (e.g., separate Gas-Analysis and VFA-Analysis projects merged into Fermentation-Analysis), handle it cleanly:
- Create the new project with a combined CLAUDE.md
- Move relevant skills and workflows
- Update memory in the old projects to point to the new canonical location
- Update Command Center's CLAUDE.md to reference the new project name

### 6. Memory and CLAUDE.md serve different purposes

This distinction matters and is easy to get wrong:

| | CLAUDE.md | Memory (MEMORY.md + topic files) |
|---|---|---|
| **Content** | Stable instructions, workflows, project structure | Dynamic state, recent decisions, active context |
| **Loaded** | Every session, guaranteed | Every session, but size-limited (~200 lines for MEMORY.md) |
| **Changes** | Rarely — when workflows evolve | Frequently — as work progresses |
| **Edited by** | You (human) | Claude (auto-saved) and you |

If you find the same information in both places, remove it from memory. CLAUDE.md always wins.

---

## How to Set Up a New Project

### Step 1: Create the folder

```bash
mkdir ~/Projects/my-new-project
```

### Step 2: Write a CLAUDE.md

This is the most important step. A good CLAUDE.md includes:
- What the project is (1-2 sentences)
- Key goals or research questions
- Folder structure (if non-obvious)
- Workflows Claude should support (e.g., "when I say X, do Y")
- Conventions (naming, formatting, safety rules)
- Integration details specific to this project

Start minimal. You'll add to it as patterns emerge.

### Step 3: Let memory build naturally

Don't pre-populate memory files. As you work in the project, Claude will auto-save important context — collaborator names, database IDs, recurring patterns, blockers. This organic accumulation is more useful than anything you'd write upfront.

### Step 4: Add custom skills for recurring workflows

If you find yourself giving the same multi-step instructions repeatedly, create a skill:

```
.claude/skills/
  upload-data.md      -- Steps for uploading experiment data to Airtable
  analyze-results.md  -- How to run standard analysis on new results
```

Skills are loaded on demand (e.g., `/upload-data`) and keep your CLAUDE.md from getting bloated with procedural instructions.

### Step 5: Reference shared MCP servers

No per-project MCP configuration needed. Your global `~/.mcp.json` provides access to all integrations. Just reference the tools you need in your CLAUDE.md workflows.

---

## Cross-Project Coordination Patterns

### Work items flow to the hub

When working in a domain project (e.g., AB04) and you discover something that needs follow-up — an email to send, a meeting to schedule, a decision to discuss — it gets added to Command Center's todo list. The domain project doesn't track cross-cutting tasks.

```
# In AB04 session: discover we need to order primers
# -> Add to Command Center's todos/work.md under "Follow Up"

# In Command Center session: morning briefing surfaces the todo
# -> Handle it (send email, update Notion, etc.)
# -> Mark complete
```

### Decision Log captures cross-project decisions

Major decisions (changing a protocol, selecting a compound, choosing an analysis method) get logged in the Notion Decision Log database. This creates a searchable history that spans all projects. Each entry records what was decided, why, who decided, and which project it applies to.

### Daily logs reference domain work

Command Center's daily logs are the diary of your workday. When you spend two hours in Fermentation-Analysis processing VFA data, the daily log captures what you did and what you found — but the detailed analysis stays in the Fermentation-Analysis project.

```markdown
# 2026-03-13 Daily Log

## Morning
- Morning briefing: 3 emails needing response, 2 Slack threads
- Processed AB03 Week 12 VFA data in Fermentation-Analysis
  - All treatments showing expected acetate:propionate ratios
  - Uploaded to Airtable (batch 12a, 12b)

## Afternoon
- AB04: Reviewed qPCR primer options with Tommi
- Decision: Use Eurofins for primer synthesis (logged in Decision Log)
```

---

## Template: Minimal CLAUDE.md for a New Project

```markdown
# Project Name

One-sentence description of what this project is.

## Goals
- Primary goal
- Secondary goal

## Folder Structure
- `data/` — raw and processed data
- `notes/` — working notes and analysis
- `scripts/` — automation scripts

## Workflows

### When I say "analyze [X]"
1. Step one
2. Step two
3. Step three

## Conventions
- Data files: YYYY-MM-DD-description.csv
- Notes: lowercase-with-dashes.md
```

Start here. Expand as you work.
