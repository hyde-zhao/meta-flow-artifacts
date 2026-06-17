---
cr_id: "CR-079"
status: "closed-current-delivery"
title: "Real Data/reports Access Gate"
created_at: "2026-06-17T05:56:16+08:00"
created_by: "host-orchestrator"
parent_cr: "CR-073, CR-074, CR-076"
source_tracking: "USER-20260617-START-REAL-DATA-REPORTS-ACCESS-CR"
source_checkpoint: "process/checkpoints/CP8-CR076-DELIVERY-READINESS.md"
source_decision_id: "CR076-real-access-candidate"
approval_required: true
approval_result: "approved"
approved_by: "user"
approved_at: "2026-06-17T06:12:28+08:00"
cp8_approval_result: "approved"
cp8_approved_by: "user"
cp8_approved_at: "2026-06-17T06:28:29+08:00"
release_decision: "READY_WITH_RISK"
closed_by: "user"
closed_at: "2026-06-17T06:28:29+08:00"
impact_level: "high"
rollback_to: "CR076 closed-current-delivery / CR076-real-access-candidate"
workflow_mode_after_change: "standard"
target_root: "/home/hyde/workspace/quant-lab"
legacy_root: "/home/hyde/workspace/local_backtest"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR-079 Real Data/reports Access Gate

## 背景

用户明确要求启动真实 `data/reports access CR`。CR076 已关闭为 policy-first / no-data-touch 交付，并把未来真实 `reports/`、`data/` 访问登记为独立 runtime authorization / execution CR。

本 CR 将 `CR076-real-access-candidate` 转为正式 CR079。启动阶段只创建门禁和审查稿；在 CP2 / CP3 / CP5 全部人工批准前，不读取、列出、复制、恢复、比对或删除 `reports/`、`data/`。

## 范围

| 项 | 决策 |
|---|---|
| In Scope | 对 `reports/` 与 `data/` 做最小只读清点和 readiness 验证的授权门禁。 |
| In Scope | 仅在 CP2 / CP3 / CP5 approve 后读取目录元数据：存在性、顶层项、文件 / 目录数量、总字节规模、mtime 范围和风险标记。 |
| In Scope | 目标根 `/home/hyde/workspace/quant-lab` 与 legacy 根 `/home/hyde/workspace/local_backtest` 的 `reports/`、`data/` 只读清点。 |
| Out of Scope | 读取文件内容、打印样本、解析 parquet/csv/json、计算内容 hash、复制、恢复、比对内容、移动、重命名或删除。 |
| Out of Scope | `.env`、token、密码、cookie、session、private key、账户、交易事实或任何凭据。 |
| Out of Scope | provider fetch、lake write、catalog publish、MARKET_DATA_LAKE_ROOT replacement、NAS promote 或 provider rebuild。 |
| Out of Scope | 远端 Git 写入、branch/default branch governance、QMT/MiniQMT runtime、CR046 recovery、old root retirement、optional groups、full tests、destructive cleanup。 |

## 冲突预检

| 对象 | 影响面 | 结论 | 说明 |
|---|---|---|---|
| CR046 | QMT/MiniQMT strategy framework | no-overlap | CR079 不恢复 CR046，不读取凭据，不启动交易 runtime。 |
| CR073 | target root convergence | continuation | CR079 承接 target/legacy root 收敛后的 data/reports 后续项。 |
| CR074 | logical root cutover | continuation | CR079 尊重 authoritative root 为 `/home/hyde/workspace/quant-lab`，legacy root 只读清点用于 retained assets 事实确认。 |
| CR076 | data/reports access policy | source | CR079 是 CR076 后续 real access candidate 的正式化，不重新打开 CR076。 |
| CR077-candidate | old root retirement | related-no-overlap | CR079 不删除、移动或重命名旧根；清点结果可作为 CR077 未来输入。 |
| CR078-candidate | remote Git governance | no-overlap | CR079 不做远端 Git 写入或分支治理。 |

## 五维影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否新增真实访问需求 | data/reports real access | true | 新增 CR079 runtime authorization / execution gate。 |
| 场景层 | 是否改变验证范围 | target/legacy `reports/`、`data/` readiness | true | 新增只读清点场景；不做内容恢复或复制。 |
| 计划层 | 是否改变执行顺序 | CP2/CP3/CP5 -> CP6 inventory -> CP7 verification -> CP8 closure | true | 三门 approve 前阻断任何访问；approve 后只执行最小只读清点。 |
| 安全层 | 是否引入高风险权限 | 本地保留资产、敏感文件名、潜在数据样本 | high | 只记录脱敏摘要；不读取内容、不写 memory、不触碰凭据。 |
| 交付层 | 是否需要新证据 | CR079 checks / verification / release context | true | 生成只读清点证据、验证证据和 CP8 终验材料。 |

## 文档处理决策

| 文档 | 处理方式 | 旧基线保留方式 | 说明 |
|---|---|---|---|
| `process/STATE.md` | 原文档更新 | 保留 history | 记录 CR079 active gate、pending human decisions 和不授权边界。 |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留 CR076 candidate 追溯 | 将 `CR076-real-access-candidate` 指向 CR079。 |
| `process/changes/CR-079-REAL-DATA-REPORTS-ACCESS-GATE-2026-06-17.md` | 新增 | N/A | 本正式 CR。 |
| `docs/product/*` | 不变 | 原文档不改 | 本 CR 不改产品需求。 |
| `docs/design/*` | 不变 | 原文档不改 | 本 CR 不改系统架构。 |
| `docs/release/*-CR076.md` | 不变 | CR076 已关闭 | CR079 不重写 CR076 release 口径。 |

## 执行方案

推荐采用最小只读清点方案：

1. CP2 / CP3 / CP5 approve 前，只允许生成门禁文件、CR 索引和 human-gate 消息。
2. approve 后执行前置校验：CR tracking consistency、三份 human-gate 校验、target/legacy 状态一致性检查。
3. 仅对下列路径做目录元数据清点：`/home/hyde/workspace/quant-lab/reports`、`/home/hyde/workspace/quant-lab/data`、`/home/hyde/workspace/local_backtest/reports`、`/home/hyde/workspace/local_backtest/data`。
4. 证据只记录存在性、顶层目录 / 文件数量、总字节规模、mtime 范围、是否出现疑似凭据 / `.env` 风险标记；默认不打印深层完整清单。
5. 清点结果写入 `process/checks/CR079-DATA-REPORTS-ACCESS-INVENTORY-2026-06-17.md`，验证结果写入 `process/checks/CR079-DATA-REPORTS-ACCESS-VERIFICATION-2026-06-17.md`。

## 执行与验证结果

| 阶段 | 结果 | 证据 | 说明 |
|---|---|---|---|
| CP6 metadata-only inventory | PASS_WITH_RISK | `process/checks/CR079-DATA-REPORTS-ACCESS-INVENTORY-2026-06-17.md` | target root 缺少 `reports/` 和 `data/`；legacy root 存在 retained assets。 |
| CP7 verification | PASS_WITH_RISK | `process/checks/CR079-DATA-REPORTS-ACCESS-VERIFICATION-2026-06-17.md` | 禁止项计数为 0；真实迁移 / 恢复未完成且不属于本 CR 批准范围。 |
| CP8 release readiness | approved | `process/checkpoints/CP8-CR079-DELIVERY-READINESS.md` | 用户已接受 READY_WITH_RISK 并关闭当前交付。 |

## Inventory 摘要

| 路径 | 存在 | 聚合结论 |
|---|---|---|
| `/home/hyde/workspace/quant-lab/reports` | false | target 缺失。 |
| `/home/hyde/workspace/quant-lab/data` | false | target 缺失。 |
| `/home/hyde/workspace/local_backtest/reports` | true | legacy retained assets 存在，约 4.53GB。 |
| `/home/hyde/workspace/local_backtest/data` | true | legacy retained assets 存在，约 61.7MB。 |

## 不授权项

| ID | 不授权内容 |
|---|---|
| NA-CR079-001 | CP2 / CP3 / CP5 approve 前，不读取、列出、复制、恢复、比对或删除 `reports/`、`data/`。 |
| NA-CR079-002 | 不读取文件内容、不打印样本、不解析 parquet/csv/json、不计算内容 hash。 |
| NA-CR079-003 | 不读取、打印、记录、复制或迁移 `.env`、token、密码、cookie、session、private key、账户或交易事实。 |
| NA-CR079-004 | 不执行 provider fetch、lake write、catalog publish、MARKET_DATA_LAKE_ROOT replacement、NAS promote 或 provider rebuild。 |
| NA-CR079-005 | 不执行远端 Git 写入、branch/default branch governance、history rewrite、force push、tag、remote deletion 或 set-url。 |
| NA-CR079-006 | 不启动 QMT / MiniQMT runtime、连接、账号查询、下单、撤单、simulation/live。 |
| NA-CR079-007 | 不删除、重命名或移动旧根 `/home/hyde/workspace/local_backtest`。 |
| NA-CR079-008 | 不恢复或推进 CR046。 |
| NA-CR079-009 | 不执行 optional dependency groups、full tests、删除 `.venv` 或 destructive cleanup。 |

## 替代 Story / LLD 门控说明

- 替代门禁：`CP5-CR079-DATA-REPORTS-ACCESS-READINESS`
- 理由：CR079 是 runtime authorization / execution gate，不修改代码、接口、Schema、Story 或文件 owner；使用 CP5 readiness 替代 Story LLD 批次。
- 执行启动条件：
  - [x] `process/checkpoints/CP2-CR079-DATA-REPORTS-ACCESS-BASELINE.md` 结论 `approved`
  - [x] `process/checkpoints/CP3-CR079-DATA-REPORTS-ACCESS-DESIGN-REVIEW.md` 结论 `approved`
  - [x] `process/checkpoints/CP5-CR079-DATA-REPORTS-ACCESS-READINESS.md` 结论 `approved`
  - [x] CR tracking consistency PASS
  - [x] human-gate 校验 PASS
  - [x] 不授权项仍未被执行

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 创建 CR079 并发起三门审查 | 用户请求 / CR076 follow-up | 本 CR、CP2/CP3/CP5 门禁 | human-gate 校验 PASS | 等待用户确认 |
| 2 | `host-orchestrator` | 执行只读清点 | 三门 approved、preflight PASS | inventory check | CP6 | 进入验证 |
| 3 | `host-orchestrator` | 验证清点证据和不授权边界 | inventory check | verification check | CP7 | 准备 CP8 |
| 4 | `host-orchestrator` | 收敛终验 | CR079 证据 | CP8 自动预检和人工审查稿 | 用户确认 | 关闭或回退 |

## 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 适用检查点：N/A
- 授权原文：N/A

## 后续事项台账

- 是否存在后续事项：true
- 本 CR 当前不预创建后续正式 CR。

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 正式 CR 路径 | 当前门控 | 阻塞原因 | 下一步 |
|---|---|---|---|---:|---|---|---|---|
| CR079-restore-candidate | data/reports restore or copy gate | candidate | CR | 1 |  | not-started | 需要先完成只读清点并明确恢复路径 | 清点后由用户决定是否启动。 |
| CR079-nas-compare-candidate | NAS archive compare gate | candidate | CR | 2 |  | not-started | 本轮不访问 NAS 归档内容 | 如本地 retained assets 不足，再评审。 |
| CR079-provider-rebuild-candidate | provider rebuild gate | candidate | CR | 3 |  | not-started | 本轮不执行 provider fetch/lake/catalog | 如本地和 NAS 不足，再评审。 |

## 处理结论

- 审批结论：approved（用户回复“好的，先完成CR079的迁移”，按 CP2/CP3/CP5 approve 处理；“迁移”不扩大为复制 / 恢复 / 删除授权）
- 当前执行结论：CP6 / CP7 为 `PASS_WITH_RISK`；只完成 metadata-only 清点和验证，不代表 `data/reports` 已真实迁移。
- 当前门禁：CP8 人工终验已通过，当前 metadata-only inventory 交付关闭为 `closed-current-delivery / READY_WITH_RISK`。
- [ ] 自动批准（低风险）
- [x] CP2/CP3/CP5 人工审批已通过（高风险，仅授权 metadata-only 只读清点）
- [x] CP6 / CP7 metadata-only 清点与验证已完成
- [x] CP8 人工终验

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| CR | `CR-076` | data/reports access policy 来源。 |
| Candidate | `CR076-real-access-candidate` | 本 CR 来源候选。 |
| Target root | `/home/hyde/workspace/quant-lab` | authoritative root。 |
| Legacy root | `/home/hyde/workspace/local_backtest` | retained assets 可能来源；CR079 不删除、不移动、不重命名。 |
