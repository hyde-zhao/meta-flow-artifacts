---
story_id: "ST-RA-06.2-REFRESH"
canonical_story_id: "ST-RA-06.2-REFRESH"
title: "S2 增量重算、差异报告"
source_cr: "CR-030"
feature: "FEAT-RA-ANALYSIS"
checkpoint: "CP6"
type: "auto"
status: "PASS"
author: "meta-dev"
created_at: "2026-07-16T15:15:00+00:00"
reviewed_at: null
implementation_evidence_ref: "process/stories/STORY-RA-06.2-REFRESH-s2-incremental-LLD.md"
---

# CP6 编码完成检查 — ST-RA-06.2-REFRESH: S2 增量重算与差异报告

## Entry Criteria

| # | 条件 | 状态 |
|:---:|---|---|
| 1 | Story 卡片存在且 LLD 设计证据已确认 | ✅ STORY-RA-06.2-REFRESH-s2-incremental-LLD.md（full-lld，14 章节，v1.1），CP5 已审批通过 |
| 2 | Feature 设计（`feat-ra-analysis/DESIGN.md`）已确认 | ✅ Feature 设计矩阵覆盖 |
| 3 | 前置 Story 已完成：ST-RA-05.3-ANALYZE（S1 管线）+ ST-RA-06.1-DETECT（变更检测） | ✅ S1 管线已完成（SKILL.md §6）；变更检测已完成（change_history 表 + ChangeResult） |
| 4 | 当前 Wave 可执行 | ✅ Wave 3，文件所有权无冲突（shared: reverse-analysis SKILL.md，本 Story 写 §9） |
| 5 | `dev_gate` 满足 | ✅ 前置 Story 的 §6（S1 管线）+ §1-§5（六维引擎）已完成并验证 |

## Exit Criteria Checklist

| # | 项目 | 结果 | 证据 |
|:---:|---|:---:|---|
| 1 | 输出文件存在且非空 | ✅ PASS | 1 个文件共 +982 行（SKILL.md §9: 982 行） |
| 2 | 文件名符合 kebab-case 规范 | ✅ PASS | —（仅修改已有文件） |
| 3 | 未修改 `REQUIREMENTS.md`、`HLD.md`、`ARCHITECTURE-DECISION.md` | ✅ PASS | 未触碰 |
| 4 | DEV-LOG 已追加 | ✅ PASS | 见本文件实现摘要 |
| 5 | 实现摘要已生成 | ✅ PASS | 见下文实现执行证据 |
| 6 | 实现对象清单可追溯 | ✅ PASS | 1 个文件（SKILL.md §9） |
| 7 | 设计契约映射完整 | ✅ PASS | 见下文契约映射 |
| 8 | 验收标准 5 条全部覆盖 | ✅ PASS | 见下文验收标准覆盖 |
| 9 | 仅重算受影响维度（增量策略） | ✅ PASS | SKILL.md §9.2 变更字段→维度映射表（13 字段×6 维度）+ §9.3 逐维度增量策略 |
| 10 | 全量重算触发检测（规则版本变更） | ✅ PASS | SKILL.md §9.5: schema_version/mapping_version/analysis_rule_version 三步检测 |
| 11 | 环比同比含窗口/基数/变化率/可信度/N-A 原因 | ✅ PASS | SKILL.md §9.5.6 + §9.7.1 comparison 段: 7 种 N/A 条件 |
| 12 | 差异报告含前后对比 + 变化量/变化率 | ✅ PASS | SKILL.md §9.6 + §9.7.1: dimension_diffs + significant_changes（默认 20% 阈值） |
| 13 | comparison_batch_ref 管理 | ✅ PASS | SKILL.md §9.4: 5 场景取值 + 3 步合法性校验 |
| 14 | analysis_run 创建（recompute_mode=incremental|full） | ✅ PASS | SKILL.md §9.7.3: insert_analysis_run() + comparison_batch_ref |
| 15 | 安全约束：不覆盖历史 run/报告，措施建议不自动执行 | ✅ PASS | SKILL.md §9.9: 6 项 P-S2 禁止项 + 允许操作范围 |
| 16 | 共享写入边界正确（§9 未覆盖 §1-§8） | ✅ PASS | §9 从第 2280 行开始 |

## 实现执行证据

### 实现前置检查

| 检查项 | 结论 |
|---|---|
| Story 状态 | `ready-for-verification`，LLD 已确认（CP5 已审批），实现产物已存在 |
| 前置依赖 ST-RA-05.3-ANALYZE | ✅ `skills/reverse-analysis/SKILL.md` §6 S1 管线已就绪（670 行，逐单+批量） |
| 前置依赖 ST-RA-06.1-DETECT | ✅ change_history 表 + ChangeResult 可消费 |
| 前置依赖 ST-RA-02（六维引擎） | ✅ §3-§5 已就绪，可供 S2 复用 |
| 文件所有权 | `skills/reverse-analysis/SKILL.md` shared（FEAT-RA-ANALYSIS），本 Story 写 §9 |
| LLD 与 Feature 设计一致性 | ✅ full-lld v1.1，14 章节，CP5 B12 整改已闭环 |
| 实现形态 | Skill Markdown 规则定义（§9 替换占位符），低风险声明式规范 |

### 实现对象清单

| # | 文件 | 操作 | 行数 | 状态 |
|:---:|---|---|---|---|
| 1 | `skills/reverse-analysis/SKILL.md` | 修改（替换 §9 占位符） | +982 | ✅ |

### 设计契约映射

| LLD 章节 | 实现位置 | 覆盖率 |
|---|---|---|
| LLD §2.1 S2 增量重算入口契约 | SKILL.md §9.1.1 S2 入口判定 + §9.1.3 前置校验 | 触发条件（new_batch_id+baseline_batch_id+change_set）+ 4 项前置校验 |
| LLD §2.2 变更字段→维度映射表 | SKILL.md §9.2.1 | 13 个 ticket 字段 × 6 个分析维度完整映射 |
| LLD §2.3 环比同比接口 | SKILL.md §9.5.6 环比同比 N/A 判定 + §9.7.1 comparison 段 | mode/window/metrics/credibility/na_reasons 齐全 |
| LLD §3.1 差异报告数据模型 | SKILL.md §9.7.1 完整 YAML 结构 | change_summary + 6 维 dimension_diffs + comparison + improvement_refresh + skipped_tickets |
| LLD §3.2 SQLite 表依赖 | SKILL.md §9.1.2 change_set 消费 + §9.4.2 合法性校验 + §9.10.1 上游依赖 | 5 张表（ticket/ticket_version/ingestion_batch/analysis_run/change_history） |
| LLD §4 核心流程（8 步） | SKILL.md §9.7.4 S2 管线 7 步总览 | 合并为 7 步（差异生成+环比同比合并），逻辑完整 |
| LLD §5 状态机（recompute_mode） | SKILL.md §9.5 | full/incremental 决策树 + 条件表 + 规则版本变更检测 |
| LLD §6 错误处理与降级 | SKILL.md §9.8 | 10 种异常场景 × 处理方式 |
| LLD §7 测试设计 | 本文件验收标准覆盖 | 14 个测试场景（7 正向 + 7 负向/边界）+ 7 组 Fixture |
| LLD §8 安全与权限 | SKILL.md §9.9 | 6 项 P-S2 禁止项 + 允许操作范围表 |
| LLD §9 实施步骤（5 TASK-ID） | SKILL.md §9 各子节 | TASK-ANL-21~25 全部完成 |
| LLD §10 回滚策略 | 通过 analysis_run 分离设计覆盖 | 新 run 不覆盖旧 run，回滚只需删除新 run |
| LLD §12 集成契约 | SKILL.md §9.10 | 上游 4 Story + 下游 3 Story + S1/S2 边界对比表 |
| LLD §13 开放项 | SKILL.md §9.11 | 5 项 OPEN 决策/假设 |

### 验收标准覆盖

| # | 验收标准 | 结果 | 证据 |
|:---:|---|---|---|
| AC-1 | 仅重算受影响维度（非全量） | ✅ PASS | SKILL.md §9.2.1 映射表 + §9.3.2 逐维度增量策略（6 维度各有独立策略）+ §9.3.3 增量重算执行流程 |
| AC-2 | 环比同比含窗口/基数/变化率/可信度/N-A 原因 | ✅ PASS | SKILL.md §9.5.6 环比同比 N/A 7 条件 + §9.7.1 comparison 段（mode/baseline_period/current_period/metrics/credibility/na_reasons） |
| AC-3 | 窗口不足标记 N/A + 原因 | ✅ PASS | SKILL.md §9.5.6: insufficient_window/insufficient_sample/denominator_change/rule_version_change 4 种 N/A 条件 |
| AC-4 | 规则/口径版本变化触发全量重算 | ✅ PASS | SKILL.md §9.5.2: schema_version/mapping_version/analysis_rule_version 三步检测 + §9.5.3 full 模式 |
| AC-5 | 差异报告含前后对比 + 变化量/变化率 | ✅ PASS | SKILL.md §9.6.1 差异计算流程 + §9.7.1 dimension_diffs（before_summary/after_summary/significant_changes）|

### 单元测试与 Fixture 计划

| 说明 |
|---|
| 本 Story 输出为 Skill Markdown 规则定义，实现形态为声明式规范文本。 |
| LLD §7 定义了 14 个测试场景：正向（T-ANL-16/17/S04-01/S04-02/S04-03 等 7 个）+ 负向/边界（T-ANL-18/19/S04-04~07 等 7 个）。 |
| Fixture 计划：7 组 fixture（s2_before_after_datasets / s2_rule_version_change / s2_empty_change_set / s2_zero_baseline + 3 组辅助 fixture）覆盖正向和边界场景。 |
| 声明式规范的正确性通过 CP6 checklist、LLD 设计契约映射和结构完整性校验保证。 |
| 运行时验证由 ptm-tse Agent 消费方在 CP7 执行。 |

### 最小实现切片

| 切片 | 内容 | 依赖 | 状态 |
|:---:|---|---|---|
| 1 | §9.1 增量重算触发 + §9.2 受影响维度判定 | ST-RA-06.1-DETECT change_history 表 | ✅ |
| 2 | §9.3 增量重算策略 + §9.4 comparison_batch_ref 管理 | 切片 1 | ✅ |
| 3 | §9.5 recompute_mode 判定 + §9.6 差异报告生成 | 切片 2 | ✅ |
| 4 | §9.7 差异报告输出格式 + §9.8-§9.13 补充章 | 切片 3 | ✅ |

### 平台差异处理

| 结论 |
|---|
| N/A。本 Story 仅追加 Skill Markdown 规则定义到 `skills/reverse-analysis/SKILL.md`，不涉及平台特定路径或代码。差异报告的环比同比计算为纯数据聚合逻辑，所有平台一致。LLD §11 已确认无平台差异。 |

### 验证结果

| # | 验证项 | 方法 | 结果 |
|:---:|---|---|---|
| 1 | SKILL.md §9 结构完整 | 人工审查 | ✅ §9 含 13 个子节（9.1-9.13），逐节含 sub-章节 |
| 2 | 变更字段→维度映射表覆盖 13 字段 | 人工审查 | ✅ 13/13 ticket 字段有目标维度（root_cause/product/module/severity/status/test_missed_analysis/test_missed_phase/improvement_measures/openeddate/resolveddate/title/description + 新增 ticket） |
| 3 | 受影响维度计算流程完整 | 人工审查 | ✅ DM-1→DM-4 四步：提取变更字段→查表→派生逻辑→输出 affected_dimensions |
| 4 | 逐维度增量策略 6 维度独立 | 人工审查 | ✅ RC/PQ/ESC/TM/IMP/CMP 各维度有独立策略（重算/复用/部分重算） |
| 5 | 聚合维度增量合并规则 | 人工审查 | ✅ PQ/TM/IMP 三维度的增量合并示例 |
| 6 | comparison_batch_ref 5 场景覆盖 | 人工审查 | ✅ S1 首次/规则不变/规则变更/手动指定/无历史 |
| 7 | recompute_mode 判定流程 | 人工审查 | ✅ 规则版本变更→full / 有受影响维度→incremental / 无→跳过 |
| 8 | 差异报告 YAML 结构完整 | 人工审查 | ✅ change_summary + 6 维 dimension_diffs + comparison + improvement_refresh + skipped_tickets |
| 9 | 环比同比 N/A 7 条件 | 人工审查 | ✅ insufficient_window / insufficient_sample / denominator_change / rule_version_change / zero_baseline / no_baseline / quality_concern |
| 10 | 安全禁止项 6 条 | 人工审查 | ✅ P-S2-01~06: 不覆盖历史/不修改 ticket/不自动执行措施/不推断缺失数据/不覆盖 baseline/仅 DAO 写入 |
| 11 | S1/S2 协同路由 | 人工审查 | ✅ §9.13: S1/S2 角色分配表 + 入口路由规则 |
| 12 | 错误处理 10 场景 | 人工审查 | ✅ baseline 不存在/window 不足/样本不足/规则变更/某维度异常/措施不可用等 |
| 13 | 开放项 5 项声明 | 人工审查 | ✅ OPEN-RA062-01~05 逐项标注状态/回访条件/当前策略 |
| 14 | 不覆盖 §1-§8 | 人工审查 | ✅ §9 从第 2280 行开始，§1-§5（资格检查/可信输入/六维引擎/根因状态机/指标定义）不变，§6（S1 管线）不变 |

### 未覆盖项

| 项 | 说明 |
|---|---|
| 实际运行验证 | 本 Story 为声明式规范，运行时正确性由 ptm-tse Agent 消费方在 CP7 验证 |
| ST-RA-06.3-TRACK 集成 | improvement_refresh 的消费逻辑在 ST-RA-06.3 中实现，当前链路未端到端验证 |
| metric-definition.yaml 版本信号 | analysis_rule_version 变更检测需 metric-definition.yaml 的实际版本信号 |

### 设计缺口反馈

1. SKILL.md §9.5.2 规则版本变更检测中的 `analysis_rule_version` 来源为 `metric-definition.yaml`，当前 Skill 只定义了检测逻辑，未定义如何从 YAML 模板文件中提取活跃版本号。需在 metric-definition.yaml 中显式声明 `active_metric_version` 字段。
2. SKILL.md §9.2.1 映射表中 `title` 和 `description` 变更标记为"影响根因"，但该影响的精确性取决于实际 AI 分析对标题/描述字段的依赖程度，需在 runtime 验证中确认。

### 后续交接

| 接收方 | 交付内容 |
|---|---|
| ST-RA-03（改进治理） | 差异报告中的新 CA/PA 候选 |
| ST-RA-06.3-TRACK（措施基线管理） | 措施刷新提示（keep/needs_review/invalidated） |
| meta-qa (CP7) | 本 CP6 文件 + SKILL.md §9 |

#### 给 meta-qa 的验证入口

| 入口 | 路径 | 验证内容 |
|------|------|---------|
| **SKILL.md 结构完整性** | `skills/reverse-analysis/SKILL.md` §9 | 验证 §9.1-§9.13 共 13 个子章节存在且非空 |
| **设计契约映射** | 本文件「设计契约映射」表 | 验证 14/14 LLD 章节有对应 §9 实现 |
| **变更字段映射覆盖** | §9.2.1 映射表 | 验证 13/13 ticket 字段有映射目标维度 |
| **安全禁止项** | §9.9 | 验证 6 项 P-S2-* 禁止项 + 允许操作范围完整 |
| **差异报告 YAML 结构** | §9.7.1 | 验证 YAML 结构覆盖 change_summary/dimension_diffs/comparison/improvement_refresh/skipped_tickets |
| **错误处理矩阵** | §9.8 | 验证 10 场景覆盖 blocked/skipped/N-A/degraded |
| **前后一致性** | §1-§8 vs §9 | 验证 §9 未修改 §1-§8 |
| **前端元数据** | SKILL.md frontmatter | 验证 version/shared_writers 描述更新 |

#### 风险提示

| 风险 | 严重度 | 说明 |
|------|--------|------|
| change_set 格式变化 | medium | 如 ST-RA-06.1-DETECT 后续调整 change_history 表结构，§9.1.2 的消费逻辑需同步更新 |
| 环比同比阈值误用 | low | 默认阈值 10 可能导致小窗口误 N/A，但可通过 metric-definition.yaml 覆盖 |
| 措施刷新提示未闭环 | medium | improvement_refresh 的 keep/needs_review/invalidated 提示需要 ST-RA-06.3-TRACK 消费；当前链路未端到端验证 |

## Agent Dispatch Evidence

| 字段 | 值 |
|---|---|
| dispatch_mode | inline-fallback（host-orchestrator 直接指派本 Story CP6 补齐任务） |
| agent | meta-dev |
| thread_id | 当前对话主线程 |
| implemented_at | 2026-07-16T15:15:00+00:00 |

## 检查结论

**CP6: PASS** — 全部 16 项 Exit Criteria 通过。1 个输出文件共 +982 行（SKILL.md §9: 982 行，替换原 11 行占位符为完整 S2 增量重算章节）。LLD 14 章节设计契约完整映射，5 条验收标准全部覆盖。13 个子章节覆盖：增量重算触发、变更字段→维度映射（13×6）、逐维度增量策略（6 维度独立）、comparison_batch_ref 管理、recompute_mode 判定（full/incremental）、差异报告生成（before/after+显著变化 20% 阈值）、环比同比（7 N/A 条件）、措施刷新提示（4 建议）、错误处理（10 场景）和安全禁止项（6 条 P-S2）。Story 可移交 meta-qa 进行 CP7 验证。
