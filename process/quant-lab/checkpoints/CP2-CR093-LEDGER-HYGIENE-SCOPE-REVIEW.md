---
checkpoint_id: "CP2"
checkpoint_name: "CR093 Ledger Hygiene Scope Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-18T17:30:00+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-18T17:45:00+08:00"
auto_check_result: "process/checks/CP2-CR093-LEDGER-HYGIENE-SCOPE.md"
target:
  phase: "requirement-clarification"
  story_id: "CR093"
  artifacts:
    - "process/context/CP2-CR093-LEDGER-HYGIENE-CONTEXT.yaml"
    - "process/changes/CR-093-LEDGER-HYGIENE-CR019-CR025-TRACKING-CLEANUP-2026-06-18.md"
    - "process/changes/CR-091-FOLLOW-UP-TRACKING-2026-06-18.md"
    - "process/changes/CR-092-FOLLOW-UP-TRACKING-2026-06-18.md"
---

# CP2 CR093 Ledger Hygiene Scope Review 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR093-LEDGER-HYGIENE-SCOPE.md` | PASS | 0 | CR093 可进入 CP2 人工审查；当前 `cr-tracking` exit 1 是本 CR 的目标缺陷，不是启动阻断。 |

## Decision Brief

CR093 是从 CR091 / CR092 后续候选 `CR091-FU-04 Ledger hygiene` 启动的账本治理 CR。它的 CP2 只确认范围、风险和后续设计方向：是否允许继续产出 CP3 / CP5 设计材料，用于治理 CR019 follow-up tracking 和 CR025 historical active_change 对 `cr-tracking` 的阻断。

推荐结论：批准 CP2 范围基线，进入 CP3 / CP5 设计。CP2 批准不授权立即修改检查器或旧账本，也不授权任何 runtime、NAS、凭据、账户、交易写、simulation/live、provider/lake/publish。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR093-LEDGER-HYGIENE-CONTEXT.yaml` |
| capsule 状态 | `ready-for-human-review` |
| read_profile | `compact` |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 1 次；本轮读取 STATE 顶层状态、CR091/CR092 follow-up 行、CR-INDEX 和 checker 摘要用于 CP2 intake |
| 缺失 / waived 理由 | N/A |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 本轮 CP2 决策在本 checkpoint 中新增。 |
| 自动预检结果 | `process/checks/CP2-CR093-LEDGER-HYGIENE-SCOPE.md` | scanned | 3 | 3 | target defect / no-execution / no-runtime 纳入 DQ。 |
| CR093 正式 CR | `process/changes/CR-093-LEDGER-HYGIENE-CR019-CR025-TRACKING-CLEANUP-2026-06-18.md` | scanned | 5 | 5 | scope、implementation、security、risk、follow-up 均纳入 DQ。 |
| CR091 follow-up tracking | `process/changes/CR-091-FOLLOW-UP-TRACKING-2026-06-18.md` | scanned | 4 | 1 | 仅 CR091-FU-04 本轮启动；FU-02..03 保持候选。 |
| CR092 follow-up tracking | `process/changes/CR-092-FOLLOW-UP-TRACKING-2026-06-18.md` | scanned | 5 | 1 | 仅 CR091-FU-04 本轮启动；runtime / NAS / order-write 保持候选。 |
| 用户显式选择题 | 当前会话 | scanned | 1 | 1 | 用户选择启动 CR091-FU-04。 |
| discussion log / checkpoint | N/A | n/a | 0 | 0 | 本轮是 CP8 follow-up CR 启动，不新增 SGQ 异步讨论。 |

### 待人工决策清单

如果你回复 `approve`，表示你接受以下 5 项推荐方案；不表示授权后面的禁止操作。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR093-01 | scope | CR093 是否只治理 CR019 / CR025 相关历史账本与 `cr-tracking` 一致性？ | 是，仅覆盖静态账本、状态索引和检查器语义。 | 扩大到 runtime / NAS 台账；取消 CR093 保留旧账。 | 推荐方案聚焦目标缺陷且风险低；扩大范围会混入高风险授权；取消会继续阻断检查。 | 不处理交易业务或真实环境，只提升过程账本可靠性。 | 用户要求扩大范围时拆新 CR；用户 reject 时回到 follow-up backlog。 |
| DQ-CP2-CR093-02 | implementation | 后续修复路线采用什么策略？ | 采用“检查器语义修正 + 最小台账规范化”：解析当前权威状态、接受等价 closed 状态、排除历史审计文本误扫。 | 只改旧账本迎合旧检查器；只 suppress 警告；重写全量 checker。 | 推荐方案能修根因且控制改动面；只改账本会保留误扫缺陷；suppress 风险较高；全量重写成本过大。 | CP5 后可能修改 checker、测试 fixture、CR019 follow-up tracking 和状态索引。 | CP3/CP5 发现检查器结构不可控时降级为最小账本规范化。 |
| DQ-CP2-CR093-03 | security | CR093 是否继续禁止 runtime、NAS、凭据、账户、交易写和数据发布？ | 继续全部禁止。 | 允许读取 NAS 台账；允许执行 runtime smoke；允许读取账户证据。 | 推荐方案符合 ledger-only 范围；备选会引入外部状态和敏感边界。 | 无外部副作用，避免把账本治理误扩展为运行授权。 | 用户明确授权外部动作时必须新建独立 runtime / NAS gate。 |
| DQ-CP2-CR093-04 | risk_acceptance | 是否接受历史正文保留但 checker 不把历史审计文本当作当前缺陷？ | 接受；历史追溯保留，当前状态以权威字段 / 当前表格为准。 | 删除历史片段；继续全量文本扫描；把所有旧状态统一改写。 | 推荐方案兼顾追溯与检查准确性；删除历史会损失审计；全量扫描会继续误报。 | 需要在设计中明确“当前状态”和“历史审计文本”的边界。 | 若审计要求必须全文扫描，则改为分层扫描并标注 audit-only。 |
| DQ-CP2-CR093-05 | follow_up_tracking | CR092-FU-02 runtime、CR091-FU-02 NAS、CR091-FU-03 order-write 如何处理？ | 全部保持候选，不随 CR093 启动。 | 合并 runtime / NAS / order-write；取消全部候选。 | 推荐方案避免范围膨胀且保留后续路线；合并会扩大风险；取消会丢失 backlog。 | CR093 approve 只推进 ledger hygiene，不授权其他候选。 | 用户明确选择其他候选时另起独立 CR。 |

### 用户需决策事项

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 DQ-CP2-CR093-01..05，允许 CR093 进入 CP3 / CP5 设计 |
| 备选方案 | `修改: <具体修改点>` 后重出 CP2；或 `reject` 回退到 follow-up backlog |
| 影响维度 | 范围、实施路线、安全边界、历史追溯风险、后续候选分流 |
| 优劣分析 | 推荐方案聚焦 CR019 / CR025 历史旧账和 checker 语义缺陷，不引入外部运行授权；备选要么保留根因，要么扩大高风险范围 |
| 风险与回退 | CP2 通过仍不修复旧账；只允许进入设计。reject 或修改时回退到 CR093 active pending CP2 或 follow-up backlog |
| 用户需决策事项 | DQ-CP2-CR093-01 是否限定 ledger hygiene；DQ-CP2-CR093-02 是否采用推荐修复路线；DQ-CP2-CR093-03 是否继续禁止外部动作；DQ-CP2-CR093-04 是否接受历史保留但 checker 分层；DQ-CP2-CR093-05 是否保持其他候选不启动 |

## 不授权范围

本 CP2 即使通过，也不授权：

- 修改 `scripts/check_cr_tracking_consistency.py` 或旧账本正文。
- QMT / MiniQMT / XtQuant / gateway / runner runtime 启动、连接、安装或运行。
- NAS 访问、列取、读取、复制、拉取、写入、发布或删除。
- 读取 `.env`、凭据、账号、账户、资金、持仓、委托、成交或日志原文。
- submit / cancel、buy / sell。
- simulation / live。
- provider fetch、lake write、catalog publish。
- 自动启动 CR089、CR091-FU-02、CR091-FU-03 或恢复 CR020 用户删除的 gateway 路线。

## Entry Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| CR093 已创建 | 待审查 | `process/changes/CR-093-LEDGER-HYGIENE-CR019-CR025-TRACKING-CLEANUP-2026-06-18.md` |  |
| CP2 context capsule ready | 待审查 | `process/context/CP2-CR093-LEDGER-HYGIENE-CONTEXT.yaml` |  |
| 自动预检通过 | 待审查 | `process/checks/CP2-CR093-LEDGER-HYGIENE-SCOPE.md` |  |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | Decision Brief 覆盖推荐、备选、影响、风险和回退 | 待审查 | 本文件 Decision Brief |  |
| 2 | Decision Collection Coverage 已列出 | 待审查 | 本文件覆盖表 |  |
| 3 | 用户视角不授权项已列出 | 待审查 | 本文件“不授权范围” |  |
| 4 | CR019 / CR025 目标缺陷已明确 | 待审查 | 自动预检和 CR093 正式 CR |  |
| 5 | CP2 approve 仅允许进入设计 | 待审查 | DQ-CP2-CR093-02 |  |

## Exit Criteria

| 用户回复 | 处理 |
|---|---|
| `approve` | 接受 5 项推荐方案，CR093 进入 CP3 / CP5 设计。 |
| `修改: <具体修改点>` | 按修改点修订 CP2 材料并重新发起确认。 |
| `reject` | 回退 CR093 到 follow-up backlog，或按用户要求取消 / 拆分。 |

## Deliverables

- `process/context/CP2-CR093-LEDGER-HYGIENE-CONTEXT.yaml`
- `process/checks/CP2-CR093-LEDGER-HYGIENE-SCOPE.md`
- `process/checkpoints/CP2-CR093-LEDGER-HYGIENE-SCOPE-REVIEW.md`
- `process/checks/CP2-CR093-HUMAN-GATE-LAUNCH-MESSAGE.md`

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-18T17:45:00+08:00
- 用户回复：同意
- 接受的决策项：DQ-CP2-CR093-01、DQ-CP2-CR093-02、DQ-CP2-CR093-03、DQ-CP2-CR093-04、DQ-CP2-CR093-05
- 修改意见：无
- 风险接受项：接受历史正文保留，但后续 checker 设计需要区分当前状态与历史审计文本；接受 CP2 只授权进入 CP3 / CP5 设计，不授权立即修复实施。
- 不授权边界：本次 approve 不授权修改 `scripts/check_cr_tracking_consistency.py` 或旧账本正文；不授权 QMT / MiniQMT / XtQuant / gateway / runner runtime、NAS、`.env` / 凭据 / 账号 / 账户 / 资金 / 持仓 / 委托 / 成交 / 日志原文、submit / cancel、buy / sell、simulation / live、provider / lake / catalog / publish、CR089 自动启动、CR091-FU-02、CR091-FU-03 或 CR020 gateway 路线恢复。
