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

## CRITICAL: Use AskUserQuestion Tool

**You MUST use the `AskUserQuestion` tool to collect user input during setup.**

Do NOT simply output markdown prompts and wait for user response. Instead, use AskUserQuestion with appropriate options for each step.

Example:
```
AskUserQuestion({
  questions: [{
    header: "Language",
    question: "Preferred language for Goal Pilot?",
    options: [
      { label: "English", description: "English responses" },
      { label: "中文", description: "Chinese responses" },
      { label: "日本語", description: "Japanese responses" }
    ],
    multiSelect: false
  }]
})
```

This ensures a structured, interactive setup experience.

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

## Workflow (MUST FOLLOW IN ORDER)

**CRITICAL: You MUST complete each phase before proceeding to the next. Do NOT skip any phase.**

### Phase 0: Initialize

```
1. Run: date +%Y-%m-%d → Store as TODAY_DATE
2. Check if data/state.json exists
   - If exists: Ask if user wants to reset or update
   - If not: Proceed to Phase 1
```

---

### Phase 1: Basic Info (REQUIRED - DO NOT SKIP)

**You MUST call AskUserQuestion and wait for response before proceeding.**

```
AskUserQuestion({
  questions: [
    {
      header: "Goal Type",
      question: "What type of goal are you working towards?",
      options: [
        { label: "Product/Business", description: "SaaS, startup, revenue goals" },
        { label: "Fitness", description: "Weight loss, muscle gain, health" },
        { label: "Learning", description: "Language, skills, certifications" },
        { label: "Career", description: "Promotion, job change, skills" }
      ],
      multiSelect: false
    },
    {
      header: "Timeline",
      question: "When do you want to achieve this goal?",
      options: [
        { label: "3 months", description: "Short-term intensive" },
        { label: "6 months", description: "Medium-term focused" },
        { label: "12 months", description: "Full year plan" }
      ],
      multiSelect: false
    },
    {
      header: "Language",
      question: "Preferred language for Goal Pilot?",
      options: [
        { label: "English", description: "English responses" },
        { label: "中文", description: "Chinese responses" },
        { label: "日本語", description: "Japanese responses" }
      ],
      multiSelect: false
    }
  ]
})
```

**After receiving response**: Store Goal Type, Timeline, Language. Then proceed to Phase 2.

---

### Phase 2: Goal Details (REQUIRED - DO NOT SKIP)

**You MUST call AskUserQuestion and wait for response before proceeding.**

Ask user to describe their specific goal (they will use "Other" to input free text):

```
AskUserQuestion({
  questions: [
    {
      header: "Goal Statement",
      question: "Describe your specific goal (be measurable, e.g. 'Reach $10K MRR' or 'Lose 10kg')",
      options: [
        { label: "Example: Reach $10K MRR", description: "Revenue target" },
        { label: "Example: Lose 10kg", description: "Weight loss target" },
        { label: "Example: Pass JLPT N2", description: "Certification target" }
      ],
      multiSelect: false
    }
  ]
})
```

**After receiving response**: Store Goal Statement. Then proceed to Phase 3.

---

### Phase 3: Time Budget (REQUIRED - DO NOT SKIP)

**You MUST call AskUserQuestion and wait for response before proceeding.**

```
AskUserQuestion({
  questions: [
    {
      header: "Daily Hours",
      question: "How many hours PER DAY can you dedicate to this goal?",
      options: [
        { label: "30 minutes", description: "Minimal, squeezed from busy schedule" },
        { label: "1 hour", description: "Focused daily session" },
        { label: "2 hours", description: "Significant daily commitment" },
        { label: "3+ hours", description: "Major time investment" }
      ],
      multiSelect: false
    },
    {
      header: "Best Time",
      question: "When are you most productive?",
      options: [
        { label: "Morning (6-12)", description: "Early bird" },
        { label: "Afternoon (12-18)", description: "Midday focus" },
        { label: "Evening (18-24)", description: "Night owl" },
        { label: "Flexible", description: "Varies by day" }
      ],
      multiSelect: false
    }
  ]
})
```

**After receiving response**: Store Daily Hours, Best Time. Then proceed to Phase 4.

---

### Phase 4: Current Situation (REQUIRED - DO NOT SKIP)

**You MUST call AskUserQuestion and wait for response before proceeding.**

```
AskUserQuestion({
  questions: [
    {
      header: "Starting Point",
      question: "Where are you now relative to your goal?",
      options: [
        { label: "Complete beginner", description: "Just starting, no prior experience" },
        { label: "Some experience", description: "Have tried before, know basics" },
        { label: "Intermediate", description: "Solid foundation, need to scale" },
        { label: "Advanced", description: "Close to goal, need final push" }
      ],
      multiSelect: false
    },
    {
      header: "Support",
      question: "Do you have help or working solo?",
      options: [
        { label: "Solo", description: "Working alone" },
        { label: "Partner", description: "Have a partner/co-founder" },
        { label: "Team", description: "Have a team" },
        { label: "Can hire", description: "Budget for help" }
      ],
      multiSelect: false
    }
  ]
})
```

**After receiving response**: Store Starting Point, Support. Then proceed to Phase 5.

---

### Phase 5: Skills Assessment (REQUIRED - DO NOT SKIP)

**You MUST call AskUserQuestion and wait for response before proceeding.**

Based on Goal Type from Phase 1, ask the appropriate skill questions:

**If Goal Type = Product/Business:**
```
AskUserQuestion({
  questions: [
    {
      header: "Tech Skills",
      question: "What's your technical capability?",
      options: [
        { label: "Non-technical", description: "Need to hire for dev" },
        { label: "Basic", description: "Simple tasks only" },
        { label: "Intermediate", description: "Can build most features" },
        { label: "Expert", description: "Full-stack capable" }
      ],
      multiSelect: false
    },
    {
      header: "Strengths",
      question: "Which areas are you strongest in?",
      options: [
        { label: "Marketing", description: "Content, SEO, ads" },
        { label: "Sales", description: "Outreach, closing" },
        { label: "Product", description: "User research" },
        { label: "Operations", description: "Process, systems" }
      ],
      multiSelect: true
    }
  ]
})
```

**If Goal Type = Fitness:**
```
AskUserQuestion({
  questions: [
    {
      header: "Exercise Level",
      question: "How often do you currently exercise?",
      options: [
        { label: "Rarely", description: "< 1x per week" },
        { label: "Sometimes", description: "1-2x per week" },
        { label: "Regular", description: "3-4x per week" },
        { label: "Daily", description: "5+ per week" }
      ],
      multiSelect: false
    },
    {
      header: "Equipment",
      question: "What equipment do you have?",
      options: [
        { label: "None", description: "Bodyweight only" },
        { label: "Home gym", description: "Basic equipment" },
        { label: "Gym membership", description: "Full access" }
      ],
      multiSelect: false
    }
  ]
})
```

**If Goal Type = Learning:**
```
AskUserQuestion({
  questions: [
    {
      header: "Current Level",
      question: "What's your current skill level?",
      options: [
        { label: "Beginner", description: "Just starting" },
        { label: "Elementary", description: "Know basics" },
        { label: "Intermediate", description: "Can handle common tasks" },
        { label: "Advanced", description: "Refining skills" }
      ],
      multiSelect: false
    },
    {
      header: "Learning Style",
      question: "How do you learn best?",
      options: [
        { label: "Video", description: "Courses, tutorials" },
        { label: "Reading", description: "Books, docs" },
        { label: "Practice", description: "Hands-on projects" },
        { label: "Mentorship", description: "1-on-1 guidance" }
      ],
      multiSelect: true
    }
  ]
})
```

**If Goal Type = Career:**
```
AskUserQuestion({
  questions: [
    {
      header: "Experience",
      question: "Years of experience in target field?",
      options: [
        { label: "< 1 year", description: "Just starting" },
        { label: "1-3 years", description: "Early career" },
        { label: "3-7 years", description: "Mid career" },
        { label: "7+ years", description: "Senior level" }
      ],
      multiSelect: false
    },
    {
      header: "Gap",
      question: "What's your biggest gap to close?",
      options: [
        { label: "Technical skills", description: "Hard skills" },
        { label: "Leadership", description: "Management skills" },
        { label: "Network", description: "Connections" },
        { label: "Visibility", description: "Recognition" }
      ],
      multiSelect: false
    }
  ]
})
```

**After receiving response**: Store Skills data. Then proceed to Phase 6.

---

### Phase 6: Generate Milestones (ONLY AFTER PHASES 1-5 COMPLETE)

**STOP! Before proceeding, verify you have collected:**
- [ ] Goal Type (Phase 1)
- [ ] Timeline (Phase 1)
- [ ] Language (Phase 1)
- [ ] Goal Statement (Phase 2)
- [ ] Daily Hours (Phase 3)
- [ ] Best Time (Phase 3)
- [ ] Starting Point (Phase 4)
- [ ] Support (Phase 4)
- [ ] Skills (Phase 5)

**If any item is missing, go back and collect it.**

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
- Daily capacity: [X hours/day]
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

### Phase 7: Confirm Milestones (REQUIRED)

**You MUST call AskUserQuestion to confirm milestones before creating files.**

```
AskUserQuestion({
  questions: [
    {
      header: "Confirm Plan",
      question: "Does this milestone plan look good to you?",
      options: [
        { label: "Yes, looks good", description: "Proceed with setup" },
        { label: "Adjust timeline", description: "Change milestone dates" },
        { label: "Adjust scope", description: "Change milestone targets" }
      ],
      multiSelect: false
    }
  ]
})
```

**After confirmation**: Proceed to Phase 8.

---

### Phase 8: Create Data Files

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
      "daily_hours": "[number]",
      "best_time": "[description]",
      "support": "[description]"
    },
    "skills": {
      "[skill_category]": "[level or list]"
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
    "last_weekly_review": "",
    "last_monthly_review": "",
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

### Phase 9: Update Claude Memory

Set Memory pointers:

```
GP_DATA_PATH: ./data
GP_LANG: [language]
GP_LAST_SESSION_DATE: [today]
```

### Phase 10: Display Summary

```markdown
## Setup Complete!

### Your Goal
**Goal**: [statement]
**Target**: [date]
**North Star**: [metric]
**Gap to Close**: [current] → [target]

### Your Profile Summary
- **Daily Capacity**: [X hours/day]
- **Key Strength**: [strength] - we'll leverage this
- **Key Gap**: [gap] - Q1 focuses on building this

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

## Important Notes

- Each phase uses AskUserQuestion tool - do NOT output markdown and wait
- Complete all phases in order before generating milestones
- Always run `date +%Y-%m-%d` before any date calculations

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
