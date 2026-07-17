---
story_id: "ST-RA-04"
title: "闭环跟踪与有效性决策"
status: "ready-for-verification"
priority: "P1"
wave: 3
source_cr: "CR-030"
source_story: "ST-RA-04"
feature: "FEAT-RA-TRACKING"
feature_design_refs:
  - "docs/features/feat-ra-tracking/DESIGN.md"
  - "docs/features/feat-ra-tracking/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["cross-module-contract", "concurrency"]
  rationale: "行动项状态机、关闭条件、观察窗逻辑"
depends_on:
  - "ST-RA-03"
output_files:
  - "skills/improvement-tracker/SKILL.md"
  - "skills/improvement-tracker/templates/action-item.yaml"
  - "skills/improvement-tracker/templates/effectiveness-check.yaml"
  - "skills/improvement-tracker/templates/closure-decision.yaml"
created_by: "meta-se"
created_at: "2026-07-16"
---

# ST-RA-04: 闭环跟踪与有效性决策

## dev_context

### 背景
在 improvement-tracker Skill 中追加跟踪侧。基于已批准 CA/PA，管理行动项、检查有效性、审查观察窗并做出关闭/保持开放决策。

### 输出文件
- `skills/improvement-tracker/SKILL.md`（追加跟踪侧）
- 模板文件（action-item、effectiveness-check、closure-decision）

### 核心能力
1. Action Item 状态机：not-started→in-progress→done/overdue
2. Effectiveness Check：planned/passed/failed/inconclusive
3. Closure Decision：四个关闭条件（行动项完成+有效性通过+观察窗满足+无同类复发）
4. 关闭条件不可绕过

### 设计约束
- 只有人工 reviewer 能做关闭决定
- 不自动关闭 RA
- 30 天观察窗以 fixture/dry-run 验证规则

### AI 可执行任务清单
1. 实现 Action Item 状态机
2. 实现 Effectiveness Check 模型
3. 实现 Closure Decision 模型（四条件）
4. 实现不可绕过逻辑
5. 创建模板文件

## validation_context
- 关键验证：四条件全部满足 → 关闭；任一不足 → 保持开放；不自动关闭

## acceptance_criteria
1. Action Item 状态正确流转
2. 过期行动项标记 overdue
3. 四关闭条件全部满足才可关闭
4. 任一条件不足 → 保持开放 + residual_risks
5. 不自动关闭 RA
