# Goal Pilot - Claude Code 插件

[English](README.md) | [中文](README_CN.md)

个人目标达成指导员，将年度目标转化为可执行的每日任务，具备**动态复盘与校准**功能。

## 功能特性

- **结构化数据持久化**：复盘记录保存到 CSV 文件，状态追踪使用 JSON
- **分层上下文衰减**：越早的数据权重越低
- **自动校准**：基于检测到的模式自动调整任务
- **Slash 命令**：`/gp:setup`、`/gp:today`、`/gp:review`
- **子代理架构**：Planner、Calibrator、Domain Analyst
- **目标 → SOP 框架**：将高层目标转化为带季度里程碑的结构化计划
- **每日任务生成**：基于分层上下文（最近 7 天复盘、周摘要、pins）的优先级任务列表
- **逐步引导**：将每个任务分解为具体可执行的步骤
- **数据持久化**：进度保存到本地文件（`data/` 目录）
- **复盘系统**：结构化的日/周/月复盘，带校准功能
- **自适应规划**：基于偏离模式的自动任务调整
- **多语言**：支持 English、中文、日本語

## 安装

### 通过 Claude Code CLI

```bash
/plugin marketplace add github:duoduohe/goal-pilot-plugin
```

### 手动安装

```bash
git clone https://github.com/duoduohe/goal-pilot-plugin.git
cd goal-pilot-plugin
claude plugin install .
```

## 快速开始

### 首次设置

运行设置命令：

```
/gp:setup
```

或者自然描述你的目标：

```
我想在 12 个月内上线我的 SaaS 产品并达到 1 万美元 MRR。
目前有 MVP，每周可以投入 20 小时。
```

Claude 将会：
1. 创建 `data/state.json`，包含你的目标框架
2. 生成季度里程碑
3. 初始化复盘用的 CSV 文件
4. 生成第一天的任务

### 日常使用

```
/gp:today
```

或自然语言：

```
今天做什么？
What's today's task?
```

Claude 将会：
1. 加载你的目标和最近上下文
2. 应用衰减权重（最近的复盘更重要）
3. 检查校准调整
4. 生成 Top 3 成果及下一步行动

### 日复盘

```
/gp:review
```

Claude 将会：
1. 收集结构化字段（成果、阻塞、精力、情绪）
2. 保存到 `data/reviews_daily.csv`
3. 运行校准规则
4. 如检测到模式则应用自动调整

### 周/月复盘

```
/gp:review week
/gp:review month
```

## Slash 命令

| 命令 | 动作 |
|------|------|
| `/gp:setup` | 初始化目标，创建数据文件 |
| `/gp:today` | 基于分层上下文生成今日任务 |
| `/gp:review` | 日复盘（默认）|
| `/gp:review week` | 周复盘 + 生成摘要 |
| `/gp:review month` | 月复盘 + 生成摘要 |

## 自然语言

| 短语 | 映射到 |
|------|--------|
| "今天做什么" / "What's today's task?" | `/gp:today` |
| "做复盘" / "Do review" | `/gp:review` |
| "周复盘" / "Weekly review" | `/gp:review week` |
| "查看进度" / "Show progress" | 显示 state.json 摘要 |
| "重置目标" / "Reset goal" | 清除数据，重新开始 |

## 校准系统

### 自动任务调整

系统检测模式并自动调整：

| 检测到的模式 | 调整 |
|--------------|------|
| "下一步不明确" 7 天内出现 3+ 次 | 更小的步骤粒度（2-3 分钟）|
| "范围太大" 7 天内出现 2+ 次 | 将成果拆分为子成果 |
| "精力不足" 7 天内出现 3+ 次 | 低摩擦模式（仅 1 个深度工作项）|

### 进度偏差警报

| behind_ratio | 警报级别 | 动作 |
|--------------|----------|------|
| < 15% | 正常 | 正常运行 |
| 15-30% | 黄色 | 建议缩小范围 |
| > 30% | 红色 | 阻止直到解决 |

## 数据结构

### 文件结构

```
goal-pilot-plugin/
├── .claude-plugin/
│   └── plugin.json          # 插件清单
├── commands/
│   ├── setup.md             # /gp:setup 命令
│   ├── today.md             # /gp:today 命令
│   └── review.md            # /gp:review 命令
├── agents/
│   ├── planner.md           # 任务规划子代理
│   ├── calibrator.md        # 校准规则子代理
│   └── domain_analyst.md    # 领域专项分析
├── skills/
│   ├── goal-pilot/
│   │   ├── SKILL.md         # 主技能说明
│   │   ├── REVIEWS.md       # 复盘模板
│   │   ├── TEMPLATES.md     # 可复用模板
│   │   └── EXAMPLES.md      # 示例对话
│   └── goal-data/
│       └── SKILL.md         # 数据层操作
├── data/                    # 用户数据（设置时创建）
│   ├── state.json           # 目标、里程碑、校准
│   ├── reviews_daily.csv    # 日复盘记录
│   ├── reviews_weekly.csv   # 周复盘记录
│   ├── reviews_monthly.csv  # 月复盘记录
│   ├── summaries_weekly.csv # 周摘要 (L1)
│   ├── summaries_monthly.csv # 月摘要 (L2)
│   └── pins.csv             # 长期约束/教训
├── README.md
├── README_CN.md
└── LICENSE
```

### Memory（仅指针）

Claude Memory 现在只存储轻量指针：

```
GP_DATA_PATH: ./data
GP_LANG: zh
GP_LAST_SESSION_DATE: 2026-01-19
```

所有实际数据都在 `data/` 目录的文件中。

## 示例

### 设置流程

```
用户: /gp:setup

Claude: ## Goal Pilot 设置

让我们设置你的目标框架。请提供：

1. **目标陈述**：你想达成什么？
2. **目标日期**：什么时候？(YYYY-MM-DD)
3. **北极星指标**：什么数字衡量成功？
4. **领域**（可选）：fitness、english 等
5. **语言**：English/中文/日本語

用户: 到 2026-12-31 把 SaaS 做到 1 万美元 MRR。
北极星是 MRR。领域：产品、营销。用中文。

Claude: ## 设置完成！

**目标**: SaaS 达到 1 万美元 MRR
**截止**: 2026-12-31
**北极星**: MRR
**阶段**: Q1-验证

### 已创建里程碑
| Q | 阶段 | 截止 | 里程碑 |
|---|------|------|--------|
| Q1 | 验证 | 2026-03-31 | 第一个付费客户 |
| Q2 | 扩展 | 2026-06-30 | $2.5K MRR |
| Q3 | 系统化 | 2026-09-30 | $5K MRR |
| Q4 | 达成 | 2026-12-31 | $10K MRR |

运行 `/gp:today` 开始！
```

### 带校准的每日任务

```
用户: /gp:today

Claude: ## 今天: 2026年1月19日 周一

**目标**: SaaS → 1万美元 MRR
**阶段**: Q1-验证 | **进度**: 15%
**里程碑**: M1 还剩 71 天

**激活的调整**: 小步模式（检测到 unclear_next_action 3 次）

### Top 3 成果

#### 1. 完成用户认证 (P0)
**小步骤：**
- [ ] 创建认证路由文件 (~2 分钟)
  - 检查点：文件存在
- [ ] 定义用户 schema (~3 分钟)
  - 检查点：Schema 验证通过
- [ ] 实现登录处理器 (~5 分钟)
  - 检查点：返回 token

...
```

## 贡献

欢迎贡献！请随时提交 issues 和 pull requests。

## 许可证

MIT 许可证 - 可自由使用、修改和分发。


---

**注意**：数据存储在本地文件（`data/` 目录）。可以考虑将此目录加入版本控制以备份，或通过 `.gitignore` 排除以保护隐私。
