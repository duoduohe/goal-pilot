# Goal Pilot - Claude Code Plugin

[English](README.md) | [中文](README_CN.md)

Personal goal achievement guidance that transforms annual goals into actionable daily tasks with **dynamic review and calibration**.

## Features

- **Structured Data Persistence**: Review records saved to CSV files, state tracking with JSON
- **Layered Context Decay**: Older data has less weight
- **Automatic Calibration**: Task adjustments based on detected patterns
- **Slash Commands**: `/goal-pilot:setup`, `/goal-pilot:today`, `/goal-pilot:review`
- **Subagent Architecture**: Planner, Calibrator, Domain Analyst
- **Goal → SOP Framework**: High-level goals into structured plans with quarterly milestones
- **Daily Task Generation**: Prioritized task lists based on layered context (last 7 days reviews, weekly summaries, pins)
- **Step-by-Step Guidance**: Each task broken down into specific actionable steps
- **Data Persistence**: Progress saved to local files (`data/` directory)
- **Review System**: Structured daily/weekly/monthly reviews with calibration
- **Adaptive Planning**: Automatic task adjustment based on deviation patterns
- **Multi-language**: Supports English, 中文, 日本語

## Installation

### Via Claude Code CLI

```bash
/plugin marketplace add github:duoduohe/goal-pilot
```

### Manual Installation

```bash
git clone https://github.com/duoduohe/goal-pilot.git
cd goal-pilot
claude plugin install .
```

## Quick Start

### First Time Setup

Run the setup command:

```
/goal-pilot:setup
```

Or describe your goal naturally:

```
I want to launch my SaaS product and reach $10K MRR in 12 months.
I have an MVP and can dedicate 20 hours per week.
```

Claude will:
1. Create `data/state.json` with your goal framework
2. Generate quarterly milestones
3. Initialize CSV files for reviews
4. Generate your first day's tasks

### Daily Usage

```
/goal-pilot:today
```

Or use natural language:

```
What should I do today?
今天做什么？
```

Claude will:
1. Load your goal and recent context
2. Apply decay weights (recent reviews matter more)
3. Check calibration adjustments
4. Generate Top 3 Outcomes with Next Actions

### Daily Review

```
/goal-pilot:review
```

Claude will:
1. Collect structured fields (accomplishments, blockers, energy, mood)
2. Save to `data/reviews_daily.csv`
3. Run calibration rules
4. Apply automatic adjustments if patterns detected

### Weekly/Monthly Reviews

```
/goal-pilot:review week
/goal-pilot:review month
```

## Slash Commands

| Command | Action |
|---------|--------|
| `/goal-pilot:setup` | Initialize goal, create data files |
| `/goal-pilot:today` | Generate today's tasks with layered context |
| `/goal-pilot:review` | Daily review (default) |
| `/goal-pilot:review week` | Weekly review + summary generation |
| `/goal-pilot:review month` | Monthly review + summary generation |

## Natural Language

| Phrase | Maps To |
|--------|---------|
| "What's today's task?" / "今天做什么" | `/goal-pilot:today` |
| "Do a review" / "做复盘" | `/goal-pilot:review` |
| "Weekly review" / "周复盘" | `/goal-pilot:review week` |
| "Show progress" / "查看进度" | Display state.json summary |
| "Reset goal" / "重置目标" | Clear data, start fresh |

## Calibration System

### Automatic Task Adjustments

The system detects patterns and adjusts automatically:

| Pattern Detected | Adjustment |
|------------------|------------|
| "unclear next action" 3+ times in 7 days | Smaller step granularity (2-3 min) |
| "scope too big" 2+ times in 7 days | Split outcomes into sub-outcomes |
| "low energy" 3+ times in 7 days | Low friction mode (only 1 deep work item) |

### Progress Deviation Alerts

| behind_ratio | Alert Level | Action |
|--------------|-------------|--------|
| < 15% | On Track | Normal operation |
| 15-30% | Yellow | Suggest scope reduction |
| > 30% | Red | Block until addressed |

## Data Structure

### File Structure

```
goal-pilot-plugin/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── commands/
│   ├── setup.md             # /goal-pilot:setup command
│   ├── today.md             # /goal-pilot:today command
│   └── review.md            # /goal-pilot:review command
├── agents/
│   ├── planner.md           # Task planning subagent
│   ├── calibrator.md        # Calibration rules subagent
│   └── domain_analyst.md    # Domain-specific analysis
├── skills/
│   ├── goal-pilot/
│   │   ├── SKILL.md         # Main skill definition
│   │   ├── REVIEWS.md       # Review templates
│   │   ├── TEMPLATES.md     # Reusable templates
│   │   └── EXAMPLES.md      # Example conversations
│   └── goal-data/
│       └── SKILL.md         # Data layer operations
├── data/                    # User data (created on setup)
│   ├── state.json           # Goal, milestones, calibration
│   ├── reviews_daily.csv    # Daily review records
│   ├── reviews_weekly.csv   # Weekly review records
│   ├── reviews_monthly.csv  # Monthly review records
│   ├── summaries_weekly.csv # Weekly summaries (L1)
│   ├── summaries_monthly.csv # Monthly summaries (L2)
│   └── pins.csv             # Long-term constraints/lessons
├── README.md
├── README_CN.md
└── LICENSE
```

### Memory (Pointers Only)

Claude Memory now stores only lightweight pointers:

```
GP_DATA_PATH: ./data
GP_LANG: en
GP_LAST_SESSION_DATE: 2026-01-19
```

All actual data is in files in the `data/` directory.

## Examples

### Setup Flow

```
User: /goal-pilot:setup

Claude: ## Goal Pilot Setup

Let's set up your goal framework. Please provide:

1. **Goal Statement**: What do you want to achieve?
2. **Target Date**: When? (YYYY-MM-DD)
3. **North Star Metric**: What number measures success?
4. **Domains** (optional): fitness, english, etc.
5. **Language**: English/中文/日本語

User: Get my SaaS to $10K MRR by 2026-12-31.
North star is MRR. Domains: product, marketing. English please.

Claude: ## Setup Complete!

**Goal**: SaaS to $10K MRR
**Target**: 2026-12-31
**North Star**: MRR
**Phase**: Q1-Validate

### Milestones Created
| Q | Phase | Due | Milestone |
|---|-------|-----|-----------|
| Q1 | Validate | 2026-03-31 | First paying customer |
| Q2 | Scale | 2026-06-30 | $2.5K MRR |
| Q3 | Systematize | 2026-09-30 | $5K MRR |
| Q4 | Achieve | 2026-12-31 | $10K MRR |

Run `/goal-pilot:today` to start!
```

### Daily Tasks with Calibration

```
User: /goal-pilot:today

Claude: ## Today: Monday, January 19, 2026

**Goal**: SaaS → $10K MRR
**Phase**: Q1-Validate | **Progress**: 15%
**Milestone**: M1 due in 71 days

**Active Adjustment**: Small steps mode (unclear_next_action detected 3 times)

### Today's Top 3 Outcomes

#### 1. Complete user authentication (P0)
**Small Steps:**
- [ ] Create auth route file (~2 min)
  - Checkpoint: File exists
- [ ] Define user schema (~3 min)
  - Checkpoint: Schema validates
- [ ] Implement login handler (~5 min)
  - Checkpoint: Returns token

...
```

## Contributing

Contributions are welcome! Feel free to submit issues and pull requests.

## License

MIT License - Free to use, modify, and distribute.

---

**Note**: Data is stored in local files (`data/` directory). Consider adding this directory to version control for backup, or exclude it via `.gitignore` for privacy.
