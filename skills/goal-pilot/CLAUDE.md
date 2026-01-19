[Root](../../CLAUDE.md) > [skills](../) > **goal-pilot**

# Goal Pilot Skill

> Core goal guidance capability - goal framework, task generation, reviews, and calibration

## Module Responsibility

This skill is the brain of the Goal Pilot plugin. It:

1. **Defines the goal achievement methodology** for Claude Code
2. **Orchestrates workflows** for goal setup, daily planning, and reviews
3. **Manages layered context** with decay weights
4. **Triggers calibration** based on detected patterns

## Entry and Startup

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill definition and instructions |
| `REVIEWS.md` | Review templates (daily/weekly/monthly) |
| `TEMPLATES.md` | Reusable planning templates |
| `EXAMPLES.md` | Example conversations |

## External Interfaces

### Commands That Use This Skill

| Command | How It Uses goal-pilot |
|---------|----------------------|
| `/goal-pilot:setup` | Creates goal framework per SKILL.md |
| `/goal-pilot:today` | Generates tasks using planner subagent |
| `/goal-pilot:review` | Collects review data, triggers calibrator |

### Integration Points

```
goal-pilot <-- commands (setup, today, review)
    |
    +--> goal-data (data persistence)
    +--> planner (task generation)
    +--> calibrator (rule evaluation)
    +--> domain_analyst (domain-specific analysis)
```

## Key Dependencies and Configuration

### Dependencies

| Dependency | Purpose |
|------------|---------|
| goal-data | CSV/JSON operations, decay calculation |
| agents/planner | Task planning logic |
| agents/calibrator | Calibration rules |
| agents/domain_analyst | Domain-specific reports |

### Configuration (state.json)

```json
{
  "user": { "language": "zh", "timezone": "Asia/Shanghai" },
  "goal": { "statement": "...", "target_date": "..." },
  "plan": { "current_phase": "Q1-Validate", "milestones": [...] },
  "calibration": {
    "decay": { "daily_half_life_days": 30 },
    "task_adjustment": { "force_small_steps": false }
  }
}
```

## Data Model

### Layered Context

| Layer | Source | Age | Weight Formula |
|-------|--------|-----|----------------|
| L0 | reviews_daily.csv | 0-7 days | exp(-ln(2) * age / 30) |
| L1 | summaries_weekly.csv | 31-90 days | exp(-ln(2) * age / 120) |
| L2 | summaries_monthly.csv | 91-365 days | exp(-ln(2) * age / 365) |
| Pins | pins.csv | Any | 1.0 (no decay) |

### Calibration State

| Field | Type | Description |
|-------|------|-------------|
| force_small_steps | boolean | Output 2-3 min steps |
| force_split_outcomes | boolean | Split outcomes into sub-outcomes |
| low_friction_mode | boolean | Max 1 deep work P0 |
| deviation_counters | object | Pattern occurrence counts |

## Testing and Quality

### Manual Testing

1. Run `/goal-pilot:setup` with sample goal
2. Run `/goal-pilot:today` to verify task generation
3. Run `/goal-pilot:review` to test calibration flow

### Example Scenarios

See `EXAMPLES.md` for:
- New user setup flow
- Daily task generation with calibration
- Weekly review with domain reports

## FAQ

**Q: How do I add a new deviation pattern?**
A: Update `agents/calibrator.md` with the new pattern detection rule, then update `skills/goal-data/SKILL.md` to include the new deviation_reason enum value.

**Q: How do I change decay weights?**
A: Decay parameters are in `state.json.calibration.decay`. Update defaults in `skills/goal-data/SKILL.md`.

**Q: How do I add a new language?**
A: Add translations to the language tables in `SKILL.md` and update the language enum in `skills/goal-data/SKILL.md`.

## Related Files

| File | Description |
|------|-------------|
| `SKILL.md` | Main skill definition (318 lines) |
| `REVIEWS.md` | Review templates |
| `TEMPLATES.md` | Planning templates |
| `EXAMPLES.md` | Example conversations |
| `../goal-data/SKILL.md` | Data layer operations |

## Changelog

| Date | Change |
|------|--------|
| 2026-01-19 | Initial module documentation |
