---
name: goal-pilot
description: Personal goal achievement guidance that helps users achieve their annual goals. Use when users mention goals, plans, daily tasks, productivity, time management, task tracking, reviews, or need help breaking down objectives into actionable steps. Features dynamic review system with structured data persistence, layered context decay, and automatic calibration. Supports multiple languages.
---

# Goal Pilot - Personal Goal Achievement System

## Overview

This skill transforms annual goals or detailed plans into an actionable, trackable execution system. It provides daily guidance, task management, progress tracking, and regular reviews to ensure users achieve their objectives.

**Key Features**:
- Converts goals into structured SOPs with timelines
- Generates daily prioritized task lists with layered context
- Breaks down tasks into specific steps
- Persists data to local CSV/JSON files
- Automatic context decay - older data has less weight
- Calibration rules - automatic task adjustment based on patterns
- Triggers appropriate reviews (daily/weekly/monthly/quarterly)
- Adapts plans based on progress and feedback

## Architecture

### Data Storage (Primary)

Uses local files as the source of truth:

```
data/
├── state.json              # Configuration, plan, calibration settings
├── reviews_daily.csv       # Daily review records
├── reviews_weekly.csv      # Weekly review records
├── reviews_monthly.csv     # Monthly review records
├── summaries_weekly.csv    # Weekly summaries (L1 context)
├── summaries_monthly.csv   # Monthly summaries (L2 context)
└── pins.csv                # Long-term non-decaying knowledge
```

### Memory (Pointers Only)

Claude Memory only stores lightweight pointers:

```
GTD_DATA_PATH: ./data
GP_LANG: zh
GP_LAST_SESSION_DATE: 2026-01-19
```

This enables:
- **Decay control**: Older data automatically gets lower weight
- **Structured queries**: Filter reviews by date range
- **Backup/migration**: Data is portable and versionable
- **Calibration**: Pattern detection triggers automatic adjustments

## Workflow

```
┌─────────────────────────────────────────────────────────────┐
│  SESSION START                                              │
│  Check data/state.json existence                            │
└─────────────────────────────────────────────────────────────┘
                            ↓
              ┌─────────────┴─────────────┐
              ↓                           ↓
┌─────────────────────────┐   ┌─────────────────────────┐
│  NEW USER               │   │  RETURNING USER         │
│  • Run /goal-pilot:setup       │   │  • Load state.json      │
│  • Define goal          │   │  • Check calibration    │
│  • Create state.json    │   │  • Load layered context │
│  • Initialize CSVs      │   │  • Generate today's     │
└─────────────────────────┘   │    tasks via planner    │
                              └─────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  DAILY EXECUTION (/goal-pilot:today)                               │
│  • Load L0/L1/L2/pins context with decay weights            │
│  • Invoke planner subagent                                  │
│  • Apply task_adjustment flags (small_steps, split, etc.)   │
│  • Present prioritized task list                            │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  REVIEW (/goal-pilot:review [day|week|month])                      │
│  • Collect structured fields                                │
│  • Append to appropriate CSV                                │
│  • Invoke calibrator subagent                               │
│  • Apply state.json patch                                   │
│  • Generate summaries (for week/month)                      │
└─────────────────────────────────────────────────────────────┘
```

## Layered Context Retrieval

When generating tasks or making decisions, retrieve context in layers:

| Layer | Age | Content | Weight |
|-------|-----|---------|--------|
| L0 | 0-7 days | Full daily reviews | High (1.0 - 0.84) |
| L0 partial | 8-30 days | Key fields only | Medium (0.84 - 0.5) |
| L1 | 31-90 days | Weekly summaries | Medium-Low |
| L2 | 91-365 days | Monthly summaries | Low |
| Pins | Any | All active pins | Always 1.0 |

**Decay Formula:**
```
weight = exp(-ln(2) * age_days / half_life_days)
```

Default half-life values (configurable in state.json):
- Daily: 30 days
- Weekly: 120 days
- Monthly: 365 days

## Calibration Rules

### Progress Deviation Detection

Calculate `behind_ratio` for each milestone:
```
time_progress = (today - milestone_start) / (due - milestone_start)
work_progress = completed_weight / total_weight
behind_ratio = time_progress - work_progress
```

| behind_ratio | Status | Action |
|--------------|--------|--------|
| < 0.15 | On Track | Normal operation |
| >= 0.15, < 0.30 | Yellow | Suggest scope reduction |
| >= 0.30 | Red | Block /goal-pilot:today until addressed |

### Daily Deviation Pattern Triggers

Count `plan_deviation_reason` occurrences in 7-day window:

| Pattern | Trigger | Adjustment |
|---------|---------|------------|
| `unclear_next_action` >= 3 | Set `force_small_steps = true` | Output 2-3 min step granularity |
| `scope_too_big` >= 2 | Set `force_split_outcomes = true` | Split each outcome into 2 sub-outcomes |
| `low_energy` >= 3 | Set `low_friction_mode = true` | Max 1 deep work P0, rest are light tasks |

Adjustments auto-reset after 7 consecutive days without the pattern.

## Instructions for Claude

### Step 0: Session Initialization

**Check for data first:**

```
1. Check if data/state.json exists
2. If exists → Load state, check calibration flags, proceed to Returning User
3. If not exists → New User flow
```

**For Returning Users:**
```markdown
## Welcome back!

**Goal**: [from state.json]
**Progress**: [X]% | **Phase**: [current phase]
**Last review**: [date]

[Show any calibration warnings if behind_ratio triggers]
[Show active task_adjustment modes if any]

Today is [Day, Date]

[Invoke planner subagent with layered context]
```

**For New Users:**
```markdown
## Goal Pilot - Let's Set Up Your Goal

Run `/goal-pilot:setup` to initialize your goal framework.

Or tell me:
1. **What's your goal?** (Be specific and measurable)
2. **What's your timeline?** (Target completion date)
3. **What's your current situation?** (Starting point, resources, constraints)
4. **Preferred language?** (English/中文/日本語)
```

### Step 1: Goal Framework Generation

After user provides goal information via `/goal-pilot:setup`:

1. Create `data/state.json` with schema
2. Create empty CSV files with headers
3. Store Memory pointers (GTD_DATA_PATH, GP_LANG, GP_LAST_SESSION_DATE)
4. Display goal framework summary

### Step 2: Daily Task Generation (/goal-pilot:today)

**Context Loading:**
1. Read state.json for plan and calibration settings
2. Load L0 context (last 7 days daily reviews)
3. Load L1 context (last 4 weeks summaries) if available
4. Load all active pins
5. Apply decay weights

**Task Adjustment:**
- If `force_small_steps`: Output 2-3 minute action steps
- If `force_split_outcomes`: Split each outcome into 2 sub-outcomes
- If `low_friction_mode`: Limit P0 to 1 deep work item

**Invoke Planner Subagent:**
- Input: state + weighted context
- Output: Today's Top 3 Outcomes with Next Actions

### Step 3: Review Execution (/goal-pilot:review)

**Collect Structured Fields:**

For daily review:
- done_top3, blocked, plan_deviation_reason
- effort_minutes, energy_1_5 (1-5), mood_1_5 (1-5)
- progress_signal, next_day_focus
- pins_to_add (optional)

For weekly review:
- win, loss, main_blocker
- metric_delta, plan_adjustment_suggested

For monthly review:
- major_outcome, trend
- root_causes, next_month_strategy

**After Collection:**
1. Append to appropriate CSV via goal-data skill
2. Invoke calibrator subagent
3. Apply state.json patch
4. Generate summary (for week/month reviews)
5. Display calibration changes to user

### Step 4: Calibration Application

When calibrator returns patch:

```markdown
## Calibration Applied

| Setting | Previous | New | Reason |
|---------|----------|-----|--------|
| force_small_steps | false | true | unclear_next_action appeared 3 times in 7 days |

**Recommendation**: [calibrator's suggestion]
```

## Slash Commands

| Command | Action |
|---------|--------|
| `/goal-pilot:setup` | Initialize goal framework, create data files |
| `/goal-pilot:today` | Generate today's tasks with layered context |
| `/goal-pilot:review` | Daily review (default) |
| `/goal-pilot:review week` | Weekly review + summary generation |
| `/goal-pilot:review month` | Monthly review + summary generation |

## Natural Language Compatibility

These phrases still work:

| Phrase | Maps To |
|--------|---------|
| "今天做什么" / "What's today's task" | `/goal-pilot:today` |
| "做复盘" / "Do a review" | `/goal-pilot:review` |
| "周复盘" / "Weekly review" | `/goal-pilot:review week` |
| "月复盘" / "Monthly review" | `/goal-pilot:review month` |
| "查看进度" / "Show progress" | Display state.json summary |

## Subagent Integration

### Planner Subagent
- **When**: `/goal-pilot:today` execution
- **Input**: state.json + weighted context (L0/L1/L2/pins)
- **Output**: Top 3 Outcomes with Next Actions

### Calibrator Subagent
- **When**: After any `/goal-pilot:review`
- **Input**: Latest review + state.json + recent summaries
- **Output**: state.json patch + explanation

### Domain Analyst Subagent
- **When**: Week/month review with configured domains
- **Input**: domain parameter + domain-specific reviews
- **Output**: Domain progress report

## Language Support

Respond in user's preferred language (from state.json.user.language).

| English | 中文 | 日本語 |
|---------|------|--------|
| Goal | 目标 | 目標 |
| Task | 任务 | タスク |
| Progress | 进度 | 進捗 |
| Review | 复盘 | 振り返り |
| Calibration | 校准 | 調整 |
| Decay | 衰减 | 減衰 |

## Status Icons

- ⬜ Todo
- 🔄 In Progress
- ✅ Completed
- ❌ Cancelled
- ⏸️ Paused
- ⚠️ At Risk (Yellow)
- 🔴 Critical (Red)

## File References

- [REVIEWS.md](REVIEWS.md) - Review templates (daily/weekly/monthly/quarterly)
- [TEMPLATES.md](TEMPLATES.md) - Reusable planning templates
- [EXAMPLES.md](EXAMPLES.md) - Example conversations
- [../goal-data/SKILL.md](../goal-data/SKILL.md) - Data layer operations
