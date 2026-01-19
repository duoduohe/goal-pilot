[Root](../CLAUDE.md) > **commands**

# Commands Module

> Slash commands for Goal Pilot: setup, today, review

## Module Responsibility

This module defines the user-facing slash commands:

| Command | File | Description |
|---------|------|-------------|
| `/goal-pilot:setup` | `setup.md` | Initialize goal framework |
| `/goal-pilot:today` | `today.md` | Generate daily tasks |
| `/goal-pilot:review` | `review.md` | Conduct structured reviews |

## Command Details

### /goal-pilot:setup

**Purpose**: Create initial goal framework and data files

**Flow**:
1. Check if state.json exists
2. Collect goal information (statement, target_date, north_star_metric)
3. Generate quarterly milestones
4. Create data/ directory with CSV files
5. Set Claude Memory pointers

**Created Files**:
- `data/state.json`
- `data/reviews_daily.csv`
- `data/reviews_weekly.csv`
- `data/reviews_monthly.csv`
- `data/summaries_weekly.csv`
- `data/summaries_monthly.csv`
- `data/pins.csv`

### /goal-pilot:today

**Purpose**: Generate today's prioritized task list

**Flow**:
1. Load state.json
2. Check progress deviation (behind_ratio)
3. Load layered context (L0/L1/L2/pins)
4. Apply decay weights
5. Check task adjustment flags
6. Invoke planner subagent
7. Display Top 3 Outcomes

**Dependencies**:
- `agents/planner.md` - Task generation
- `skills/goal-data/SKILL.md` - Context loading

### /goal-pilot:review

**Purpose**: Conduct structured review and trigger calibration

**Arguments**: `[day|week|month]` (default: day)

**Flow**:
1. Collect structured fields per review type
2. Validate and append to CSV
3. Invoke calibrator subagent
4. Apply state.json patch
5. Generate summaries (week/month only)
6. Display calibration results

**Dependencies**:
- `agents/calibrator.md` - Pattern detection
- `agents/domain_analyst.md` - Domain reports (week/month)
- `skills/goal-data/SKILL.md` - Data persistence

## Natural Language Mapping

| Phrase | Command |
|--------|---------|
| "Setup my goal" | /goal-pilot:setup |
| "What's today's task?" / "What should I do today?" | /goal-pilot:today |
| "Do a review" / "End of day" | /goal-pilot:review |
| "Weekly review" | /goal-pilot:review week |
| "Monthly review" | /goal-pilot:review month |

## Registration

Commands are registered in `.claude-plugin/plugin.json`:

```json
{
  "commands": [
    { "name": "setup", "path": "commands/setup.md" },
    { "name": "today", "path": "commands/today.md" },
    { "name": "review", "path": "commands/review.md" }
  ]
}
```

## Related Files

| File | Lines | Description |
|------|-------|-------------|
| `setup.md` | 209 | Goal initialization |
| `today.md` | 236 | Daily task generation |
| `review.md` | 385 | Review and calibration |

## Changelog

| Date | Change |
|------|--------|
| 2026-01-19 | Initial module documentation |
