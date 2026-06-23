---
title: "CR111 STATE Summary Staleness Cleanup Policy"
cr_id: "CR-111"
status: "closed-ready"
created_at: "2026-06-22T15:15:41+08:00"
owner: "host-orchestrator"
scope: "no-code-no-runtime-governance"
---

# CR111 STATE Summary Staleness Cleanup Policy

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-06-22 | host-orchestrator | 创建 CR111 CP2 draft，定义 STATE 顶层摘要陈旧处理范围、非范围和决策项。 |
| v0.2 | 2026-06-22 | host-orchestrator | 回填 CP2 approval，确认顶层摘要优先、历史长表 audit-only、不授权源码 / checker / runtime。 |
| v0.3 | 2026-06-22 | host-orchestrator | 回填 CP8 approval，关闭 CR111 为 READY；不新增自身后续候选。 |

## 目标

CR111 的目标是把 `process/STATE.md` 的人类摘要陈旧处理规则状态化，避免后续恢复上下文时把历史长表、旧 snapshot 或过期 `next_action` 误读为当前执行锁。

本文件只定义治理策略，不修改源码、不修改 checker implementation、不启动 runtime，也不批量重写历史长表。

## 状态源优先级

| 优先级 | 状态源 | 用途 | 处理规则 |
|---:|---|---|---|
| 1 | `process/changes/CR-INDEX.yaml.active_crs[]` | 当前 active formal CR 汇总入口 | 判断当前是否存在正式执行锁的首要来源。 |
| 2 | Formal CR frontmatter | 单个 CR 的生命周期 / 门禁状态 | 必须与 CR-INDEX 同步；冲突时先修索引和正式 CR 证据。 |
| 3 | Follow-up tracking row | 候选项 / 别名生命周期 | candidate 不占执行锁；转正式 CR 后必须指向 `formal_cr_path`。 |
| 4 | `process/STATE.md` 顶层摘要 | 人类恢复上下文入口 | 必须刷新 `active_change`、`last_action`、`next_action` 和 `cr_tracking` 摘要。 |
| 5 | `process/STATE.md` 历史长表 / archived snapshots | 审计历史 | 不作为 current failure；不在 CR111 中批量重写。 |

## 清洁范围

| 对象 | 是否纳入 CR111 | 默认动作 | 理由 |
|---|---|---|---|
| 顶层 `active_change` | 是 | 与 CR-INDEX active formal CR 同步 | 恢复上下文时最容易被当作当前执行锁。 |
| 顶层 `last_action` | 是 | 改为最近用户动作和 CR111 启动事实 | 旧推送建议已过期，继续保留会误导下一步。 |
| 顶层 `next_action` | 是 | 改为 CP2 review / approve 路线 | 当前下一步是 CR111 CP2 门禁。 |
| `cr_tracking.status` | 是 | 改为 `active-formal-cr` | 与 CR-INDEX 同步。 |
| `cr_tracking.last_consistency_check` | 是 | 记录 CR111 active / CP2 pending | 便于后续检查点恢复。 |
| 历史长表中的旧 `active_change` | 否 | 保持 audit-history | 批量改写会破坏追溯，也不是 current state。 |
| checker implementation | 否 | 保持不变 | CR111 不授权代码层整改。 |

## 陈旧分类

| 类型 | 判定条件 | 默认处理 |
|---|---|---|
| current-summary-stale | 顶层摘要与 CR-INDEX active CR 不一致 | 在当前 CR 内刷新顶层摘要和 `cr_tracking` 小节。 |
| completed-action-stale | `next_action` 指向已经完成的提交、推送或检查 | 刷新为当前门禁动作，并在 `last_action` 说明已完成事项。 |
| audit-history-reference | 历史长表、旧 snapshot、旧 decision queue 中的旧 CR 引用 | 不改写；需要时作为 audit-only 说明。 |
| checker-expectation-gap | 机器检查对 current / history 的期望不清晰 | 不在 CR111 修改；登记为后续 checker expectation notes 候选。 |

## 非范围

| 非范围项 | 原因 | 未来处理方式 |
|---|---|---|
| 源码修改 | CR111 是 no-code governance gate | 另起 implementation CR。 |
| checker implementation change | 用户未授权，且 FU-CR110-002 已覆盖 expectation notes 候选 | 先启动 FU-CR110-002 或单独实现 CR。 |
| 历史长表批量重写 | 风险高、审计价值高、收益低 | 仅在明确审计清理 CR 中处理。 |
| CR105 / order-write / simulation / live | 高风险 runtime / trading 路线 | 必须独立授权。 |
| CR089 恢复 | 当前保持 blocked，不由 CR111 隐式恢复 | 必须独立恢复 CR。 |
| NAS / credentials / provider publish | 外部资源和凭据边界 | 必须独立 authorization gate。 |

## CP2 决策项

| 决策 ID | 决策类型 | 推荐方案 | 风险 | 回退 |
|---|---|---|---|---|
| DQ-CP2-CR111-01 | scope | CR111 只做 no-code/no-runtime STATE summary staleness cleanup gate | 风险低；只产生治理证据 | 暂停或退回 candidate。 |
| DQ-CP2-CR111-02 | follow_up_tracking | 优先处理顶层 STATE 摘要和 `cr_tracking` 小节；历史长表 audit-only 不批量重写 | 可能仍能搜索到历史旧引用，但不再代表 current state | 如需历史清理，另起 CR。 |
| DQ-CP2-CR111-03 | runtime_authorization | 继续禁止源码、checker implementation、CR105、CR089、runtime、交易写、NAS、凭据、publish | 高风险工作继续延期 | 用户明确要求时另起独立 gate。 |

## 验收标准

| 标准 | 验收方式 |
|---|---|
| CR111 active formal CR 状态可追踪 | `meta-flow check cr-tracking --project-root .` 通过。 |
| CP2 人工门禁结构完整 | `meta-flow check human-gate --checkpoint process/checkpoints/CP2-CR111-STATE-SUMMARY-STALENESS-CLEANUP-REVIEW.md` 通过。 |
| 不授权边界清晰 | CR111、context capsule、CP2 checkpoint 均列出禁止项。 |
| 后续候选未被隐式启动 | FU-CR110-002 保持 candidate；CR105 / CR089 / runtime 不启动。 |

## CP2 确认结果

| 字段 | 内容 |
|---|---|
| 结论 | approved |
| 时间 | 2026-06-22T15:23:02+08:00 |
| 接受范围 | no-code/no-runtime STATE summary staleness cleanup gate。 |
| 接受优先级 | 顶层 `active_change`、`last_action`、`next_action`、`cr_tracking.status`、`last_consistency_check` 优先；历史长表 audit-only。 |
| 继续禁止 | 源码修改、checker implementation change、历史长表批量重写、CR105、CR089 恢复、runtime、交易写、NAS、凭据、publish。 |

## CP8 关闭结果

| 字段 | 内容 |
|---|---|
| 结论 | approved / READY |
| 时间 | 2026-06-22T15:36:41+08:00 |
| 关闭范围 | STATE 顶层摘要陈旧处理策略、`cr_tracking` 摘要同步策略、历史长表 audit-only 边界。 |
| 后续候选 | CR111 不新增自身候选；`FU-CR110-002` 仍保留在 CR110 台账等待单独选择。 |
| 继续禁止 | 源码修改、checker implementation change、历史长表批量重写、CR105、CR089 恢复、runtime、交易写、NAS、凭据、publish。 |
