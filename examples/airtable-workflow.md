# Airtable Workflow for Experiment Data

## Why Airtable

Markdown files are great for narrative notes, daily logs, and todos. But when you need structured, queryable data — "show me all experiments where compound X was tested at 25 µM" or "what's the average VFA recovery across all replicates?" — you need a database. Airtable gives us:

- **Linked records**: Experiments → Treatments → Results form a proper relational structure
- **Filtering and sorting**: Find patterns across experiments without writing code
- **Shared access**: The whole team can browse results in a familiar spreadsheet-like UI
- **API/MCP access**: Claude can read and write records programmatically

## Base Structure

We use a single Airtable base for all in-vitro experiment data. The key tables:

### Experiments
One record per experiment run.
| Field | Type | Description |
|-------|------|-------------|
| Experiment ID | Text | e.g., "AB03-W09" (project + week) |
| Date | Date | When the experiment was run |
| Project | Single select | AB-01, AB-02, AB-03, AB-04 |
| Description | Long text | What was being tested and why |
| Status | Single select | Planned, Running, Complete, Analysed |
| Notes | Long text | Observations, issues, deviations from plan |

### Treatments
One record per unique treatment condition. Linked to Experiments.
| Field | Type | Description |
|-------|------|-------------|
| Treatment | Text | e.g., "Rhein 25 µM + AB01" |
| Compound | Text | Compound name |
| Concentration | Number | In µM or mg/mL |
| Solvent | Single select | DMSO, Water, Ethanol, None |
| Experiment | Link | → Experiments table |
| Replicate Count | Number | How many replicates |

### VFA Results
One record per treatment per experiment with calculated values. Linked to Treatments.
| Field | Type | Description |
|-------|------|-------------|
| Treatment | Link | → Treatments table |
| Total VFA (mM) | Number | Sum of all VFA |
| Acetate (mM) | Number | Individual VFA |
| Propionate (mM) | Number | Individual VFA |
| Butyrate (mM) | Number | Individual VFA |
| A:P Ratio | Number | Acetate to propionate ratio |
| % vs Control | Number | Recovery relative to negative control |
| SD | Number | Standard deviation across replicates |

### Gas Results
One record per treatment per experiment. Linked to Treatments.
| Field | Type | Description |
|-------|------|-------------|
| Treatment | Link | → Treatments table |
| CH₄ (mL) | Number | Methane production |
| H₂ (mL) | Number | Hydrogen production |
| Total Gas (mL) | Number | Total gas production |
| CH₄ Reduction (%) | Number | vs control |

## How Claude Uses Airtable

### After Each Experiment (data processing)

```
User: "Process this week's VFA data" [provides Google Sheet link or CSV]

Claude:
1. Reads raw data from the source
2. Parses compound names, concentrations, replicate numbers
3. Calculates treatment means and standard deviations
4. Compares to negative control (% recovery)
5. Creates/updates records in Airtable:
   - Experiment record (if new)
   - Treatment records (one per condition)
   - VFA Result records (linked to treatments)
   - Gas Result records (linked to treatments)
6. Generates a written analysis report
```

### For Analysis (cross-experiment queries)

```
User: "How has Rhein performed across all experiments?"

Claude:
1. Searches Airtable for all treatments containing "Rhein"
2. Pulls VFA and gas results across experiments
3. Calculates trends: is performance consistent? dose-dependent?
4. Compares to other compounds tested at similar concentrations
5. Presents a summary with recommendations
```

### For Experiment Planning

```
User: "What compounds haven't been replicated yet?"

Claude:
1. Queries Airtable for treatments with replicate count = 1
2. Filters for promising results (e.g., VFA recovery > 90%)
3. Suggests which ones to replicate next
4. Generates experiment plan CSV files
```

## Tips for Setting Up

1. **Start simple**: You don't need every field from day one. Start with Experiments and Treatments, add Results tables as you need them.
2. **Use consistent naming**: Compound names, project codes, and status values should be standardized. Claude will follow whatever convention you set.
3. **Link everything**: The power of Airtable is in linked records. Always link treatments to experiments, results to treatments.
4. **Let Claude do the data entry**: Manual data entry into Airtable is error-prone and slow. Process data through Claude and have it write to Airtable via MCP.
5. **Keep narrative in markdown**: Airtable is for structured data. Your interpretation, decisions, and context go in daily logs and notes.

## Common Patterns

### Pattern: Weekly Experiment Cycle
```
Friday:   /experiment → generates CSV files for lab
Monday:   Lab receives files, weighs chemicals
Tuesday:  Experiment runs
Wednesday: Raw data uploaded
Thursday: Claude processes data → Airtable + analysis report
Friday:   Review results → plan next week → repeat
```

### Pattern: Compound Ranking
```
1. Query all treatments for a project from Airtable
2. Join VFA results + gas results + safety data
3. Rank by composite score (efficacy × safety × feasibility)
4. Present as prioritized list with rationale
```

### Pattern: Dose-Response Analysis
```
1. Query all concentrations tested for a specific compound
2. Plot VFA recovery vs. concentration
3. Identify optimal dose range
4. Flag if more concentrations are needed to fill gaps
```
