---
description: Conduct structured review and trigger calibration. Supports daily (default), weekly, and monthly review types. Collects structured fields, persists to CSV, and updates state.json based on calibration rules.
args: "[day|week|month]"
examples:
  - "/goal-pilot:review"
  - "/goal-pilot:review day"
  - "/goal-pilot:review week"
  - "/goal-pilot:review month"
  - "做复盘"
  - "周复盘"
---

# /goal-pilot:review - Structured Review & Calibration

## Purpose

Conduct a structured review that:
1. Collects specific fields (not free-form)
2. Persists data to CSV files
3. Triggers calibrator subagent
4. Updates state.json with calibration patch
5. Generates summaries (for week/month)

## Review Types

| Type | Trigger | CSV File | Summary Generated |
|------|---------|----------|-------------------|
| day | Default, or explicit "day" | reviews_daily.csv | No |
| week | Explicit "week" | reviews_weekly.csv | summaries_weekly.csv |
| month | Explicit "month" | reviews_monthly.csv | summaries_monthly.csv |

## Daily Review

### Fields to Collect

| Field | Required | Description | Validation |
|-------|----------|-------------|------------|
| done_top3 | Yes | Top 1-3 accomplishments | Pipe-separated |
| blocked | No | What blocked progress | Pipe-separated |
| plan_deviation_reason | Yes | Why plan wasn't followed | Enum (see below) |
| effort_minutes | Yes | Total focused work time | Integer >= 0 |
| energy_1_5 | Yes | Energy level | 1-5 |
| mood_1_5 | Yes | Mood level | 1-5 |
| progress_signal | Yes | Did you move toward goal? | Short text |
| next_day_focus | Yes | Top priority for tomorrow | Short text |
| pins_to_add | No | Long-term constraint/lesson | Short text |
| notes | No | Additional notes | Free text |

**plan_deviation_reason values:**
- `scope_too_big` - Task was larger than expected
- `unclear_next_action` - Didn't know what to do next
- `low_energy` - Too tired to work effectively
- `external_interrupt` - Interrupted by others/events
- `procrastination` - Avoided the work
- `other` - Other reason
- `none` - No deviation, plan was followed

### Collection Flow

```markdown
## Daily Review - [Date]

Let's capture today's progress.

### 1. Accomplishments
What were your top 1-3 accomplishments today?
(Separate multiple items with |)

> [User input]

### 2. Blockers
What blocked your progress today? (Enter "none" if nothing)

> [User input]

### 3. Plan Deviation
Did you follow today's plan? If not, why?
1. scope_too_big - Task was larger than expected
2. unclear_next_action - Didn't know what to do next
3. low_energy - Too tired
4. external_interrupt - External interruption
5. procrastination - Avoided work
6. other - Other reason
7. none - Plan was followed

> [User selects]

### 4. Effort
How many minutes of focused work today?

> [User input: integer]

### 5. Energy Level (1-5)
1 = Exhausted, 5 = Full of energy

> [User selects]

### 6. Mood Level (1-5)
1 = Very low, 5 = Great mood

> [User selects]

### 7. Progress Signal
One sentence: Did you move closer to your goal today?

> [User input]

### 8. Tomorrow's Focus
What's the #1 priority for tomorrow?

> [User input]

### 9. Pin Candidate (Optional)
Any long-term constraint or lesson to remember?
(These won't decay over time)

> [User input or skip]

### 10. Notes (Optional)
Anything else to record?

> [User input or skip]
```

### After Collection

1. **Validate fields** - Ensure required fields are present
2. **Append to CSV** - Write to reviews_daily.csv
3. **Update deviation counter** - If deviation_reason != "none", increment counter
4. **Invoke calibrator** - Pass review + state + recent context
5. **Apply patch** - Update state.json with calibrator output
6. **Display summary**

### Output

```markdown
## Review Saved

| Field | Value |
|-------|-------|
| Date | [date] |
| Accomplishments | [done_top3] |
| Blocked | [blocked] |
| Deviation | [reason] |
| Effort | [X] min |
| Energy | [X]/5 |
| Mood | [X]/5 |
| Progress | [signal] |
| Tomorrow | [focus] |

### Calibration Applied

[If any adjustments triggered]

| Setting | Change | Reason |
|---------|--------|--------|
| [setting] | [old] → [new] | [reason] |

### Next Steps
- Tomorrow: [next_day_focus]
- Run `/goal-pilot:today` tomorrow to get tasks
```

## Weekly Review

### Fields to Collect

| Field | Required | Description |
|-------|----------|-------------|
| week_start | Auto | Monday of the week |
| week_end | Auto | Sunday of the week |
| win | Yes | Biggest win this week |
| loss | Yes | Biggest challenge/loss |
| main_blocker | Yes | Primary blocker |
| metric_delta | Yes | How did key metric change? |
| plan_adjustment_suggested | No | Any plan changes needed? |
| notes | No | Additional notes |

### Collection Flow

```markdown
## Weekly Review - Week [X]
[week_start] to [week_end]

### This Week's Summary
[Auto-generated from daily reviews]
- Total effort: [X] minutes
- Avg energy: [X]/5
- Avg mood: [X]/5
- Tasks completed: [X]

### 1. Biggest Win
What was your biggest accomplishment this week?

> [User input]

### 2. Biggest Challenge
What was your biggest challenge or loss?

> [User input]

### 3. Main Blocker
What was the primary thing blocking your progress?

> [User input]

### 4. Metric Change
How did your north star metric change this week?
(e.g., "+500 MRR" or "3 new users" or "no change")

> [User input]

### 5. Plan Adjustment (Optional)
Any adjustments needed to your plan?

> [User input or skip]

### 6. Notes (Optional)
Anything else to record?

> [User input or skip]
```

### After Collection

1. **Append to reviews_weekly.csv**
2. **Generate weekly summary** from daily reviews
3. **Append to summaries_weekly.csv**
4. **Invoke calibrator** with weekly context
5. **Invoke domain_analyst** for each configured domain (optional)
6. **Apply state.json patch**

### Weekly Summary Generation

Aggregate from daily reviews:
```
total_effort_minutes = sum(effort_minutes)
avg_energy = mean(energy_1_5)
avg_mood = mean(mood_1_5)
deviation_reasons_count = {
  "unclear_next_action": X,
  "scope_too_big": Y,
  ...
}
blocked_items = unique(blocked)
summary_text = "[Generated 1-2 sentence summary]"
```

## Monthly Review

### Fields to Collect

| Field | Required | Description |
|-------|----------|-------------|
| month | Auto | YYYY-MM |
| major_outcome | Yes | Biggest outcome this month |
| trend | Yes | improving/stable/declining |
| root_causes | Yes | What drove the trend |
| plan_adjustment_suggested | Yes | Recommended plan changes |
| next_month_strategy | Yes | Focus for next month |
| notes | No | Additional notes |

### Collection Flow

```markdown
## Monthly Review - [Month Year]

### This Month's Summary
[Auto-generated from weekly summaries]
- Total effort: [X] hours
- Avg energy: [X]/5
- Avg mood: [X]/5
- Weeks reviewed: [X]

### 1. Major Outcome
What was the biggest outcome this month?

> [User input]

### 2. Trend
How is progress trending?
1. improving - Getting better each week
2. stable - Consistent progress
3. declining - Struggling or slowing down

> [User selects]

### 3. Root Causes
What drove this trend? (Be specific)

> [User input]

### 4. Plan Adjustment
Based on this month, what changes should we make to the plan?

> [User input]

### 5. Next Month Strategy
What's the key strategy for next month?

> [User input]

### 6. Notes (Optional)

> [User input or skip]
```

### After Collection

1. **Append to reviews_monthly.csv**
2. **Generate monthly summary** from weekly summaries
3. **Append to summaries_monthly.csv**
4. **Invoke calibrator** with monthly context
5. **Check milestone progress** - May trigger phase change
6. **Apply state.json patch**

## Calibration Integration

After each review, calibrator evaluates:

### Hard Rules (Automatic)

**Daily deviation pattern detection:**
```
Read reviews_daily for last 7 days
Count occurrences of each plan_deviation_reason

If unclear_next_action >= 3:
  Set task_adjustment.force_small_steps = true

If scope_too_big >= 2:
  Set task_adjustment.force_split_outcomes = true

If low_energy >= 3:
  Set task_adjustment.low_friction_mode = true
```

**Pattern reset:**
```
If pattern not seen for 7 consecutive days:
  Reset corresponding flag to false
```

### Progress Check

Calculate behind_ratio and update warnings.

### Calibrator Output

```json
{
  "patch": {
    "calibration.task_adjustment.force_small_steps": true,
    "calibration.deviation_counters.unclear_next_action": 3
  },
  "explanation": "unclear_next_action appeared 3 times in last 7 days",
  "recommendation": "Tomorrow's tasks will use smaller step granularity"
}
```

## Error Handling

**No state.json:**
```markdown
Please run `/goal-pilot:setup` first to initialize your goal.
```

**No daily reviews for weekly summary:**
```markdown
No daily reviews found for this week.
Run `/goal-pilot:review` at least once before weekly review.
```

## Natural Language Triggers

| Phrase | Maps To |
|--------|---------|
| "做复盘" | /goal-pilot:review (day) |
| "日复盘" | /goal-pilot:review day |
| "周复盘" | /goal-pilot:review week |
| "月复盘" | /goal-pilot:review month |
| "Do a review" | /goal-pilot:review (day) |
| "Weekly review" | /goal-pilot:review week |
| "Monthly review" | /goal-pilot:review month |
