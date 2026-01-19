[Root](../CLAUDE.md) > **agents**

# Agents Module

> Subagents for specialized tasks: planning, calibration, domain analysis

## Module Responsibility

This module contains three subagents that handle specialized logic:

1. **planner** - Generates daily task plans with Top 3 Outcomes
2. **calibrator** - Evaluates reviews and applies calibration rules
3. **domain_analyst** - Provides domain-specific progress reports

## Subagent Overview

| Agent | File | Invoked By | Purpose |
|-------|------|------------|---------|
| planner | `planner.md` | `/goal-pilot:today` | Task planning with context |
| calibrator | `calibrator.md` | `/goal-pilot:review` | Pattern detection and state patches |
| domain_analyst | `domain_analyst.md` | `/goal-pilot:review week/month` | Domain-specific analysis |

## Planner Subagent

### Input
```json
{
  "state": { "goal": {...}, "plan": {...}, "calibration": {...} },
  "context": { "L0": [...], "L1": [...], "pins": [...] },
  "today": "2026-01-19"
}
```

### Output
```json
{
  "outcomes": [
    { "title": "...", "why": "...", "next_actions": [...], "priority": "P0" }
  ],
  "risks": [...],
  "constraints_applied": [...]
}
```

### Task Adjustment Modes
- `force_small_steps`: 2-3 minute action steps
- `force_split_outcomes`: Split each outcome into 2 sub-outcomes
- `low_friction_mode`: Max 1 deep work P0

## Calibrator Subagent

### Input
```json
{
  "review": { "type": "day|week|month", "data": {...} },
  "state": {...},
  "context": { "recent_daily_reviews": [...] },
  "today": "2026-01-19"
}
```

### Output
```json
{
  "patch": { "path.to.field": "new_value" },
  "hard_rule_triggers": [...],
  "soft_recommendations": [...],
  "warnings": [...]
}
```

### Hard Rules

| Pattern | Trigger | Action |
|---------|---------|--------|
| unclear_next_action | >= 3 in 7 days | force_small_steps = true |
| scope_too_big | >= 2 in 7 days | force_split_outcomes = true |
| low_energy | >= 3 in 7 days | low_friction_mode = true |

### Progress Deviation

| behind_ratio | Level | Action |
|--------------|-------|--------|
| < 0.15 | On Track | Normal |
| >= 0.15 | Yellow | Warning |
| >= 0.30 | Red | Block until resolved |

## Domain Analyst Subagent

### Input
```json
{
  "domain": "fitness",
  "state": {...},
  "reviews": { "daily": [...], "weekly": [...] },
  "pins": [...],
  "review_type": "week|month",
  "period": { "start": "...", "end": "..." }
}
```

### Output
```json
{
  "domain": "fitness",
  "summary": { "status": "on_track|needs_attention|at_risk", "progress_text": "...", "key_metric": {...} },
  "wins": [...],
  "challenges": [...],
  "recommendations": [...]
}
```

### Supported Domains
- **fitness** - workout tracking, energy correlation
- **english** - language learning progress
- **product** - feature completion, milestone alignment

## Data Flow

```
/goal-pilot:today
    |
    v
[Load state + context]
    |
    v
[planner subagent] --> Top 3 Outcomes
    |
    v
[Display to user]

/goal-pilot:review
    |
    v
[Collect review fields]
    |
    v
[calibrator subagent] --> state.json patch
    |
    v
[domain_analyst] (if weekly/monthly) --> Domain reports
    |
    v
[Update state.json + Display]
```

## Related Files

| File | Lines | Description |
|------|-------|-------------|
| `planner.md` | 269 | Task planning logic |
| `calibrator.md` | 411 | Calibration rules |
| `domain_analyst.md` | 335 | Domain analysis |

## Changelog

| Date | Change |
|------|--------|
| 2026-01-19 | Initial module documentation |
