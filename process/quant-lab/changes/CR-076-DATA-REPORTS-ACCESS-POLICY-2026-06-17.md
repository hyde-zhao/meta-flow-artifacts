---
cr_id: "CR-076"
status: "closed-current-delivery"
title: "Data/reports Access Policy"
created_at: "2026-06-17T00:18:01+08:00"
created_by: "host-orchestrator"
parent_cr: "CR-073, CR-074, CR-075"
source_tracking: "USER-20260617-AFTER-CR075-START-CR076"
source_decision_id: "CR076-candidate"
approval_required: true
approval_result: "approved"
approved_by: "user"
approved_at: "2026-06-17T00:41:44+08:00"
closed_by: "user"
closed_at: "2026-06-17T05:36:08+08:00"
impact_level: "high"
rollback_to: "CR075 closed-current-delivery / CR076 candidate"
workflow_mode_after_change: "standard"
target_root: "/home/hyde/workspace/quant-lab"
legacy_root: "/home/hyde/workspace/local_backtest"
---

# CR-076 Data/reports Access Policy

## 背景

用户在确认 CR075 CP8 后要求继续处理 CR076。CR076 来自 CR073 / CR074 / CR075 后续事项台账，目标是为 `reports/` 与 `data/` 的后续处理建立访问策略、授权边界、证据要求和恢复路径选择。

本 CR 启动时不读取、列出、复制、恢复、比对或删除 `reports/`、`data/`，不读取 `.env` 或任何凭据，不执行 provider fetch、lake write、catalog publish、远端 Git 写入、QMT/MiniQMT runtime 或 CR046 recovery。

## 范围

| 项 | 决策 |
|---|---|
| In Scope | data/reports 访问策略、权限边界、候选恢复路径、后续执行门禁 |
| Out of Scope | 真实读取 / 复制 / 恢复 / 比对 / 删除 `reports/`、`data/` |
| Out of Scope | `.env`、token、密码、cookie、session、private key、账户或交易事实 |
| Out of Scope | provider fetch、lake write、catalog publish、MARKET_DATA_LAKE_ROOT replacement |
| Out of Scope | 远端 Git、QMT/MiniQMT runtime、CR046 recovery、optional groups、full tests、destructive cleanup |

## 冲突预检

| 对象 | 影响面 | 结论 | 说明 |
|---|---|---|---|
| CR046 | QMT/MiniQMT strategy framework | no-overlap | CR076 不恢复 CR046，不运行交易 runtime。 |
| CR075 | target env rebuild / minimal smoke | continuation | CR075 已完成并进入关闭；CR076 承接后续 data/reports policy。 |
| CR077 | old root retirement | related | CR076 可影响旧根是否仍需保留，但不执行旧根删除 / 移动。 |
| CR078 | remote Git governance | no-overlap | CR076 不做远端写入。 |

## 五维影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否新增访问策略需求 | data/reports governance | true | 新增 CR076 范围基线，不修改产品需求正文。 |
| 场景层 | 是否新增恢复 / 访问场景 | local retained / NAS archive / rebuild | true | 在 CR076 Decision Brief 暴露路径选择，不触碰数据。 |
| 计划层 | 是否改变执行顺序 | CP2/CP3/CP5 -> policy decision | true | 三门 approved 前不得做任何 data/reports 操作。 |
| 安全层 | 是否触碰敏感数据或凭据 | reports/data/.env/credentials | high | 当前仅策略；任何真实访问需后续明确授权。 |
| 交付层 | 是否需要新证据 | policy context/checkpoints | true | 生成 CR076 CP2/CP3/CP5 证据。 |

## 文档处理决策

| 文档 | 处理方式 | 旧基线保留方式 | 说明 |
|---|---|---|---|
| `process/STATE.md` | 原文档更新 | 保留 history | 记录 CR075 关闭和 CR076 pending gate。 |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留候选台账和正式 CR 索引 | CR076-candidate 转 active。 |
| `process/changes/CR-076-DATA-REPORTS-ACCESS-POLICY-2026-06-17.md` | 新增 | N/A | 本正式 CR。 |
| `docs/product/*` | 不变 | 原文档不改 | 本 CR 不改产品需求。 |
| `docs/design/*` | 不变 | 原文档不改 | 本 CR 不改架构设计。 |

## 替代 Story / LLD 门控说明

- 替代门禁：`CP5-CR076-DATA-REPORTS-POLICY-READINESS`
- 理由：CR076 是访问策略和授权边界 CR，不修改代码、接口、Schema、Story 或文件 owner；使用 CP5 readiness 替代 Story LLD 批次。
- 执行启动条件：
  - [ ] `process/checkpoints/CP2-CR076-DATA-REPORTS-BASELINE.md` 结论 `approved`
  - [ ] `process/checkpoints/CP3-CR076-DATA-REPORTS-POLICY-REVIEW.md` 结论 `approved`
  - [ ] `process/checkpoints/CP5-CR076-DATA-REPORTS-READINESS.md` 结论 `approved`
  - [ ] CR tracking consistency PASS
  - [ ] 不授权项仍未被执行

## 推荐策略

推荐采用 policy-first / no-data-touch 策略：

1. CR076 先确认 data/reports 访问政策、证据要求和后续路径。
2. CP2/CP3/CP5 approve 只确认策略，不授权真实访问。
3. 后续如需真实访问 local retained assets、NAS archive 或 provider rebuild，另起 runtime authorization / execution CR。

## 不授权项

| ID | 不授权内容 |
|---|---|
| NA-CR076-001 | 读取、列出、复制、恢复、比对或删除 `reports/`、`data/` |
| NA-CR076-002 | 读取、打印、记录、复制或迁移 `.env`、token、密码、cookie、session、private key、账户或交易事实 |
| NA-CR076-003 | provider fetch、lake write、catalog publish、MARKET_DATA_LAKE_ROOT replacement |
| NA-CR076-004 | 远端 Git 写入、branch/default branch governance、history rewrite、force push、tag、remote deletion 或 set-url |
| NA-CR076-005 | QMT / MiniQMT runtime、连接、账号查询、下单、撤单、simulation/live |
| NA-CR076-006 | 删除、重命名或移动旧根 `/home/hyde/workspace/local_backtest` |
| NA-CR076-007 | 自动恢复或推进 CR046 |
| NA-CR076-008 | optional dependency groups、full tests、删除 `.venv` 或 destructive cleanup |

## 处理结论

- 审批结论：approved（用户回复“同意”，2026-06-17T00:41:44+08:00）
- 执行结论：PASS（policy / ledger only；无真实 data/reports 访问）
- [ ] 自动批准（低风险）
- [x] CP2/CP3/CP5 人工审批已通过
- [x] CP8 人工确认已通过（READY_WITH_RISK）
- 关闭结论：closed-current-delivery（用户回复“同意”，2026-06-17T05:36:08+08:00）。关闭范围只覆盖 policy-first / no-data-touch 当前交付，不授权真实 data/reports、凭据、provider/lake/catalog、远端 Git、runtime、CR046 或 cleanup。

## 执行与验证结果

| 项 | 范围 | 结果 | 证据 |
|---|---|---|---|
| CP2/CP3/CP5 人工门禁 | policy-only / no-data-touch | approved | `process/checkpoints/CP2-CR076-DATA-REPORTS-BASELINE.md`; `process/checkpoints/CP3-CR076-DATA-REPORTS-POLICY-REVIEW.md`; `process/checkpoints/CP5-CR076-DATA-REPORTS-READINESS.md` |
| policy ledger execution | 将 CR076 policy-first / no-data-touch 结论固化到 process ledger | PASS | `process/checks/CR076-POLICY-LEDGER-EXECUTION-2026-06-17.md` |
| no-data-touch verification | 验证本轮证据只包含 process / release 文档，不包含真实 data/reports 访问 | PASS_WITH_RISK | `process/checks/CR076-NO-DATA-TOUCH-VERIFICATION-2026-06-17.md` |
| 未授权项 | `reports/`、`data/`、`.env`、provider/lake/catalog、remote Git、runtime、CR046、cleanup | 未执行 | `process/checks/CR076-NO-DATA-TOUCH-VERIFICATION-2026-06-17.md` |

## CP8 收尾门禁

| 项 | 结果 | 证据 |
|---|---|---|
| CP8 自动预检 | PASS_WITH_RISK | `process/checks/CP8-CR076-DELIVERY-READINESS.md` |
| CP8 人工审查稿 | approved | `process/checkpoints/CP8-CR076-DELIVERY-READINESS.md` |
| Release Context | READY_WITH_RISK / minimal | `process/release/RELEASE-CONTEXT-CR076.yaml` |
| Human Gate Launch | PASS | `process/checks/CP8-CR076-HUMAN-GATE-LAUNCH-MESSAGE.md` |

CP8 已由用户在当前对话回复“同意”批准。CR076 当前正式关闭为 `closed-current-delivery / READY_WITH_RISK`；后续真实 data/reports 访问、旧根退役、远端 Git 治理或 CR046 recovery 必须另起独立门禁。

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| CR | `CR-075` | target env rebuild / minimal smoke 来源。 |
| Candidate | `CR076-candidate` | 本 CR 来源。 |
| Target root | `/home/hyde/workspace/quant-lab` | authoritative root。 |
| Legacy root | `/home/hyde/workspace/local_backtest` | 保留；CR076 不删除、不移动、不重命名。 |
