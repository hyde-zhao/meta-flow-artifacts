---
cr_id: "CR-098"
status: "active-cp7-passed-with-risk-pending-cp8"
impact_level: "high"
workflow_mode_before: "production"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "命中 QMT runner、Windows gateway、HMAC client env、真实只读账户查询、redacted evidence 和未来交易写扩展边界；不得使用 fast-lane。"
rollback_to: "CR097 closed-current-delivery / READY"
approval_result: "cp7-pass-with-risk-pending-cp8"
created_at: "2026-06-19T11:55:26+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-19T11:55:26+08:00"
source: "user"
linked_issue: ""
parent_cr: "CR-097"
source_checkpoint: "process/checkpoints/CP8-CR097-DELIVERY-READINESS.md"
source_decision_id: "USER-20260619-START-QMT-RUNNER-READONLY-INTEGRATION-GATE"
follow_up_type: "runner-readonly-gateway-integration-gate"
risk_class: "trading-runtime-boundary"
owner: "host-orchestrator"
revisit_condition: "用户明确要求“启动 QMT runner readonly integration gate”。"
acceptance_criteria: "runner 在受控门禁内消费已验证的 Windows gateway 只读能力，覆盖 health、capabilities、query_positions_readonly，并仅输出 `.quant-lab` 脱敏 evidence；禁止 NAS、Windows `.env`、账户原文、交易写、simulation/live、provider/lake/publish。"
close_condition: "CP2/CP3/CP5 批准后完成 runner readonly integration 的实现、验证和 CP8；或因缺少授权、gateway、runner 环境、HMAC env 或安全边界转 blocked。"
cr_index_path: "process/changes/CR-INDEX.yaml"
---

# CR098 QMT Runner Readonly Gateway Integration Smoke

## 变更描述

用户要求启动 `QMT runner readonly integration gate`。本 CR 的目标是把 CR091 已交付的离线 strategy runner 与 CR097 已验证的 Windows gateway 只读链路衔接起来：runner 不直接 import XtQuant，不直接读取 Windows `.env`，只通过受控 gateway client 完成 `health`、`capabilities` 和 `query_positions_readonly`，并把结果保存为 `.quant-lab` 下的脱敏 evidence。

本 CR 已通过 CP2 scope、CP3 HLD 和 CP5 design readiness 人工确认，已完成 CP6 离线实现，并通过 CP7 离线验证为 `PASS_WITH_RISK`；它仍不授权读取 HMAC secret、启动 gateway、执行 runner runtime、查询账户或写任何交易动作。

## 上游事实

| 对象 | 当前事实 | 对 CR098 的含义 |
|---|---|---|
| CR091 | 已关闭为 `closed-current-delivery / READY_WITH_RISK` | 离线 runner 合同、fake readonly gateway、package/cache/evidence 模型可作为基础 |
| CR092 | 已关闭为 `closed-current-delivery / READY_WITH_RISK` | 提供真实 readonly runtime smoke 的设计、测试计划和分流边界 |
| CR097 | 已关闭为 `closed-current-delivery / READY` | Windows gateway 实测通过；`query_positions_readonly` 返回空持仓脱敏 evidence |
| CR089 | 仍为 blocked formal CR | 不恢复、不合并、不自动启动 |
| CR020 | 历史 gateway route 已关闭 / 归档 | 只参考当前源码合同，不恢复旧 route 状态 |

## 冲突预检

| 检查项 | 结论 | 证据 | 说明 |
|---|---|---|---|
| 当前 active formal CR | PASS | `meta-flow check cr-tracking --strict-warnings` | CR097 关闭后 active formal CR 为 none |
| blocked CR 重叠 | PASS_WITH_RISK | `CR-089` | CR089 涉及 QMT interface / package / NAS 语义，CR098 不恢复 CR089，也不使用 NAS |
| runner 基线 | PASS | `trading/strategy_runner/*` | CR091 离线 runner 存在 fake readonly gateway 和 evidence 合同 |
| gateway 基线 | PASS | CR097 evidence | Windows gateway health / capabilities / query_positions_readonly 已通过，但本 CR 仍需独立执行授权 |
| 安全边界 | PASS | 本 CR | CP2 approve 不授权 secret/env/account/raw positions/trading write |

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/changes/CR-098-QMT-RUNNER-READONLY-GATEWAY-INTEGRATION-SMOKE-2026-06-19.md` | 新增 | 保留 CR091 / CR092 / CR097 原交付事实 | 本文件 | pending-cp2 |
| `process/changes/CR-097-FOLLOW-UP-TRACKING-2026-06-19.md` | 新增 | 保留 CR097 CP8 后续分流 | 台账表 | pending-cp2 |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 保留 CR097 closed 和历史 CR 状态 | CR098 entry | pending-cp2 |
| `process/STATE.md` | 原文档更新 | 保留 CR097 closure history | 顶层 active_change / next_action | pending-cp2 |
| `process/context/CP2-CR098-RUNNER-READONLY-INTEGRATION-CONTEXT.yaml` | 新增 | 作为 CP2 最小上下文胶囊 | capsule metadata | pending-cp2 |
| `process/checkpoints/CP2-CR098-QMT-RUNNER-READONLY-INTEGRATION-SCOPE-REVIEW.md` | 新增 | CP2 人工门禁 | 人工审查结果 | pending |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否新增或重定义需求 | runner 消费真实 readonly gateway 的最小闭环 | true | 新增 CR098 gate，不改既有产品需求基线 |
| 场景层 | 是否改变测试矩阵覆盖范围 | runner health、gateway health、capabilities、query_positions_readonly、redacted evidence | true | CP2 确认后进入 CP3/CP5，形成 runner integration 场景和测试计划 |
| 计划层 | 是否改变 Phase / Story / Wave | CR091 runner -> CR097 gateway -> CR098 integration | true | 标准模式：CP2 scope -> CP3 HLD -> CP5 design readiness -> CP6/CP7/CP8 |
| 安全层 | 是否引入高风险动作或权限要求 | HMAC client env、Windows gateway、账户只读查询 | true | 逐 run 授权；默认不读取 secret、不运行 gateway、不查询账户 |
| 交付层 | 是否需要交付或回归 | runner integration code / docs / tests / runtime evidence | true | 先冻结设计和测试边界，CP5 approve 后才实现 |

## 目标范围

### In Scope

- runner 通过受控 gateway client 调用：
  - `health`
  - `capabilities`
  - `query_positions_readonly`
- runner 输出 `.quant-lab/evidence/qmt/cr098/redacted/` 下的脱敏 evidence。
- 复用 CR091 的 evidence redaction 和 forbidden counter 思路。
- 复用 CR097 的 Windows gateway host/port/run-ref 模式，但不复用 CR097 的一次性授权。
- 设计和实现前必须明确最小 client env 路由、权限和文件模式。

### Out of Scope / Not Authorized

- NAS package exchange / pull / publish。
- Windows `.env` 读取。
- HMAC secret 自动读取或复用 CR097 env。
- 账号、资金、持仓明细原文、委托、成交或原始日志输出。
- submit / cancel、buy / sell、simulation / live。
- provider fetch、lake write、catalog publish。
- CR089 auto-start 或 CR020 gateway route restore。

## 推荐方案

推荐采用 `runner-owned readonly gateway client adapter`：

- 在 runner 层新增真实 readonly gateway adapter，但保持默认 fake/offline。
- 真实 adapter 只能由显式 runtime gate 启用。
- adapter 只消费最小 client env，且每次运行都需要用户授权读取。
- runner evidence 只保存摘要：status、position_count bucket、digest、redaction_status、forbidden counters。

## 备选方案

| 方案 | 优点 | 缺点 | 切换条件 |
|---|---|---|---|
| A. runner-owned readonly gateway client adapter（推荐） | runner 成为真实只读能力的消费方；边界清楚；可测试 | 需要改 runner adapter 和 evidence 合同 | CP2/CP3/CP5 批准后实施 |
| B. 继续手工 gateway smoke，不接 runner | 风险低、无代码改动 | 不能证明 runner integration | 用户只想保留手工验证 |
| C. 合并 NAS package exchange | 可同时验证包交换 | 范围膨胀，触碰 NAS 权限 | 用户另起 NAS gate |
| D. 直接扩展到 order-write | 更接近交易闭环 | 风险最高，越过当前安全阶段 | 不建议；必须另起 order-write CR |

## 当前门控

| 门控 | 状态 | 说明 |
|---|---|---|
| CP2 scope review | approved | 用户已确认范围、不授权边界和推荐方案 |
| CP3 HLD | approved | 用户已确认 HLD 和不授权边界 |
| CP5 design readiness | approved | 用户已确认 LLD / TEST-PLAN / TASKS 和不授权边界 |
| CP6 implementation | passed | 离线 adapter / evidence / tests 已完成 |
| CP7 verification | PASS_WITH_RISK | 离线 / fixture 验证通过；真实 runtime、非空持仓和交易日路径未证明 |
| CP8 closure | pending-human-review | CP8 自动预检和人工审查稿已生成 |

## 处理结论

- 审批结论：`cp7-pass-with-risk-pending-cp8`
- [x] 已启动正式 CR（用户明确要求启动）
- [x] CP2 人工确认完成（高风险 runner / gateway integration）
- [x] CP3 HLD 人工确认完成（runner-owned readonly facade）
- [x] CP5 design readiness 人工确认完成
- [x] CP6 offline implementation 完成
- [x] CP7 offline verification 完成，结论 `PASS_WITH_RISK`
- [ ] 未授权运行（当前不读取 HMAC env、不启动 gateway、不执行 runner）

## CP2 审查结果

| 字段 | 内容 |
|---|---|
| 结论 | `approved` |
| 审查时间 | `2026-06-19T12:00:19+08:00` |
| 用户回复 | “同意” |
| 接受决策 | `DQ-CP2-CR098-01..05` |
| 下一门控 | CP3 HLD review |

## CP3 审查结果

| 字段 | 内容 |
|---|---|
| 结论 | `approved` |
| 审查时间 | `2026-06-19T12:07:32+08:00` |
| 用户回复 | “同意” |
| 接受决策 | `DQ-CP3-CR098-01..05` |
| 下一门控 | CP5 design readiness |

## CP5 审查结果

| 字段 | 内容 |
|---|---|
| 结论 | `approved` |
| 审查时间 | `2026-06-19T12:18:02+08:00` |
| 用户回复 | “同意” |
| 接受决策 | `DQ-CP5-CR098-01..05` |
| 下一门控 | CP6 offline implementation |

## CP6 实现结果

| 字段 | 内容 |
|---|---|
| 结论 | `PASS` |
| 检查文件 | `process/checks/CP6-CR098-RUNNER-READONLY-INTEGRATION-CODING-DONE.md` |
| 实现说明 | `process/stories/CR098-RUNNER-READONLY-INTEGRATION-IMPLEMENTATION.md` |
| 源码范围 | `trading/strategy_runner/readonly_gateway.py`、`trading/strategy_runner/evidence.py`、`trading/qmt_client.py`、`tests/test_cr098_runner_readonly_integration.py` |
| 下一门控 | CP7 offline verification；真实 runner runtime smoke 仍需逐 run 授权 |

## CP7 验证结果

| 字段 | 内容 |
|---|---|
| 结论 | `PASS_WITH_RISK` |
| 检查文件 | `process/checks/CP7-CR098-RUNNER-READONLY-INTEGRATION-VERIFICATION-DONE.md` |
| 验证报告 | `docs/features/cr098-runner-readonly-integration/VERIFICATION.md` |
| 测试报告 | `docs/features/cr098-runner-readonly-integration/TEST-REPORT.md` |
| 质量评审 | `docs/features/cr098-runner-readonly-integration/REVIEW.md` |
| 回修 | `docs/features/cr098-runner-readonly-integration/FIXES.md`，N/A |
| 下一门控 | CP8 delivery readiness / risk acceptance |

## CP8 发起结果

| 字段 | 内容 |
|---|---|
| 自动预检 | `PASS_WITH_RISK` |
| 自动预检文件 | `process/checks/CP8-CR098-DELIVERY-READINESS.md` |
| 人工审查稿 | `process/checkpoints/CP8-CR098-DELIVERY-READINESS.md` |
| Release Context | `process/release/RELEASE-CONTEXT-CR098.yaml` |
| 推荐结论 | `READY_WITH_RISK` |

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| Parent CR | `CR-097` | 已完成真实 gateway readonly smoke |
| Runner baseline | `CR-091` | 已完成离线 runner 合同 |
| Design baseline | `CR-092` | 已完成真实 readonly smoke design gate |
| Evidence | `/home/hyde/.quant-lab/evidence/qmt/cr097/redacted/cr097-query-positions-redacted-20260619.json` | CR097 空持仓脱敏成功路径 evidence |
| Source code | `trading/strategy_runner/readonly_gateway.py` | 当前 fake readonly wrapper |
| Tests | `tests/test_cr091_strategy_runner_contracts.py` | 当前离线 runner fixture tests |
