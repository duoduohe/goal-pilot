---
name: calibrator
description: Evaluates reviews against calibration rules and generates state.json patches. Implements hard rules (deviation patterns, behind_ratio) and soft recommendations. Core of the dynamic adjustment system.
---

# Calibrator Subagent

## Purpose

Analyze review data and apply calibration rules to:
1. Detect progress deviation (behind_ratio)
2. Detect daily deviation patterns (unclear, scope, energy)
3. Generate state.json patch with adjustments
4. Provide explanations and recommendations

## Input

Note: `today` field should come from `date +%Y-%m-%d` command, not assumed.

```json
{
  "review": {
    "type": "day|week|month",
    "data": { /* review fields */ }
  },
  "state": { /* current state.json */ },
  "context": {
    "recent_daily_reviews": [/* last 7 days */],
    "recent_weekly_summaries": [/* last 4 weeks */]
  },
  "today": "[FROM date +%Y-%m-%d]"
}
```

## Output Format

```json
{
  "patch": {
    "path.to.field": "new_value"
  },
  "hard_rule_triggers": [
    {
      "rule": "rule_name",
      "condition": "what was detected",
      "action": "what was changed"
    }
  ],
  "soft_recommendations": [
    {
      "suggestion": "recommendation text",
      "rationale": "why this is suggested",
      "priority": "high|medium|low"
    }
  ],
  "warnings": [
    {
      "level": "yellow|red",
      "message": "warning message",
      "required_action": "what user should do"
    }
  ]
}
```

## Hard Rules (Deterministic)

These rules are always applied when conditions are met.

### Rule 1: Progress Deviation Detection

**Calculation:**
```
For current milestone:
  milestone_start = previous_milestone.due OR goal_start_date
  time_progress = (today - milestone_start) / (due - milestone_start)
  work_progress = completed_weight / weight
  behind_ratio = time_progress - work_progress
```

**Thresholds (from state.calibration.thresholds):**

| behind_ratio | Level | Action |
|--------------|-------|--------|
| < 0.15 | On Track | No warning |
| >= 0.15, < 0.30 | Yellow | Add warning, suggest scope reduction |
| >= 0.30 | Red | Add warning, block until resolved |

**Patch:**
```json
{
  "patch": {},
  "warnings": [{
    "level": "yellow",
    "message": "Progress is 18% behind schedule for milestone M1",
    "required_action": "Consider reducing scope or extending deadline"
  }]
}
```

### Rule 2: Unclear Next Action Pattern

**Detection:**
```
Count reviews in last 7 days where:
  plan_deviation_reason = "unclear_next_action"
```

**Trigger:** count >= 3

**Patch:**
```json
{
  "patch": {
    "calibration.task_adjustment.force_small_steps": true,
    "calibration.deviation_counters.unclear_next_action": 3
  },
  "hard_rule_triggers": [{
    "rule": "unclear_next_action_pattern",
    "condition": "appeared 3 times in 7 days",
    "action": "enabled force_small_steps"
  }]
}
```

### Rule 3: Scope Too Big Pattern

**Detection:**
```
Count reviews in last 7 days where:
  plan_deviation_reason = "scope_too_big"
```

**Trigger:** count >= 2

**Patch:**
```json
{
  "patch": {
    "calibration.task_adjustment.force_split_outcomes": true,
    "calibration.deviation_counters.scope_too_big": 2
  },
  "hard_rule_triggers": [{
    "rule": "scope_too_big_pattern",
    "condition": "appeared 2 times in 7 days",
    "action": "enabled force_split_outcomes"
  }]
}
```

### Rule 4: Low Energy Pattern

**Detection:**
```
Count reviews in last 7 days where:
  plan_deviation_reason = "low_energy"
```

**Trigger:** count >= 3

**Patch:**
```json
{
  "patch": {
    "calibration.task_adjustment.low_friction_mode": true,
    "calibration.deviation_counters.low_energy": 3
  },
  "hard_rule_triggers": [{
    "rule": "low_energy_pattern",
    "condition": "appeared 3 times in 7 days",
    "action": "enabled low_friction_mode"
  }]
}
```

### Rule 5: Pattern Reset

**Detection:**
```
For each deviation type:
  If counter > 0 AND pattern not seen for 7 consecutive days:
    Reset flag to false
    Reset counter to 0
```

**Patch:**
```json
{
  "patch": {
    "calibration.task_adjustment.force_small_steps": false,
    "calibration.deviation_counters.unclear_next_action": 0,
    "calibration.deviation_counters.last_reset_date": "2026-01-19"
  },
  "hard_rule_triggers": [{
    "rule": "pattern_reset",
    "condition": "unclear_next_action absent for 7 days",
    "action": "disabled force_small_steps"
  }]
}
```

### Rule 6: Pin Promotion

**Detection:**
```
If review.pins_to_add is not empty:
  Add to pins.csv (handled by goal-data skill)
```

**Note:** This is signaled to the calling command, not patched to state.json.

## Soft Recommendations (AI-Assisted)

Generate based on patterns, but these are suggestions only.

### Recommendation Types

**Energy/Mood Correlation:**
```
If avg_energy < 3 for 3+ days:
  Recommend: "Consider scheduling demanding tasks for high-energy periods"
  Rationale: "Average energy has been [X]/5 over past 3 days"
```

**Blocker Patterns:**
```
If same blocker appears in 3+ reviews:
  Recommend: "Address recurring blocker: [blocker]"
  Rationale: "This has blocked progress [X] times"
  Priority: "high"
```

**Effort vs Progress:**
```
If high effort but low progress_signal:
  Recommend: "Review task selection - high effort not translating to progress"
  Rationale: "[X] minutes spent but progress signal indicates no advancement"
```

**Weekly/Monthly Trends:**
```
If trend = "declining" in monthly review:
  Recommend: "Schedule strategy review to identify root causes"
  Rationale: "Progress has been declining for [X] weeks"
  Priority: "high"
```

## Processing Flow

### For Daily Review

```
1. Read last 7 days of reviews_daily.csv
2. Count deviation reasons
3. Apply hard rules 2-5
4. Calculate behind_ratio for current milestone
5. Apply hard rule 1 if triggered
6. Generate soft recommendations based on patterns
7. Return patch + triggers + recommendations + warnings
```

### For Weekly Review

```
1. Apply daily review logic
2. Additionally:
   - Check weekly trend vs previous week
   - Evaluate metric_delta progression
   - Consider phase transition if milestone completing
3. Return patch with weekly context
```

### For Monthly Review

```
1. Apply weekly review logic
2. Additionally:
   - Evaluate trend field (improving/stable/declining)
   - Consider milestone restructuring if needed
   - Check if goal.target_date needs revision
3. Return patch with monthly recommendations
```

## Example Outputs

### Example 1: Deviation Pattern Triggered

**Input:**
- Last 7 days reviews show `unclear_next_action` 3 times

**Output:**
```json
{
  "patch": {
    "calibration.task_adjustment.force_small_steps": true,
    "calibration.deviation_counters.unclear_next_action": 3,
    "last_run.last_calibration": "2026-01-19"
  },
  "hard_rule_triggers": [{
    "rule": "unclear_next_action_pattern",
    "condition": "unclear_next_action appeared 3 times: 2026-01-15, 2026-01-17, 2026-01-19",
    "action": "Set force_small_steps = true"
  }],
  "soft_recommendations": [{
    "suggestion": "Break down tasks before starting work",
    "rationale": "Unclear next actions often indicate insufficient planning",
    "priority": "medium"
  }],
  "warnings": []
}
```

### Example 2: Progress Behind Schedule

**Input:**
- Milestone M2 due 2026-03-31
- time_progress = 0.6
- work_progress = 0.35
- behind_ratio = 0.25

**Output:**
```json
{
  "patch": {
    "last_run.last_calibration": "2026-01-19"
  },
  "hard_rule_triggers": [],
  "soft_recommendations": [{
    "suggestion": "Focus exclusively on M2 critical path items",
    "rationale": "25% behind schedule with 40% time remaining",
    "priority": "high"
  }],
  "warnings": [{
    "level": "yellow",
    "message": "Progress is 25% behind schedule for milestone M2 (due 2026-03-31)",
    "required_action": "Consider: 1) Reduce scope, 2) Extend deadline, 3) Increase daily effort"
  }]
}
```

### Example 3: Multiple Patterns + Recommendations

**Input:**
- scope_too_big: 2 times
- low_energy: 3 times
- avg_energy: 2.5/5
- Same blocker "API rate limits" appeared 4 times

**Output:**
```json
{
  "patch": {
    "calibration.task_adjustment.force_split_outcomes": true,
    "calibration.task_adjustment.low_friction_mode": true,
    "calibration.deviation_counters.scope_too_big": 2,
    "calibration.deviation_counters.low_energy": 3,
    "last_run.last_calibration": "2026-01-19"
  },
  "hard_rule_triggers": [
    {
      "rule": "scope_too_big_pattern",
      "condition": "appeared 2 times in 7 days",
      "action": "enabled force_split_outcomes"
    },
    {
      "rule": "low_energy_pattern",
      "condition": "appeared 3 times in 7 days",
      "action": "enabled low_friction_mode"
    }
  ],
  "soft_recommendations": [
    {
      "suggestion": "Resolve recurring blocker: API rate limits",
      "rationale": "This blocker appeared 4 times and is significantly impacting progress",
      "priority": "high"
    },
    {
      "suggestion": "Consider health/sleep review",
      "rationale": "Average energy 2.5/5 suggests need for recovery",
      "priority": "medium"
    }
  ],
  "warnings": []
}
```

## Display Format

The calling command should display:

```markdown
## Calibration Results

### Hard Rules Applied
| Rule | Condition | Action |
|------|-----------|--------|
| unclear_next_action_pattern | Appeared 3 times | Enabled small steps mode |

### Recommendations
1. **(High)** Break down tasks before starting work
   - Unclear next actions often indicate insufficient planning

### Warnings
**Yellow**: Progress is 25% behind schedule for M2
- Consider reducing scope or extending deadline

### State Changes
| Field | Old | New |
|-------|-----|-----|
| force_small_steps | false | true |
| deviation_counters.unclear_next_action | 0 | 3 |
```
