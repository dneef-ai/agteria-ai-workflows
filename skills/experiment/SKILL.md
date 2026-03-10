---
name: experiment
description: "Experiment planning — generates setup files and instructions for lab team. Adapt to your experiment type."
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, Agent
argument-hint: "[compound list or 'same as last week' or 'review last results']"
---

# Experiment Planning

Run `date` first. Set your team's deadline.  <!-- CUSTOMIZE deadline -->

## Context

Adapt this skill to your experiment type. The example below is for in-vitro batch fermentation experiments, but the pattern works for any recurring experiment workflow where:
- One person designs the experiment
- Another person runs it
- Setup files need to be generated consistently
- Results from the previous run inform the next one

## Step 1: Review Previous Results

- Check for last experiment's results:
  - Data uploads (e.g., VFA data, sequencing data, assay results)
  - Any notes from review meetings
- Search Slack channels for recent data/discussion
- Check your LIMS or data management system for uploaded results

## Step 2: Design Next Experiment

Based on `$ARGUMENTS`:

**If "same as last week"**: Duplicate previous experiment layout
**If "review last results"**: Analyze last results and suggest modifications
**If specifics provided**: Design new experiment with specified parameters
**If no arguments**: Ask what the focus is

For each experiment, define:
- **Compounds/treatments**: what is being tested
- **Concentrations/doses**: range for each treatment
- **Controls**: negative control, positive control if applicable
- **Replicates**: number of replicates per condition
- **Sample assignments**: map samples to vessel/plate positions
- **Analyses**: which measurements to run

## Step 3: Generate Setup Files

Generate the files your lab team needs. Example CSVs:

### compounds.csv
```
vessel_number,compound,concentration,solvent,notes
1,Control,0,none,negative control
2,CompoundA,10uM,DMSO,
3,CompoundA,50uM,DMSO,
...
```

### vessel_assignments.csv
```
vessel_number,rack,position,compound,concentration,replicate
1,A,1,Control,0,1
2,A,2,CompoundA,10uM,1
...
```

Save to a dated folder: `experiment-plan-YYYY-MM-DD/`  <!-- CUSTOMIZE path -->

## Step 4: Draft Communication

Draft a message for your lab team including:
- This experiment's focus (1-2 sentences)
- Which analyses to run
- Any special instructions (different incubation time, temperature, etc.)
- Note any materials that need special handling

Create as a draft for review — do NOT send without confirmation.

## Step 5: Confirm

Present the full plan:
- Experiment design summary
- Generated file locations
- Draft message preview
- Reminder of deadline
