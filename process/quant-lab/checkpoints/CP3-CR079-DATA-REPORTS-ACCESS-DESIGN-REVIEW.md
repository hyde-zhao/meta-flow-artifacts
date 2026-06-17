---
checkpoint_id: "CP3-CR079-DATA-REPORTS-ACCESS-DESIGN-REVIEW"
checkpoint_name: "CR079 Data/reports Access Design Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-17T05:56:16+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-17T06:12:28+08:00"
auto_check_result: "process/checks/CP3-CR079-DATA-REPORTS-ACCESS-DESIGN-CONSISTENCY.md"
context_capsule: "process/context/CP3-CR079-DATA-REPORTS-ACCESS-DESIGN-CONTEXT.yaml"
auto_final_authorization: false
target:
  phase: "solution-design"
  story_id: ""
  artifacts:
    - "process/changes/CR-079-REAL-DATA-REPORTS-ACCESS-GATE-2026-06-17.md"
---

# CP3 CR079 Data/reports Access Design Review 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP3-CR079-DATA-REPORTS-ACCESS-DESIGN-CONSISTENCY.md` | PASS | 0 | metadata-only 只读清点方案可审；本轮不做内容读取、复制、恢复或删除。 |

## Decision Brief

推荐决策：采用 metadata-only 只读清点方案，先对 target 与 legacy root 的 `reports/`、`data/` 做存在性、数量、规模和 mtime 范围确认；不读取文件内容，不访问 NAS 归档，不 provider rebuild。

备选方案：直接读取文件内容；直接从 NAS 对比 / 恢复；直接 provider rebuild；取消 data/reports 真实访问。推荐方案优先，因为它以最小真实触碰建立后续决策事实，同时保护数据内容、凭据和外部状态。

影响维度：访问架构、证据脱敏、运行授权、恢复路径、CR077 旧根退役输入、CR078 远端治理边界、CR046 paused boundary。

风险与回退：若 CP3 reject，CR079 保持 CP2 pending / design rejected，不进入 CP5 readiness；若未来需要内容读取、恢复或 NAS compare，必须修订 CP3/CP5 或另起执行 CR。失败回退只记录 BLOCKED，不 cleanup。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR079-DATA-REPORTS-ACCESS-DESIGN-CONTEXT.yaml` |
| capsule 状态 | ready-for-human-gate |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| Architecture Gray Areas | 已收敛为 metadata-only vs content/copy/restore 的边界决策。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| CP3 Context Capsule | `process/context/CP3-CR079-DATA-REPORTS-ACCESS-DESIGN-CONTEXT.yaml` | scanned | 5 | 5 | 方案、路径、授权、脱敏、回退纳入 DQ。 |
| CR079 formal CR | `process/changes/CR-079-REAL-DATA-REPORTS-ACCESS-GATE-2026-06-17.md` | scanned | 5 | 5 | 执行方案和不授权边界纳入。 |
| 自动预检结果 | `process/checks/CP3-CR079-DATA-REPORTS-ACCESS-DESIGN-CONSISTENCY.md` | scanned | 4 | 4 | design consistency 已纳入。 |
| CR076 policy | `process/checkpoints/CP3-CR076-DATA-REPORTS-POLICY-REVIEW.md` | scanned | 2 | 2 | policy-first 约束和 real access 独立门禁纳入。 |
| discussion log / checkpoint | `process/discussions/*` / `process/checks/*DISCUSSION-CHECKPOINT.json` | n/a | 0 | 0 | 无新增架构灰区交互；本轮以 CR076 follow-up 和用户显式启动为准。 |
| 委托 Agent 交还摘要 | `process/handoffs/*RETURN-SUMMARY.md` | n/a | 0 | 0 | 本轮不调度功能 agent。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR079-01 | architecture | 是否采用 metadata-only 只读清点方案？ | 采用；只读目录元数据，不读内容。 | 内容读取；取消 CR079。 | 推荐方案最小权限；内容读取风险更高。 | readiness 只证明结构和规模。 | 需要内容验证时修订或另起 CR。 |
| DQ-CP3-CR079-02 | architecture | 清点路径是否限定为 target/legacy 四个目录？ | 限定为 target/legacy 的 `reports/`、`data/`。 | 同时访问 NAS；同时访问 provider/lake/catalog。 | 推荐方案范围清楚；备选外部/数据面更大。 | NAS / provider 状态仍未知。 | 本地不足时另起 NAS/provider CR。 |
| DQ-CP3-CR079-03 | security | 证据是否只写脱敏摘要且不写 memory？ | 只写聚合摘要、顶层风险标记；不写样本或完整深层清单到 memory。 | 保存完整清单；保存样本。 | 推荐方案符合 memory policy；备选有泄露风险。 | 后续排查细节较少。 | 经用户和安全门禁批准后才放宽。 |
| DQ-CP3-CR079-04 | runtime_authorization | approve 后允许的命令族是否只限本地只读元数据命令？ | 只允许存在性、find/stat/du 类只读元数据清点。 | 允许 cat/head/pandas/parquet read；允许 rsync/cp/rm。 | 推荐方案防止内容和破坏性操作；备选越权。 | 命令设计保守。 | 需要扩展命令时重提 CP3/CP5。 |
| DQ-CP3-CR079-05 | implementation | 失败 / 回退是否仅记录 BLOCKED，不自动 cleanup？ | 是，仅记录失败和下一步决策。 | 失败后删除/清理；失败后恢复旧根。 | 推荐方案不触碰数据；备选破坏性高。 | 可能保留未知状态。 | cleanup 必须另起 destructive CR。 |

### 不授权项

如果你回复 approve，表示接受本文件 5 项推荐方案；不表示授权以下操作：

- 不授权读取文件内容、打印样本、解析 parquet/csv/json、计算内容 hash、复制、恢复、比对内容、移动、重命名或删除。
- 不授权访问 NAS 归档内容、provider fetch、lake write、catalog publish 或 provider rebuild。
- 不授权读取 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权远端 Git 写入、QMT/MiniQMT runtime、CR046 recovery、旧根退役、optional groups、full tests 或 destructive cleanup。

### 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 自动通过条件：N/A
- 授权原文：N/A

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP3 自动预检 PASS | approved | `process/checks/CP3-CR079-DATA-REPORTS-ACCESS-DESIGN-CONSISTENCY.md` | 用户回复“好的，先完成CR079的迁移”，按 approve 处理。 |
| CP3 context ready | approved | `process/context/CP3-CR079-DATA-REPORTS-ACCESS-DESIGN-CONTEXT.yaml` | 用户回复“好的，先完成CR079的迁移”，按 approve 处理。 |
| 推荐方案已记录 | approved | CR079 formal CR §执行方案 | 用户接受 metadata-only 设计。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 是否采用 metadata-only 方案 | approved | DQ-CP3-CR079-01 | 用户接受推荐方案。 |
| 2 | 清点路径是否限定 | approved | DQ-CP3-CR079-02 | 用户接受推荐方案。 |
| 3 | 证据脱敏和 memory 边界是否接受 | approved | DQ-CP3-CR079-03 | 用户接受推荐方案。 |
| 4 | 命令族是否限定为只读元数据 | approved | DQ-CP3-CR079-04 | 用户接受推荐方案。 |
| 5 | 失败 / 回退是否不 cleanup | approved | DQ-CP3-CR079-05 | 用户接受推荐方案。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | approved | 当前对话：用户回复“好的，先完成CR079的迁移” | 解释为 CR079 CP2/CP3/CP5 approve；“迁移”不扩大为复制 / 恢复 / 删除授权。 |
| CP3 approve 后仍不执行，直到 CP5 也 approved | approved | 不授权项 | 三门同步 approved 后才进入 metadata-only 清点。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP3 auto precheck | `process/checks/CP3-CR079-DATA-REPORTS-ACCESS-DESIGN-CONSISTENCY.md` | approved | PASS。 |
| CP3 context | `process/context/CP3-CR079-DATA-REPORTS-ACCESS-DESIGN-CONTEXT.yaml` | approved | ready-for-human-gate。 |
| CP3 manual checkpoint | `process/checkpoints/CP3-CR079-DATA-REPORTS-ACCESS-DESIGN-REVIEW.md` | approved | 用户接受 DQ-CP3-CR079-01..05。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-17T06:12:28+08:00
- 修改意见：无；用户回复“好的，先完成CR079的迁移”，按 approve 处理。
- 风险接受项：接受 metadata-only 设计、target/legacy 四目录限定、脱敏摘要和不写 memory 边界；确认不授权内容读取、NAS/provider、远端 Git、runtime、CR046 或 cleanup。
