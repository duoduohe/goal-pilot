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

Use AskUserQuestion tool to collect goal information:

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
    }
  ]
})
```

Then ask for specifics based on goal type:

```
AskUserQuestion({
  questions: [
    {
      header: "Timeline",
      question: "When do you want to achieve this goal?",
      options: [
        { label: "3 months", description: "Short-term intensive" },
        { label: "6 months", description: "Medium-term focused" },
        { label: "12 months", description: "Full year plan" },
        { label: "Custom", description: "Specify your own date" }
      ],
      multiSelect: false
    }
  ]
})
```

For language preference:

```
AskUserQuestion({
  questions: [
    {
      header: "Language",
      question: "Preferred language for Goal Pilot responses?",
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

**Note**: For open-ended questions like "Goal Statement" or "North Star Metric", use AskUserQuestion with the "Other" option (automatically included) to allow free-form input.

### Step 4: Assess Current Situation (NEW - IMPORTANT)

After collecting goal, assess user's starting point using AskUserQuestion:

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
    }
  ]
})
```

For specific metrics, prompt user to provide current values (via "Other" option).

### Step 5: Assess Resources & Constraints (NEW - IMPORTANT)

Use AskUserQuestion to collect resource information:

```
AskUserQuestion({
  questions: [
    {
      header: "Weekly Hours",
      question: "How many hours per week can you dedicate to this goal?",
      options: [
        { label: "5-10 hours", description: "Part-time, limited availability" },
        { label: "10-20 hours", description: "Side project level" },
        { label: "20-40 hours", description: "Significant commitment" },
        { label: "40+ hours", description: "Full-time focus" }
      ],
      multiSelect: false
    },
    {
      header: "Work Style",
      question: "When are you most productive?",
      options: [
        { label: "Morning (6am-12pm)", description: "Early bird" },
        { label: "Afternoon (12pm-6pm)", description: "Midday focus" },
        { label: "Evening (6pm-12am)", description: "Night owl" },
        { label: "Flexible", description: "Varies by day" }
      ],
      multiSelect: false
    }
  ]
})
```

For team/support:

```
AskUserQuestion({
  questions: [
    {
      header: "Support",
      question: "Do you have help or working solo?",
      options: [
        { label: "Solo", description: "Working alone" },
        { label: "Partner/Co-founder", description: "Have a partner" },
        { label: "Small team", description: "2-5 people" },
        { label: "Can hire", description: "Budget for freelancers/contractors" }
      ],
      multiSelect: false
    }
  ]
})
```

### Step 6: Assess Skills & Experience (NEW - IMPORTANT)

Based on the goal type, use AskUserQuestion to collect skill information:

**For Product/Business Goals:**
```
AskUserQuestion({
  questions: [
    {
      header: "Tech Skills",
      question: "What's your technical capability?",
      options: [
        { label: "Non-technical", description: "Need to hire for dev work" },
        { label: "Basic", description: "Can do simple tasks, need help for complex" },
        { label: "Intermediate", description: "Can build most features" },
        { label: "Expert", description: "Full-stack, can build anything" }
      ],
      multiSelect: false
    },
    {
      header: "Business Skills",
      question: "Which areas are you strongest in?",
      options: [
        { label: "Marketing", description: "Content, SEO, ads" },
        { label: "Sales", description: "Outreach, closing deals" },
        { label: "Product", description: "User research, prioritization" },
        { label: "Operations", description: "Process, systems" }
      ],
      multiSelect: true
    }
  ]
})
```

**For Fitness Goals:**
```
AskUserQuestion({
  questions: [
    {
      header: "Exercise Level",
      question: "How often do you currently exercise?",
      options: [
        { label: "Rarely", description: "Less than once a week" },
        { label: "Sometimes", description: "1-2 times per week" },
        { label: "Regular", description: "3-4 times per week" },
        { label: "Daily", description: "5+ times per week" }
      ],
      multiSelect: false
    },
    {
      header: "Gym Access",
      question: "What equipment do you have access to?",
      options: [
        { label: "None", description: "Bodyweight only" },
        { label: "Home gym", description: "Basic equipment at home" },
        { label: "Gym membership", description: "Full gym access" }
      ],
      multiSelect: false
    }
  ]
})
```

**For Learning Goals:**
```
AskUserQuestion({
  questions: [
    {
      header: "Current Level",
      question: "What's your current skill level?",
      options: [
        { label: "Beginner", description: "Just starting out" },
        { label: "Elementary", description: "Know basics" },
        { label: "Intermediate", description: "Can handle common tasks" },
        { label: "Advanced", description: "Proficient, refining skills" }
      ],
      multiSelect: false
    },
    {
      header: "Learning Style",
      question: "How do you learn best?",
      options: [
        { label: "Video courses", description: "Structured video content" },
        { label: "Reading", description: "Books, articles, docs" },
        { label: "Practice", description: "Hands-on projects" },
        { label: "Mentorship", description: "1-on-1 guidance" }
      ],
      multiSelect: true
    }
  ]
})
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
