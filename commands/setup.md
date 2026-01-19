---
description: Initialize Goal Pilot framework. Creates state.json, CSV files, and sets up milestones. Run this first before using other commands.
args: "[goal] [target_date] [domains...]"
examples:
  - "/goal-pilot:setup"
  - "/goal-pilot:setup 年底达到10万MRR 2026-12-31"
  - "/goal-pilot:setup --domains fitness,english"
---

# /goal-pilot:setup - Goal Framework Initialization

## Purpose

Initialize the Goal Pilot data structure and define your goal framework. This command:

1. Creates `data/state.json` with schema
2. Creates empty CSV files with correct headers
3. Sets up Claude Memory pointers

## Workflow

### Step 1: Check Existing Data

```
1. Check if data/state.json already exists
   - If exists: Ask if user wants to reset or update
   - If not: Fresh setup
```

### Step 2: Collect Goal Information

If not provided as arguments, ask user:

```markdown
## Goal Pilot Setup

Let's set up your goal framework. Please provide:

1. **Goal Statement**: What do you want to achieve? (Be specific and measurable)
   Example: "Launch SaaS product and reach $10K MRR"

2. **Target Date**: When do you want to achieve this? (YYYY-MM-DD)
   Example: 2026-12-31

3. **North Star Metric**: What single number best measures success?
   Example: "Monthly Recurring Revenue (MRR)"

4. **Domains** (optional): Any specific areas to track? (comma-separated)
   Example: fitness, english, product

5. **Language**: Preferred language for responses
   - English
   - 中文
   - 日本語

6. **Timezone**: Your timezone
   Example: Asia/Shanghai, America/New_York
```

### Step 3: Create Quarterly Milestones

Based on goal and target date, generate 4 milestones:

```markdown
## Quarterly Milestones

| Quarter | Phase | Due Date | Milestone | Weight |
|---------|-------|----------|-----------|--------|
| Q1 | Validate | [date] | [milestone] | 0.25 |
| Q2 | Scale | [date] | [milestone] | 0.25 |
| Q3 | Systematize | [date] | [milestone] | 0.25 |
| Q4 | Achieve | [date] | [milestone] | 0.25 |

Does this look right? Say "confirm" or suggest changes.
```

### Step 4: Create Data Files

After user confirms:

**Create data/state.json:**
```json
{
  "schema_version": "2.0",
  "user": {
    "language": "[user's choice]",
    "timezone": "[user's timezone]"
  },
  "goal": {
    "statement": "[goal statement]",
    "target_date": "[YYYY-MM-DD]",
    "north_star_metric": "[metric]"
  },
  "plan": {
    "current_phase": "Q1-Validate",
    "milestones": [
      {
        "id": "M1",
        "due": "[Q1 end date]",
        "definition_of_done": "[milestone description]",
        "weight": 0.25,
        "completed_weight": 0.0
      },
      // ... M2, M3, M4
    ]
  },
  "domains": ["[domain1]", "[domain2]"],
  "calibration": {
    "decay": {
      "daily_half_life_days": 30,
      "weekly_half_life_days": 120
    },
    "thresholds": {
      "behind_ratio_yellow": 0.15,
      "behind_ratio_red": 0.30
    },
    "task_adjustment": {
      "force_small_steps": false,
      "force_split_outcomes": false,
      "low_friction_mode": false
    },
    "deviation_counters": {
      "unclear_next_action": 0,
      "scope_too_big": 0,
      "low_energy": 0,
      "last_reset_date": "[today]"
    }
  },
  "last_run": {
    "today": "[today]",
    "last_review": "",
    "last_calibration": ""
  }
}
```

**Create CSV files with headers:**
- `data/reviews_daily.csv`
- `data/reviews_weekly.csv`
- `data/reviews_monthly.csv`
- `data/summaries_weekly.csv`
- `data/summaries_monthly.csv`
- `data/pins.csv`

### Step 5: Update Claude Memory

Set Memory pointers:

```
GTD_DATA_PATH: ./data
GP_LANG: [language]
GP_LAST_SESSION_DATE: [today]
```

### Step 6: Display Summary

```markdown
## Setup Complete!

**Goal**: [statement]
**Target**: [date]
**North Star**: [metric]
**Current Phase**: Q1-Validate

### Data Files Created
- `data/state.json` - Configuration and plan
- `data/reviews_daily.csv` - Daily review records
- `data/reviews_weekly.csv` - Weekly review records
- `data/reviews_monthly.csv` - Monthly review records
- `data/summaries_weekly.csv` - Weekly summaries
- `data/summaries_monthly.csv` - Monthly summaries
- `data/pins.csv` - Long-term knowledge

### Next Steps
1. Run `/goal-pilot:today` to get today's tasks
2. At end of day, run `/goal-pilot:review` to record progress
3. Weekly: `/goal-pilot:review week` for weekly summary
4. Monthly: `/goal-pilot:review month` for monthly summary

Ready to start! Run `/goal-pilot:today` to begin.
```

## Error Handling

**state.json already exists:**
```markdown
## Existing Data Found

You already have GTD data set up. What would you like to do?

1. **View current goal** - Display current state.json
2. **Update milestones** - Modify milestone dates or definitions
3. **Reset completely** - Delete existing data and start fresh

Choose an option (1/2/3):
```

**File creation fails:**
```markdown
## Setup Error

Could not create data files. Please check:
1. The `data/` directory exists
2. You have write permissions
3. Disk has sufficient space

Error: [specific error message]
```
