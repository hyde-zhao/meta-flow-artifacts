---
checkpoint_id: "CP2-CR080-DATA-REPORTS-RESTORE-BASELINE"
checkpoint_name: "CR080 Data/reports Restore Baseline"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-17T06:42:48+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-17T06:57:22+08:00"
auto_check_result: "process/checks/CP2-CR080-DATA-REPORTS-RESTORE-BASELINE.md"
context_capsule: "process/context/CP2-CR080-DATA-REPORTS-RESTORE-CONTEXT.yaml"
auto_final_authorization: false
target:
  phase: "requirement-clarification"
  story_id: ""
  artifacts:
    - "process/changes/CR-080-DATA-REPORTS-RESTORE-COPY-GATE-2026-06-17.md"
---

# CP2 CR080 Data/reports Restore Baseline 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR080-DATA-REPORTS-RESTORE-BASELINE.md` | PASS | 0 | CR080 formal CR 已创建；本门只确认范围基线，不执行 restore/copy。 |

## Decision Brief

推荐决策：批准 CR080 CP2 范围基线，将 `CR079-restore-candidate` 正式转为高风险 standard CR080；CP2 只确认范围、来源、目标和不授权项，不单独授权执行。实际 restore/copy 仍需 CP2 / CP3 / CP5 全部 approved 且 post-approval preflight PASS。

备选方案：保持 `CR079-restore-candidate` 为 candidate；取消恢复/复制路线；改为先启动 NAS compare；改为 provider rebuild。推荐方案优先，因为 CR079 已证明 target root 缺 `reports/` / `data/`，且 legacy retained assets 存在，本地 copy-first 是最小外部依赖路径。

影响维度：CR tracking、STATE、target authoritative root、legacy retained assets、content read boundary、credential boundary、CR077 old root retirement、CR078 remote Git governance、NAS/provider 后续候选。

风险与回退：若用户 reject，CR080 回退为 `CR079-restore-candidate`；若用户要求 NAS/provider/内容读取/删除，必须修订或另起 CR。三门批准前只回滚 process ledger，不访问或复制数据。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR080-DATA-REPORTS-RESTORE-CONTEXT.yaml` |
| capsule 状态 | ready-for-human-gate |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| SGQ / discussion | N/A：本 CR 来源为 CR079 CP8 follow-up candidate，不是新增产品场景发现。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | CR080 创建前无本 gate DQ；本文件创建后回写 pending。 |
| CR080 formal CR | `process/changes/CR-080-DATA-REPORTS-RESTORE-COPY-GATE-2026-06-17.md` | scanned | 8 | 5 | 启动、来源目标、范围、边界和不授权项纳入 DQ。 |
| CR079 CP8 | `process/checkpoints/CP8-CR079-DELIVERY-READINESS.md` | scanned | 5 | 2 | `CR079-restore-candidate` 来源、target 缺失和 legacy assets 纳入。 |
| CR079 inventory | `process/checks/CR079-DATA-REPORTS-ACCESS-INVENTORY-2026-06-17.md` | scanned | 4 | 2 | target 缺失 / legacy 存在作为范围事实；不重新读取 data/reports。 |
| CR-INDEX follow-up | `process/changes/CR-INDEX.yaml` | scanned | 5 | 3 | restore、NAS compare、provider rebuild、CR077、CR078 分流。 |
| discussion log / checkpoint | `process/discussions/*` / `process/checks/*DISCUSSION-CHECKPOINT.json` | n/a | 0 | 0 | 本轮不是新增产品需求讨论；范围来自 CR follow-up。 |
| 委托 Agent 交还摘要 | `process/handoffs/*RETURN-SUMMARY.md` | n/a | 0 | 0 | 本轮不调度功能 agent。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR080-01 | scope | 是否正式启动 CR080？ | 启动 CR080，将 `CR079-restore-candidate` 转 active，并进入 CP3/CP5 门禁。 | 保持 candidate；取消恢复/复制路线。 | 推荐方案响应用户请求并补齐 target root 可用性缺口；保持 candidate 不解决缺失。 | 新增正式高风险 CR 和待审门禁。 | reject 时回退为 candidate 或取消。 |
| DQ-CP2-CR080-02 | scope | 恢复/复制来源和目标是否限定为 legacy retained assets -> target authoritative root？ | 来源限定为 `/home/hyde/workspace/local_backtest/reports|data`，目标限定为 `/home/hyde/workspace/quant-lab/reports|data`。 | 先做 NAS compare；直接 provider rebuild。 | 推荐方案外部依赖最少；NAS/provider 路径需要额外授权和风险。 | 若 legacy retained assets 不足，可能仍需后续 NAS/provider CR。 | preflight 发现来源不足时转 NAS compare 或 provider rebuild。 |
| DQ-CP2-CR080-03 | security | CP2 是否继续禁止内容读取、样本、解析、hash 和凭据？ | 继续禁止；只允许后续经三门批准的元数据和受控 copy。 | 同时授权内容验证；同时授权凭据或 `.env`。 | 推荐方案最小权限；备选可能泄露敏感数据。 | 内容质量暂不证明。 | 需要内容验证时另起内容校验 CR。 |
| DQ-CP2-CR080-04 | runtime_authorization | CP2 approve 是否单独授权执行 restore/copy？ | 不授权；必须 CP2/CP3/CP5 全部 approved 且 preflight PASS。 | CP2 后立即执行；一次性授权所有后续访问。 | 推荐方案符合关键门控；备选边界过宽。 | 避免 CP2 被误读成运行授权。 | 三门批准后才进入 post-approval preflight。 |
| DQ-CP2-CR080-05 | follow_up_tracking | NAS compare、provider rebuild、CR077、CR078 是否继续保持独立候选？ | 保持独立候选，不自动启动。 | 同时启动 NAS compare；同时启动远端 Git governance 或旧根退役。 | 推荐方案避免并行扩大影响面；备选增加外部/破坏性风险。 | 后续仍需用户逐项授权。 | 用户明确启动对应候选时切换。 |

### 不授权项

如果你回复 approve，表示接受本文件 5 项推荐方案；不表示授权以下操作：

- 不授权 CP2 单独触发 `reports/` / `data/` restore/copy；必须 CP2 / CP3 / CP5 全部 approve 且 preflight PASS。
- 不授权读取文件内容、打印样本、解析 parquet/csv/json 或计算内容 hash。
- 不授权读取、打印、记录、复制或迁移 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权 NAS compare/promote、provider fetch、lake write、catalog publish、MARKET_DATA_LAKE_ROOT replacement 或 provider rebuild。
- 不授权远端 Git 写入、QMT/MiniQMT runtime、CR046 recovery、旧根删除/移动、optional groups、full tests、`--delete` 同步或 destructive cleanup。

### 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 自动通过条件：N/A
- 授权原文：N/A

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP2 自动预检 PASS | approved | `process/checks/CP2-CR080-DATA-REPORTS-RESTORE-BASELINE.md` | 用户回复“同意”，接受推荐方案。 |
| CR080 formal CR 存在 | approved | `process/changes/CR-080-DATA-REPORTS-RESTORE-COPY-GATE-2026-06-17.md` | 用户回复“同意”，接受推荐方案。 |
| context ready | approved | `process/context/CP2-CR080-DATA-REPORTS-RESTORE-CONTEXT.yaml` | 用户回复“同意”，接受推荐方案。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 是否正式启动 CR080 | approved | DQ-CP2-CR080-01 | 用户接受推荐方案。 |
| 2 | 来源和目标是否限定 | approved | DQ-CP2-CR080-02 | 用户接受推荐方案。 |
| 3 | 内容读取和凭据是否继续禁止 | approved | DQ-CP2-CR080-03 | 用户接受推荐方案。 |
| 4 | CP2 是否不单独授权执行 | approved | DQ-CP2-CR080-04 | 用户接受推荐方案。 |
| 5 | 后续候选是否保持独立 | approved | DQ-CP2-CR080-05 | 用户接受推荐方案。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | approved | 当前对话：用户回复“同意” | 按 CR080 CP2 approve 处理；需 CP3/CP5 同步 approved 和 preflight PASS。 |
| CP2 approve 后仍不执行真实恢复 | approved | 不授权项 | CP2 单独不触发 restore/copy；执行需 CP3/CP5 同步 approved 和 preflight PASS。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP2 auto precheck | `process/checks/CP2-CR080-DATA-REPORTS-RESTORE-BASELINE.md` | approved | PASS。 |
| CP2 context | `process/context/CP2-CR080-DATA-REPORTS-RESTORE-CONTEXT.yaml` | approved | ready-for-human-gate。 |
| CP2 manual checkpoint | `process/checkpoints/CP2-CR080-DATA-REPORTS-RESTORE-BASELINE.md` | approved | 用户接受 DQ-CP2-CR080-01..05。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-17T06:57:22+08:00
- 修改意见：无。用户回复“同意”，接受 DQ-CP2-CR080-01..05 推荐方案。
- 风险接受项：接受 CP2 范围基线；确认 CP2 单独不授权 restore/copy，执行仍需 CP3/CP5 同步 approved 和 post-approval preflight PASS。
