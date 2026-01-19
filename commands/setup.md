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
3. Collects user background for personalized guidance
4. Sets up Claude Memory pointers

## CRITICAL: Get Current Date At Key Points

**You MUST get the current date using Bash tool at these points:**

1. **At command start** - before any output
2. **Before generating milestones** - when calculating quarterly dates
3. **Before creating state.json** - for date fields

Every time you need to reference "today", "this year", or calculate dates:

```bash
date +%Y-%m-%d
```

**NEVER assume the year. ALWAYS run the date command first.**

Example of WRONG behavior:
```
# User says goal is for "end of year"
# Model assumes 2025-12-31 without checking ← WRONG
```

Example of CORRECT behavior:
```
# Before responding about dates, run: date +%Y-%m-%d
# Result: 2026-01-19
# Now use 2026 as current year
```

## Workflow

### Step 1: Get Current Date (REQUIRED)

```
1. Run: date +%Y-%m-%d
2. Store result as TODAY_DATE
3. Use TODAY_DATE for all "[today]" placeholders
```

### Step 2: Check Existing Data

```
1. Check if data/state.json already exists
   - If exists: Ask if user wants to reset or update
   - If not: Fresh setup
```

### Step 3: Collect Goal Information

If not provided as arguments, ask user:

```markdown
## Goal Pilot Setup - Step 1/4: Define Your Goal

1. **Goal Statement**: What do you want to achieve? (Be specific and measurable)
   Example: "Launch SaaS product and reach $10K MRR"

2. **Target Date**: When do you want to achieve this? (YYYY-MM-DD)
   Example: 2026-12-31

3. **North Star Metric**: What single number best measures success?
   Example: "Monthly Recurring Revenue (MRR)"

4. **Language**: Preferred language for responses
   - English / 中文 / 日本語

5. **Timezone**: Your timezone
   Example: Asia/Shanghai, America/New_York
```

### Step 4: Assess Current Situation (NEW - IMPORTANT)

After collecting goal, assess user's starting point:

```markdown
## Goal Pilot Setup - Step 2/4: Your Current Situation

To give you personalized guidance, I need to understand where you're starting from.

### Current Status
1. **Starting Point**: Where are you now relative to your goal?
   - For MRR goal: "Currently $0, have an MVP" or "Already at $2K MRR"
   - For fitness: "Currently 75kg, 25% body fat"
   - For learning: "Beginner level, can read basic texts"

2. **Gap Analysis**: What's the distance between now and goal?
   (I'll help calculate this based on your answers)
```

### Step 5: Assess Resources & Constraints (NEW - IMPORTANT)

```markdown
## Goal Pilot Setup - Step 3/4: Resources & Constraints

### Time Investment
1. **Weekly Hours**: How many hours per week can you dedicate?
   Example: "20 hours/week" or "2 hours/day on weekdays"

2. **Best Work Times**: When are you most productive?
   Example: "Mornings before 10am" or "Evenings after kids sleep"

3. **Schedule Constraints**: Any fixed commitments to work around?
   Example: "Full-time job 9-6", "Kids pickup at 3pm"

### Financial Resources (if applicable)
4. **Budget**: Any budget available for this goal?
   Example: "$500/month for tools and marketing" or "Bootstrap only"

### Support System
5. **Team/Help**: Do you have help or are you solo?
   Example: "Solo founder", "Have a co-founder for tech", "Can hire freelancers"
```

### Step 6: Assess Skills & Experience (NEW - IMPORTANT)

Based on the goal type, ask relevant skill questions:

**For Product/Business Goals:**
```markdown
## Goal Pilot Setup - Step 4/4: Skills Assessment

### Technical Skills (1-5 scale: 1=Beginner, 5=Expert)
- **Development**: Can you build the product yourself?
- **Design**: UI/UX capabilities?
- **DevOps**: Deployment, infrastructure?

### Business Skills (1-5 scale)
- **Marketing**: Content, SEO, paid ads experience?
- **Sales**: Direct selling, cold outreach?
- **Product**: User research, feature prioritization?

### Past Experience
- Have you built/launched products before? Results?
- Any relevant domain expertise?
- Biggest lessons from past attempts?
```

**For Fitness Goals:**
```markdown
## Goal Pilot Setup - Step 4/4: Fitness Background

### Current Habits
- **Exercise Frequency**: How often do you currently exercise?
- **Exercise Types**: What do you do? (gym, running, sports)
- **Diet**: Current eating habits? Any restrictions?

### History
- Past fitness attempts? What worked/didn't?
- Any injuries or health considerations?
- Access to gym/equipment?

### Knowledge (1-5 scale)
- Nutrition knowledge?
- Training program knowledge?
```

**For Learning Goals:**
```markdown
## Goal Pilot Setup - Step 4/4: Learning Background

### Current Level
- **Self-assessment**: Where would you place yourself? (Beginner/Intermediate/Advanced)
- **Specific skills**: What can you already do?
- **Gaps**: What specifically do you struggle with?

### Learning Style
- How do you learn best? (videos, reading, practice, courses)
- Past learning methods tried? Results?

### Resources
- Courses/materials you have access to?
- Native speakers/mentors available?
```

### Step 7: Generate Personalized Milestones

**IMPORTANT: Before generating milestones, run `date +%Y-%m-%d` again to get current date.**

Based on ALL collected information, generate milestones that account for:
- User's starting point (realistic targets)
- Available time (sustainable pace)
- Skill gaps (learning curve factored in)
- Resources (what's achievable with current resources)

```markdown
## Quarterly Milestones (Personalized)

Based on your background:
- Starting from: [current situation]
- Weekly capacity: [X hours]
- Key strength: [identified strength]
- Key gap: [skill to develop]

| Quarter | Phase | Due Date | Milestone | Notes |
|---------|-------|----------|-----------|-------|
| Q1 | Foundation | [date] | [milestone] | Focus on [gap] |
| Q2 | Build | [date] | [milestone] | Leverage [strength] |
| Q3 | Grow | [date] | [milestone] | Scale up |
| Q4 | Achieve | [date] | [milestone] | Final push |

Does this look right? Say "confirm" or suggest changes.
```

### Step 8: Create Data Files

**IMPORTANT: Before creating files, run `date +%Y-%m-%d` again to ensure correct date in state.json.**

After user confirms:

**Create data/state.json:**
```json
{
  "schema_version": "2.1",
  "user": {
    "language": "[user's choice]",
    "timezone": "[user's timezone]"
  },
  "goal": {
    "statement": "[goal statement]",
    "target_date": "[YYYY-MM-DD]",
    "north_star_metric": "[metric]",
    "current_value": "[starting point value]",
    "target_value": "[target value]"
  },
  "background": {
    "current_situation": {
      "starting_point": "[description]",
      "gap_analysis": "[calculated gap]"
    },
    "resources": {
      "weekly_hours": "[number]",
      "best_times": "[description]",
      "constraints": "[description]",
      "budget": "[description]",
      "support": "[description]"
    },
    "skills": {
      "[skill_category]": {
        "[skill_name]": "[1-5 rating]"
      }
    },
    "experience": {
      "past_attempts": "[description]",
      "lessons_learned": "[description]",
      "domain_expertise": "[description]"
    },
    "identified_strengths": ["[strength1]", "[strength2]"],
    "identified_gaps": ["[gap1]", "[gap2]"]
  },
  "plan": {
    "current_phase": "Q1-Foundation",
    "milestones": [
      {
        "id": "M1",
        "phase": "Foundation",
        "due": "[Q1 end date]",
        "definition_of_done": "[milestone description]",
        "focus_areas": ["[area1]", "[area2]"],
        "weight": 0.25,
        "completed_weight": 0.0
      }
    ]
  },
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

### Step 9: Update Claude Memory

Set Memory pointers:

```
GP_DATA_PATH: ./data
GP_LANG: [language]
GP_LAST_SESSION_DATE: [today]
```

### Step 10: Display Summary with Personalized Insights

```markdown
## Setup Complete!

### Your Goal
**Goal**: [statement]
**Target**: [date]
**North Star**: [metric]
**Gap to Close**: [current] → [target]

### Your Profile Summary
- **Weekly Capacity**: [X hours]
- **Key Strength**: [strength] - we'll leverage this
- **Key Gap**: [gap] - Q1 focuses on building this
- **Risk Factors**: [identified risks]

### Personalized Strategy
Based on your background, here's the recommended approach:
1. **Q1 Focus**: [specific focus based on gaps]
2. **Leverage**: [how to use strengths]
3. **Watch Out**: [potential pitfalls based on history]

### Data Files Created
- `data/state.json` - Configuration, plan, and your profile
- `data/reviews_daily.csv` - Daily review records
- `data/reviews_weekly.csv` - Weekly review records
- `data/reviews_monthly.csv` - Monthly review records
- `data/summaries_weekly.csv` - Weekly summaries
- `data/summaries_monthly.csv` - Monthly summaries
- `data/pins.csv` - Long-term knowledge

### Next Steps
1. Run `/goal-pilot:today` to get today's personalized tasks
2. At end of day, run `/goal-pilot:review` to record progress
3. Weekly: `/goal-pilot:review week` for weekly summary

Ready to start! Run `/goal-pilot:today` to begin.
```

## Conversation Flow Tips

When collecting background information:
1. **Be conversational** - Don't dump all questions at once
2. **Adapt questions** - Based on goal type, ask relevant questions only
3. **Acknowledge answers** - Summarize what you heard before moving on
4. **Probe deeper** - If answers are vague, ask follow-up questions
5. **Identify patterns** - Connect past experiences to current plan

Example flow:
```
User: I want to reach $10K MRR
Claude: Great goal! To help you get there, I need to understand your starting point.
        What's your current MRR, if any? And do you have an existing product?
User: Currently $0, I have an MVP
Claude: Got it - starting from zero with an MVP. That's actually a good position.
        How much time can you dedicate weekly? And are you technical enough to
        build/iterate on the product yourself?
...
```

## Error Handling

**state.json already exists:**
```markdown
## Existing Data Found

You already have Goal Pilot data set up. What would you like to do?

1. **View current goal** - Display current state.json
2. **Update background** - Modify your profile/resources
3. **Update milestones** - Modify milestone dates or definitions
4. **Reset completely** - Delete existing data and start fresh

Choose an option (1/2/3/4):
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
