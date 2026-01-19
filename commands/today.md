---
description: Generate today's prioritized tasks based on goal, progress, and recent context. Uses layered context retrieval with decay weights. Applies automatic task adjustments based on calibration rules.
args: ""
examples:
  - "/goal-pilot:today"
  - "今天做什么"
  - "What's today's task"
---

# /goal-pilot:today - Daily Task Generation

## Purpose

Generate today's prioritized task list based on:
- Current goal and milestones from state.json
- Recent daily reviews (L0 context, last 7 days)
- Weekly summaries (L1 context, last 4 weeks)
- Active pins (non-decaying constraints)
- Calibration adjustments (task granularity, energy modes)

## CRITICAL: Get Current Date First

**BEFORE doing anything else, you MUST get the current date using the Bash tool:**

```bash
date +%Y-%m-%d
```

Store this result as `TODAY_DATE`. Use this value for all date calculations and display. DO NOT rely on your internal knowledge for the current date.

## Prerequisites

- `data/state.json` must exist (run `/goal-pilot:setup` first if not)
- Goal and milestones must be defined

## Workflow

### Step 1: Get Current Date and Load State

```
1. Run: date +%Y-%m-%d → Store as TODAY_DATE
2. Read data/state.json
3. If not found → Prompt user to run /goal-pilot:setup
4. Validate schema_version
5. Load calibration.task_adjustment flags
```

### Step 2: Check Progress Deviation

Calculate behind_ratio for current milestone:

```
time_progress = (today - milestone_start) / (due - milestone_start)
work_progress = completed_weight / total_weight
behind_ratio = time_progress - work_progress
```

**If behind_ratio >= 0.30 (Red):**
```markdown
## Progress Alert

You are significantly behind schedule on milestone [M#].

**Time Progress**: [X]%
**Work Progress**: [Y]%
**Gap**: [behind_ratio * 100]%

Before generating today's tasks, please address this:

1. **Extend milestone deadline** - Push [milestone] to [new date]
2. **Reduce scope** - Remove [suggestions] from milestone
3. **Redefine success** - Lower the bar for [milestone]
4. **Do review first** - Run `/goal-pilot:review` to analyze what's blocking

Choose an option to proceed:
```

**If behind_ratio >= 0.15 (Yellow):**
```markdown
## Progress Warning

You're slightly behind on milestone [M#] (gap: [X]%).

Consider:
- Focusing on highest-impact tasks today
- Deferring nice-to-have items
- Running `/goal-pilot:review` if blocked

Proceeding with task generation...
```

### Step 3: Load Layered Context

**L0 - Last 7 days (high weight):**
- Read reviews_daily.csv for last 7 days
- Include full content: done_top3, blocked, deviation_reason, next_day_focus

**L0 partial - Days 8-30 (medium weight):**
- Read reviews_daily.csv for days 8-30
- Include only: blocked, plan_deviation_reason, energy_1_5, mood_1_5

**L1 - Last 4 weeks (medium-low weight):**
- Read summaries_weekly.csv for last 4 weeks
- Include: summary_text, blocked_items

**Pins (weight = 1.0 always):**
- Read pins.csv where active = true
- Include all: type, content

**Apply decay weights:**
```
weight = exp(-ln(2) * age_days / half_life_days)
```

### Step 4: Identify Active Task Adjustments

Check state.json.calibration.task_adjustment:

| Flag | If True |
|------|---------|
| force_small_steps | Output 2-3 minute action steps with checkpoints |
| force_split_outcomes | Split each outcome into 2 sub-outcomes |
| low_friction_mode | Max 1 deep work P0, rest are light tasks |

### Step 5: Invoke Planner Subagent

Pass to planner:
- state.json (goal, milestones, current_phase)
- Weighted context (L0/L1/pins)
- task_adjustment flags
- Today's date

Planner outputs:
- Top 3 Outcomes for today
- Next Actions for each outcome
- Risk/blocker warnings

### Step 6: Display Today's Tasks

**Standard output:**
```markdown
## Today: [Day, Date]

**Goal**: [goal statement]
**Phase**: [current_phase] | **Progress**: [X]%
**Milestone**: [M#] due [date] ([days] days)

[Show any active adjustments]

### Today's Top 3 Outcomes

#### 1. [Outcome 1]
**Next Actions:**
- [ ] [Action 1] (~X min)
- [ ] [Action 2] (~X min)
- [ ] [Action 3] (~X min)

#### 2. [Outcome 2]
**Next Actions:**
- [ ] [Action 1] (~X min)
- [ ] [Action 2] (~X min)

#### 3. [Outcome 3]
**Next Actions:**
- [ ] [Action 1] (~X min)
- [ ] [Action 2] (~X min)

### Risks/Blockers
[From recent reviews]
- [Risk 1]: [mitigation]
- [Risk 2]: [mitigation]

### Constraints (from pins)
- [Constraint 1]
- [Constraint 2]

---
Which outcome would you like to start with?
```

**With force_small_steps:**
```markdown
#### 1. [Outcome 1]
**Small Steps (2-3 min each):**
- [ ] Step 1: [very specific action] (~2 min)
  - Checkpoint: [how to verify]
- [ ] Step 2: [very specific action] (~3 min)
  - Checkpoint: [how to verify]
- [ ] Step 3: [very specific action] (~2 min)
  - Checkpoint: [how to verify]
```

**With low_friction_mode:**
```markdown
### Today's Tasks (Low Friction Mode)

Active because low_energy appeared 3+ times in last 7 days.

#### Deep Work (1 item only)
- [ ] [One meaningful task] (~30-45 min)

#### Light Tasks
- [ ] [Routine/maintenance task 1] (~10 min)
- [ ] [Routine/maintenance task 2] (~10 min)
- [ ] [Easy win task] (~5 min)

#### Optional
- [ ] [Nice to have] (~15 min)
```

### Step 7: Update last_run

After displaying tasks:
- Update state.json.last_run.today = today's date

## Error Handling

**No state.json:**
```markdown
## Setup Required

No GTD data found. Please run `/goal-pilot:setup` first to:
1. Define your goal
2. Set milestones
3. Create data files

Run `/goal-pilot:setup` to get started.
```

**Empty goal:**
```markdown
## Goal Not Defined

Your state.json exists but has no goal defined.
Please run `/goal-pilot:setup` to set your goal.
```

## Natural Language Triggers

These phrases invoke /goal-pilot:today:
- "今天做什么"
- "今天的任务"
- "What's today's task"
- "What should I do today"
- "Daily tasks"
- "Give me today's plan"
