---
story_id: "ST-NRA-01"
title: "证据不足保护（拒绝虚假根因）"
status: "planned"
priority: "P0"
wave: 2
source_cr: "CR-030"
source_story: "ST-NRA-01"
feature: "FEAT-RA-ANALYSIS"
feature_design_refs:
  - "docs/features/feat-ra-analysis/DESIGN.md"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: []
  rationale: "负向场景，正向 Story LLD 覆盖阈值逻辑，卡片内补充"
depends_on:
  - "ST-RA-01"
output_files:
  - "skills/reverse-analysis/SKILL.md"
created_by: "meta-se"
created_at: "2026-07-16"
---

# ST-NRA-01: 证据不足保护

## dev_context

### 技术说明
在 reverse-analysis Skill 中追加硬阻断逻辑：少于三条有效证据线时，根因状态不可标记为 confirmed。输出缺口清单（缺失 Owner + 待澄清状态）。

- 三线阈值在 ST-RA-01 中定义，此处实现硬阻断
- 缺口清单包含：缺失证据线、建议补充方、待澄清状态

## validation_context
- 关键验证：2 条有效线 + 尝试确认 → 拒绝；0 条有效线 → 拒绝

## acceptance_criteria
1. <3 条有效线时根因状态不能为 confirmed
2. 缺口清单输出缺失 Owner + 待澄清状态

## 技术说明

### 设计依据

本 Story 是 ST-RA-01（资格检查与可信输入建立）中定义的三线阈值的**硬阻断实现**。HLD REV-03 要求「根因四层不可自动跃迁」；ST-RA-01 LLD 定义了 `valid_count >= 3` 的阈值，本 Story 负责将阈值转化为不可绕过的阻断。

### 文件影响

| 文件 | 操作 | 写入段落 | 与正向 Story 的冲突 |
|------|------|---------|------------------|
| `skills/reverse-analysis/SKILL.md` | 修改 | 在 ST-RA-01 的证据边界段内追加硬阻断规则 | 依赖 ST-RA-01 先创建资格检查段（串行约束已由 CP4 确认） |

### 接口/数据/权限变化

**无新增接口**。本 Story 在 ST-RA-01 的证据分类结果上增加两步硬阻断：
1. `valid_count < 3` 时，根因状态机的 `ai_candidate → evidence_backed` 转换被硬阻断
2. 输出缺口清单，包含：缺失证据线（line_id + gap 原因）→ 建议补充方（建议 Owner）→ 待澄清状态

阻断规则：
- `valid_count ∈ {0, 1, 2}` → `transition_blocked = true` + `gap_report`
- `valid_count >= 3` → `transition_allowed = true`（正常进入 evidence_backed）

### 异常和回退

- 阻断不可被绕过，唯一解除方式是补充有效证据线（`valid_count >= 3`）
- 如果 reviewer 认为已有证据足够但 valid_count < 3：需 reviewer 通过人工流程将证据线标记为 valid，不通过自动机制

### 测试入口

| 场景 | fixture | 预期 |
|------|---------|------|
| `valid_count = 0` + 尝试确认 | insufficient_evidence_ticket.json（所有字段为空） | 阻断，输出完整缺口清单 |
| `valid_count = 2` + 尝试确认 | 同 fixture（仅 2 条可分类字段） | 阻断，输出 1 条缺口 |
| `valid_count = 3` + 尝试确认 | eligible_p1_ticket.json | 通过，进入 evidence_backed |

### 风险与重访条件

| 风险 | 缓解 | 重访条件 |
|------|------|---------|
| 阈值定为 3 可能过于机械（对于简单事件） | 保持 3 作为统一阈值 | 收集 10+ 个分析样本后评估是否需要按事件类型差异化阈值 |
| 证据线有效性判定依赖分类规则 | 分类规则在 ST-RA-01 LLD 中版本化 | 分类规则变更时同步更新阈值检查 |

### 偏离记录

无偏离。完全按 HLD REV-03 的「三线阈值：<3 条有效证据线 → 禁止 confirmed」和「事实/假设/未知项分离」实现。
