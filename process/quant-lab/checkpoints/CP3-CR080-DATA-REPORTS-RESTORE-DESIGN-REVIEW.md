---
checkpoint_id: "CP3-CR080-DATA-REPORTS-RESTORE-DESIGN-REVIEW"
checkpoint_name: "CR080 Data/reports Restore Design Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-17T06:42:48+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-17T06:57:22+08:00"
auto_check_result: "process/checks/CP3-CR080-DATA-REPORTS-RESTORE-DESIGN-CONSISTENCY.md"
context_capsule: "process/context/CP3-CR080-DATA-REPORTS-RESTORE-DESIGN-CONTEXT.yaml"
auto_final_authorization: false
target:
  phase: "solution-design"
  story_id: ""
  artifacts:
    - "process/changes/CR-080-DATA-REPORTS-RESTORE-COPY-GATE-2026-06-17.md"
---

# CP3 CR080 Data/reports Restore Design Review 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP3-CR080-DATA-REPORTS-RESTORE-DESIGN-CONSISTENCY.md` | PASS | 0 | 推荐设计为 legacy retained assets -> target root 的保守本地 copy-first；不读内容、不删、不访问 NAS/provider。 |

## Decision Brief

推荐决策：批准 CR080 CP3 设计，采用本地 copy-first restore/copy。设计约束是 target 冲突 fail-closed、credential-like path denylist、证据只写元数据摘要、禁止 `--delete` / `--checksum` / 内容读取 / 远端同步。

备选方案：先做 NAS compare；先做 provider rebuild；扩大为内容校验；暂缓 CR080。推荐方案优先，因为它最少依赖外部系统，且能先恢复 target root 的本地可用性。

影响维度：命令白名单、冲突预检、目标根写入、敏感路径过滤、验证证据粒度、失败/回退路径。

风险与回退：若 target 已有非空 `reports/` / `data/` 或来源异常，preflight BLOCKED；若复制中断，不自动删除或 cleanup，记录 partial 状态并等待人工决策。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR080-DATA-REPORTS-RESTORE-DESIGN-CONTEXT.yaml` |
| capsule 状态 | ready-for-human-gate |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| architecture discussion | N/A：本 CR 是运行授权 / 执行门禁，不新增系统架构。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 5 | 0 | CP2 DQ 已在 CP2 表中覆盖；CP3 只纳入设计问题。 |
| CR080 formal CR | `process/changes/CR-080-DATA-REPORTS-RESTORE-COPY-GATE-2026-06-17.md` | scanned | 10 | 5 | 命令族、冲突、证据、失败路径和边界纳入。 |
| CR079 inventory | `process/checks/CR079-DATA-REPORTS-ACCESS-INVENTORY-2026-06-17.md` | scanned | 4 | 2 | target 缺失和 legacy assets 存在是设计输入。 |
| 自动预检结果 | `process/checks/CP3-CR080-DATA-REPORTS-RESTORE-DESIGN-CONSISTENCY.md` | scanned | 6 | 5 | 所有设计风险均进入 DQ 或 N/A。 |
| discussion log / checkpoint | `process/discussions/*` / `process/checks/*DISCUSSION-CHECKPOINT.json` | n/a | 0 | 0 | 无新增产品/架构灰区讨论；本门使用 CR follow-up 来源。 |
| 委托 Agent 交还摘要 | `process/handoffs/*RETURN-SUMMARY.md` | n/a | 0 | 0 | 本轮不调度功能 agent。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR080-01 | architecture | 是否采用本地 legacy retained assets copy-first 设计？ | 采用本地 copy-first，先不访问 NAS/provider。 | 先做 NAS compare；先做 provider rebuild。 | 推荐方案最快补齐本地 target root；备选外部依赖和权限更多。 | 若 legacy assets 不足，仍需后续候选。 | preflight 来源不足时转 NAS/provider 候选。 |
| DQ-CP3-CR080-02 | implementation | 允许命令族是否限定为 `test/mkdir/find/stat/du/rsync` 且禁止 `--delete` / `--checksum`？ | 限定上述命令族；`rsync` 仅本地、无 delete、无 checksum、无远端。 | 允许 `cp -a`；允许 checksum；允许 delete 同步。 | 推荐方案可审计且降低误删/内容读取风险；备选更快但风险高。 | 可能无法证明内容一致性。 | 需要 checksum/content validation 时另起内容校验 CR。 |
| DQ-CP3-CR080-03 | security | credential-like path 如何处理？ | denylist 排除 `.env*`、token/secret/password/private/credential/key 等命名，证据只写脱敏计数。 | 全量复制；复制后再清理。 | 推荐方案先避免敏感复制；备选可能泄露或引入 cleanup 风险。 | 可能少复制少量敏感命名文件。 | 用户明确确认敏感例外时另起安全门禁。 |
| DQ-CP3-CR080-04 | risk_acceptance | target 非空或冲突时是否 fail-closed？ | 是，非空/冲突即 BLOCKED，不覆盖、不合并、不删除。 | 自动合并；自动覆盖；自动备份后覆盖。 | 推荐方案保守；备选可能破坏 target 状态。 | 若 target 已有部分内容，本 CR 会停下等待决策。 | 用户明确选择合并/覆盖策略后重提。 |
| DQ-CP3-CR080-05 | implementation | 失败/回退是否禁止自动 cleanup？ | 是，copy 失败只记录 partial 状态，不自动删除。 | 自动删除新建目标；自动移动到隔离目录。 | 推荐方案避免破坏性操作；备选能清理但需要更高授权。 | 失败时可能留下 partial 目录。 | cleanup 必须另起或重新授权。 |

### 不授权项

如果你回复 approve，表示接受本文件 5 项推荐方案；不表示授权以下操作：

- 不授权内容读取、样本打印、文件解析、内容 hash、checksum 校验或内容比对。
- 不授权 `rsync --delete`、远端 rsync、覆盖已有非空 target、自动 merge、自动 cleanup。
- 不授权复制 credential-like path；命中项只能脱敏计数。
- 不授权 NAS/provider/lake/catalog、remote Git、runtime、CR046、old root retirement 或 destructive cleanup。

### 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 自动通过条件：N/A
- 授权原文：N/A

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP3 自动预检 PASS | approved | `process/checks/CP3-CR080-DATA-REPORTS-RESTORE-DESIGN-CONSISTENCY.md` | 用户回复“同意”，接受推荐方案。 |
| CP3 context ready | approved | `process/context/CP3-CR080-DATA-REPORTS-RESTORE-DESIGN-CONTEXT.yaml` | 用户回复“同意”，接受推荐方案。 |
| CR080 设计边界可读 | approved | `process/changes/CR-080-DATA-REPORTS-RESTORE-COPY-GATE-2026-06-17.md` | 用户回复“同意”，接受推荐方案。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 是否采用本地 copy-first | approved | DQ-CP3-CR080-01 | 用户接受推荐方案。 |
| 2 | 命令族是否足够收敛 | approved | DQ-CP3-CR080-02 | 用户接受推荐方案。 |
| 3 | credential-like path 是否排除 | approved | DQ-CP3-CR080-03 | 用户接受推荐方案。 |
| 4 | target 冲突是否 fail-closed | approved | DQ-CP3-CR080-04 | 用户接受推荐方案。 |
| 5 | 失败是否不自动 cleanup | approved | DQ-CP3-CR080-05 | 用户接受推荐方案。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | approved | 当前对话：用户回复“同意” | 按 CR080 CP3 approve 处理；仍需 CP5 和 preflight。 |
| CP3 approve 后不单独执行 | approved | 不授权项 | 执行需 CP2/CP3/CP5 全部 approved。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP3 auto precheck | `process/checks/CP3-CR080-DATA-REPORTS-RESTORE-DESIGN-CONSISTENCY.md` | approved | PASS。 |
| CP3 context | `process/context/CP3-CR080-DATA-REPORTS-RESTORE-DESIGN-CONTEXT.yaml` | approved | ready-for-human-gate。 |
| CP3 manual checkpoint | `process/checkpoints/CP3-CR080-DATA-REPORTS-RESTORE-DESIGN-REVIEW.md` | approved | 用户接受 DQ-CP3-CR080-01..05。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-17T06:57:22+08:00
- 修改意见：无。用户回复“同意”，接受 DQ-CP3-CR080-01..05 推荐方案。
- 风险接受项：接受本地 copy-first 设计、命令族限制、credential-like path 排除、target 冲突 fail-closed 和失败不自动 cleanup。
