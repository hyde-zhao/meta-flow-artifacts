---
title: "CR112 CR Tracking Checker Expectation Notes"
cr_id: "CR-112"
status: "closed-ready"
created_at: "2026-06-22T15:49:13+08:00"
owner: "host-orchestrator"
scope: "no-code-no-runtime-governance"
---

# CR112 CR Tracking Checker Expectation Notes

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-06-22 | host-orchestrator | 创建 CR112 CP2 draft，定义 cr-tracking checker 的状态源、行生命周期、warning 边界和非范围。 |
| v0.2 | 2026-06-22 | host-orchestrator | 回填 CP2 approval，确认 no-code/no-runtime expectation notes scope、默认说明优先级和不授权边界。 |
| v0.3 | 2026-06-22 | host-orchestrator | 准备 CP8 终验，补充关闭范围和后续实现候选分流。 |
| v0.4 | 2026-06-22 | host-orchestrator | 回填 CP8 approval，关闭 CR112 为 READY；`FU-CR112-001` 仅登记为 candidate。 |

## 目标

CR112 的目标是把 `meta-flow check cr-tracking --project-root .` 的治理消费期望写清楚，降低恢复上下文、人工审查和后续实现 CR 之间的误读风险。

本文件只定义 expectation notes，不修改源码、不修改 checker implementation、不启动 runtime，也不批量重写历史长表。

## 状态源优先级

| 优先级 | 状态源 | 用途 | 期望 |
|---:|---|---|---|
| 1 | `process/changes/CR-INDEX.yaml.active_crs[]` | 当前 active formal CR 汇总入口 | 判断是否存在当前正式执行锁的首要来源。 |
| 2 | Formal CR frontmatter | 单个 CR 生命周期 / 门禁状态 | `lifecycle_status`、`readiness_status`、`gate_status` 必须与索引一致。 |
| 3 | Follow-up tracking row | 候选项 / legacy ID 生命周期 | candidate 不占执行锁；转正式 CR 后必须有 `formal_cr_path` 和 legacy 映射。 |
| 4 | `process/STATE.md` 顶层摘要 | 人类恢复上下文入口 | 只能作为当前摘要；冲突时以 CR-INDEX / formal CR 为机器真相源。 |
| 5 | 历史长表 / archived snapshots | 审计历史 | 不作为 current failure；只可产生 warning 或 audit note。 |

## 行生命周期期望

| 行类型 | `lifecycle_status` | `readiness_status` | `gate_status` | 是否占 active lock |
|---|---|---|---|---|
| 未启动 candidate | `candidate` | `n/a` | `not_started` | 否 |
| 已选择正式 CR | `active` | `ready` | `cp2_pending` / `cp3_pending` / `cp5_pending` / `cp8_pending` | 是 |
| 已关闭正式 CR | `closed` | `ready` / `ready_with_risk` | `closed` | 否 |
| blocked 正式 CR | `blocked` | `blocked` | `blocked` | 否；但必须保留阻塞理由 |
| superseded / cancelled | `superseded` / `cancelled` | `n/a` 或历史值 | `closed` | 否 |

## `gate_status` 枚举

CR112 建议 checker expectation notes 固定使用以下枚举，不把自然语言写入机器字段：

- `not_started`
- `cp2_pending`
- `cp3_pending`
- `cp5_pending`
- `cp8_pending`
- `closed`
- `blocked`

自然语言说明应写入 `next_action`、`blocked_by`、`last_consistency_check` 或 checkpoint 正文。

## Warning Boundary

| 情况 | 期望严重度 | 说明 |
|---|---|---|
| `CR-INDEX.active_crs[]` 与 formal CR frontmatter 不一致 | failure | 当前机器真相源冲突。 |
| 被选择的 follow-up 缺少 formal CR 或 `formal_cr_path` | failure | active formal CR 映射不完整。 |
| candidate 行仍为 `not_started` 且无 active 映射 | ok | candidate 不占 active lock。 |
| 历史长表中的旧 `active_change` | warning-only / audit-only | 不代表 current state。 |
| `process/STATE.md` 顶层摘要陈旧 | warning 或 failure，取决于是否污染 current active lock | 若顶层摘要误报 active CR，应优先修顶层摘要；历史块不批量重写。 |

## 非范围

| 非范围项 | 原因 | 未来处理方式 |
|---|---|---|
| checker implementation change | CR112 只写 expectation notes | 若需要改变 `meta-flow check cr-tracking` 行为，另起 implementation CR。 |
| 源码 / tests 修改 | 当前未授权 | 另起源码实现 CR。 |
| 历史长表批量重写 | 审计风险高，且不属于 current failure | 独立审计清理 CR。 |
| CR105 / order-write / simulation / live | 高风险 runtime / trading 路线 | 独立 authorization gate。 |
| CR089 恢复 | 当前保持 blocked | 独立恢复 CR。 |
| NAS / credentials / publish | 外部资源与凭据边界 | 独立 authorization gate。 |

## CP2 决策项

| 决策 ID | 决策类型 | 推荐方案 | 风险 | 回退 |
|---|---|---|---|---|
| DQ-CP2-CR112-01 | scope | CR112 只做 no-code/no-runtime checker expectation notes gate | 风险低；只产生治理说明 | 暂停或退回 candidate。 |
| DQ-CP2-CR112-02 | follow_up_tracking | 优先整理 source of truth、row lifecycle、`gate_status` enum、warning boundary | 若实际 checker 行为不同，只登记实现候选 | 另起 implementation CR。 |
| DQ-CP2-CR112-03 | runtime_authorization | 继续禁止源码、checker implementation、CR105、CR089、runtime、交易写、NAS、凭据、publish | 高风险工作继续延期 | 用户明确要求时另起独立 gate。 |

## 验收标准

| 标准 | 验收方式 |
|---|---|
| CR112 active formal CR 状态可追踪 | `meta-flow check cr-tracking --project-root .` 通过。 |
| CP2 人工门禁结构完整 | `meta-flow check human-gate --checkpoint process/checkpoints/CP2-CR112-CR-TRACKING-CHECKER-EXPECTATION-NOTES-REVIEW.md` 通过。 |
| 不授权边界清晰 | CR112、context capsule、CP2 checkpoint 均列出禁止项。 |
| 实现修改未被隐式启动 | 源码 / tests / checker implementation 均不改动。 |

## CP2 确认结果

| 字段 | 内容 |
|---|---|
| 结论 | approved |
| 时间 | 2026-06-22T15:55:31+08:00 |
| 接受范围 | no-code/no-runtime checker expectation notes gate。 |
| 接受优先级 | active formal CR source of truth、follow-up row lifecycle、`gate_status` enum、warning-only/current-vs-audit-history 语义。 |
| 继续禁止 | 源码修改、checker implementation change、CR105、CR089 恢复、runtime、交易写、NAS、凭据、publish。 |

## CP8 收口口径

| 项目 | 收口结论 |
|---|---|
| Active formal CR source of truth expectation | 以 `CR-INDEX.active_crs[]` 和 formal CR frontmatter 为当前机器真相源。 |
| Follow-up row lifecycle expectation | candidate 不占 active lock；转正式 CR 后必须有 `formal_cr_path` 和 legacy 映射。 |
| `gate_status` enum expectation | 只使用 `not_started`、`cp2_pending`、`cp3_pending`、`cp5_pending`、`cp8_pending`、`closed`、`blocked`。 |
| Warning-only / current-vs-audit-history boundary | 历史长表旧 `active_change` 只作为 audit-history，不作为 current failure。 |
| 实现对齐 | 不在 CR112 内实现；如未来需要，使用 `FU-CR112-001` 独立实现 CR。 |

## CP8 关闭结果

| 字段 | 内容 |
|---|---|
| 结论 | approved / READY |
| 时间 | 2026-06-22T16:06:10+08:00 |
| 关闭范围 | 状态源优先级、follow-up row lifecycle、`gate_status` enum、warning-only/current-vs-audit-history boundary。 |
| 后续候选 | `FU-CR112-001 Checker Implementation Alignment Candidate` 仅登记为 candidate，不启动。 |
| 继续禁止 | 源码修改、tests 修改、checker implementation change、CR105、CR089 恢复、runtime、交易写、NAS、凭据、publish。 |
