---
story_id: "ST-RA-01"
title: "资格检查与可信输入建立"
status: "ready-for-verification"
priority: "P0"
wave: 2
source_cr: "CR-030"
source_story: "ST-RA-01"
feature: "FEAT-RA-ANALYSIS"
feature_design_refs:
  - "docs/features/feat-ra-analysis/DESIGN.md"
  - "docs/features/feat-ra-analysis/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["cross-module-contract", "security"]
  rationale: "资格规则、证据阈值、越权拒绝逻辑；跨 INGESTION 读取"
depends_on:
  - "ST-RA-INGEST-DB"
output_files:
  - "skills/reverse-analysis/SKILL.md"
  - "agents/ptm-tse.md"
created_by: "meta-se"
created_at: "2026-07-16"
---

# ST-RA-01: 资格检查与可信输入建立

## dev_context

### 背景
创建 reverse-analysis Skill 的第一部分：入口资格控制。决定了哪些事件可以进入分析管线，哪些应该被拒绝或 defer。

### 输入文件
- `docs/features/feat-ra-analysis/DESIGN.md` §2.1、§3（根因四层）
- `data/schema.sql` — ticket 表结构参考
- `docs/design/HLD.md` REV-03 — 安全约束

### 输出文件
- `skills/reverse-analysis/SKILL.md`（资格检查 + 证据边界部分）
- `agents/ptm-tse.md`（更新，添加 reverse-analysis Skill 引用）

### 设计约束
- P1 必做、P2 可选、P3/P4 不自动进入
- 内部问题标记 deferred
- 三线阈值：<3 条有效证据线 → 禁止 confirmed
- 事实/假设/未知项分离

### AI 可执行任务清单
1. 创建 `skills/reverse-analysis/` 目录和 SKILL.md 骨架
2. 实现 P1/P2/P3/P4 资格判定
3. 实现内部问题识别与拒绝
4. 实现五条证据线分类
5. 实现三线阈值检查
6. 更新 `agents/ptm-tse.md` 引用

## validation_context

### 验证入口
- fixture 测试：P1 → eligible；P3 → rejected；内部问题 → deferred

### 关键验证场景
1. P1 事件标记 eligible
2. P2 事件显式选择
3. P3/P4 不自动进入
4. 内部问题标记 deferred + 说明
5. <3 条有效证据线时状态不能为 confirmed

## acceptance_criteria

1. P1 必做、P2 可选、P3/P4 不自动进入
2. 内部问题正确识别并标记 deferred
3. 五条证据线分为事实/假设/未知项
4. <3 条有效线时根因状态不能为 confirmed
5. `agents/ptm-tse.md` 正确引用 reverse-analysis
