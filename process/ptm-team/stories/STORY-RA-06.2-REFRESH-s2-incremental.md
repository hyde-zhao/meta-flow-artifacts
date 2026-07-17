---
story_id: "ST-RA-06.2-REFRESH"
title: "S2 增量重算、差异报告"
status: "ready-for-verification"
priority: "P0"
wave: 3
source_cr: "CR-030"
source_story: "ST-RA-06"
feature: "FEAT-RA-ANALYSIS"
feature_design_refs:
  - "docs/features/feat-ra-analysis/DESIGN.md"
  - "docs/features/feat-ra-analysis/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["cross-module-contract", "data-model"]
  rationale: "增量重算策略、环比同比口径、差异报告"
depends_on:
  - "ST-RA-05.3-ANALYZE"
  - "ST-RA-06.1-DETECT"
output_files:
  - "skills/reverse-analysis/SKILL.md"
created_by: "meta-se"
created_at: "2026-07-16"
---

# ST-RA-06.2-REFRESH: S2 增量重算、差异报告

## dev_context

### 背景
在 reverse-analysis Skill 中追加 S2 增量分析能力。消费变更检测结果，仅重算受影响维度，产出环比同比和差异报告。

### 输出文件
- `skills/reverse-analysis/SKILL.md`（追加 S2 增量重算部分）

### 核心能力
1. 受影响维度检测：变更字段 → 分析维度映射
2. 增量重算：仅重算受影响维度的指标
3. 全量重算触发：映射/分析规则/口径版本变化
4. 环比同比：同口径窗口、基数、变化率、N/A/可信度
5. 差异报告：前后对比 + 变化量 + 变化率

### AI 可执行任务清单
1. 实现变更字段 → 分析维度映射
2. 实现增量重算引擎
3. 实现全量重算触发检测
4. 实现环比同比计算（含 N/A 条件）
5. 实现差异报告生成

## validation_context
- 关键验证：仅受影响维度重算；窗口不足 → N/A；规则变更 → 全量重算

## acceptance_criteria
1. 仅重算受影响维度（非全量）
2. 环比同比含窗口/基数/变化率/可信度/N-A原因
3. 窗口不足标记 N/A + 原因
4. 规则/口径版本变化触发全量重算
5. 差异报告含前后对比 + 变化量/变化率
