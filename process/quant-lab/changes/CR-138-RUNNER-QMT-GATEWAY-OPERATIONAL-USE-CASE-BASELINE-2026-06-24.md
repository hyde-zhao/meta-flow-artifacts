---
cr_id: "CR-138"
title: "Runner & QMT Gateway Operational Use-Case Baseline"
cr_type: "product-scope"
cr_kind: "requirement-change"
lifecycle_status: "closed"
readiness_status: "ready_with_risk"
gate_status: "closed"
gate_profile: "standard"
status: "closed-current-delivery"
impact_level: "high"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "命中 Runner / QMT gateway 用户场景基线、运行治理、安全边界、后续架构拆分和真实交易误授权风险；不得使用 fast-lane。"
rollback_to: "CR137 closed context reset handoff / process/USE-CASES.md v1.15 confirmed baseline"
approval_result: "cp2-approved"
cp3_approval_result: "approved"
cp3_approved_by: "user"
cp3_approved_at: "2026-06-24T15:30:00+08:00"
created_at: "2026-06-24T12:12:00+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-24T13:20:00+08:00"
source: "user-request"
linked_issue: ""
parent_cr: "CR-137"
source_checkpoint: "process/context/CR137-CLOSURE-TO-RUNNER-QMT-USE-CASE-CONTEXT-RESET-HANDOFF-2026-06-24.md"
source_decision_id: "USER-20260624-START-CR138"
follow_up_type: "runner-qmt-operational-use-case-baseline"
risk_class: "product-scope-runtime-boundary"
owner: "host-orchestrator"
revisit_condition: "用户后续修改 operational scope 时，回退到 CR137 handoff、CR138 CP2 approved baseline 和归一化审查记录后重新评估。"
acceptance_criteria: "冻结 Runner 与 QMT Gateway 的真实运营使用场景：多因子运行、盘前确认、执行跟踪、事件/信号接入、组合再平衡、盘中运维、日志审计、盘后复盘、异常恢复、策略调整/发布/暂停/回滚，以及 QMT gateway 的启动、健康、交易日历、佣金 / 费用模型、收益 / PnL、账户/持仓/订单/成交查询、行情订阅、下单/撤单/回报、故障恢复、审计和配置变更。不得在 CP3 approval 中自动授权 runtime/NAS/QMT/凭据/provider/lake/catalog/trading；后续必要验证可按 action scope、运行窗口、脱敏、回滚和审计范围单独授权。"
close_condition: "CP2 use-case baseline approved；后续进入功能模型/蓝图/HLD 前，CR138 必须记录当前 gate 未授权范围、后续按需 runtime_authorization 路径和 CR 分流。"
conflict_keys:
  - runner-operational-use-case
  - qmt-gateway-operational-use-case
  - strategy-runner-core
  - qmt-gateway-runtime-boundary
  - scoped_runtime_authorization
impact_surface:
  - scripts/check_process_artifact_hygiene.py
  - tests/test_cr132_process_artifact_hygiene.py
  - process/USE-CASES.md
  - process/changes/CR-138-RUNNER-QMT-GATEWAY-OPERATIONAL-USE-CASE-BASELINE-2026-06-24.md
  - process/context/CP2-CR138-RUNNER-QMT-OPERATIONAL-USE-CASE-CONTEXT.yaml
  - process/checks/CP2-CR138-RUNNER-QMT-OPERATIONAL-USE-CASE-PRECHECK.md
  - process/checks/CR138-USE-CASE-NORMALIZATION-AUDIT-2026-06-24.md
  - process/checkpoints/CP2-CR138-RUNNER-QMT-OPERATIONAL-USE-CASE-REVIEW.md
  - process/discussions/CP2-SCENARIO-DISCUSSION-LOG.md
  - process/checks/CP2-DISCUSSION-CHECKPOINT.json
  - process/docs/design/HLD-RUNNER-QMT-OPERATIONAL-CONTROL-PLANE.md
  - process/docs/design/ARCHITECTURE-DECISION-RUNNER-QMT-OPERATIONAL-CONTROL-PLANE.md
  - process/docs/design/BLUEPRINT.md
  - process/docs/design/DOMAIN-MAP.md
  - process/docs/design/DEPENDENCY-MAP.md
  - process/context/CP3-CR138-RUNNER-QMT-HLD-CONTEXT.yaml
  - process/checks/CP3-CR138-HLD-CONSISTENCY.md
  - process/checkpoints/CP3-CR138-RUNNER-QMT-HLD-REVIEW.md
  - process/discussions/CP3-HLD-DISCUSSION-LOG.md
  - process/checks/CP3-DISCUSSION-CHECKPOINT.json
  - process/checks/CR138-PROCESS-ARTIFACT-HYGIENE-GUARDRAIL-2026-06-24.md
  - process/docs/design/FEATURE-DESIGN-MATRIX.md
  - process/docs/features/runner-control-plane/DESIGN.md
  - process/docs/features/runner-control-plane/TEST-PLAN.md
  - process/docs/features/runner-control-plane/TASKS.md
  - process/docs/features/qmt-gateway-service-layer/DESIGN.md
  - process/docs/features/qmt-gateway-service-layer/TEST-PLAN.md
  - process/docs/features/qmt-gateway-service-layer/TASKS.md
  - process/STORY-BACKLOG.md
  - process/STORY-BACKLOG-CR138.md
  - process/STORY-STATUS.md
  - process/STORY-STATUS-CR138.md
  - process/DEVELOPMENT-PLAN.yaml
  - process/DEVELOPMENT-PLAN-CR138.yaml
  - process/checks/CP4-CR138-STORY-DAG-PARALLEL-SAFETY.md
  - process/checks/CP4-CR138-STORY-DAG-PARALLEL-SAFETY.result.json
  - process/stories/CR138-S01-shared-contracts-authorization-audit.md
  - process/stories/CR138-S02-runner-plan-preflight-control.md
  - process/stories/CR138-S03-runner-event-signal-rebalance-tracking.md
  - process/stories/CR138-S04-runner-evidence-review-incident-lifecycle.md
  - process/stories/CR138-S05-gateway-lifecycle-health-rest-contract.md
  - process/stories/CR138-S06-gateway-query-calendar-commission-pnl.md
  - process/stories/CR138-S07-gateway-subscription-order-report-recovery.md
  - process/stories/CR138-S08-docs-fixtures-cp7-authorization-runbook.md
  - process/stories/CR138-S01-shared-contracts-authorization-audit-LLD.md
  - process/stories/CR138-S02-runner-plan-preflight-control-LLD.md
  - process/stories/CR138-S03-runner-event-signal-rebalance-tracking-LLD.md
  - process/stories/CR138-S04-runner-evidence-review-incident-lifecycle-LLD.md
  - process/stories/CR138-S05-gateway-lifecycle-health-rest-contract-LLD.md
  - process/stories/CR138-S06-gateway-query-calendar-commission-pnl-LLD.md
  - process/stories/CR138-S07-gateway-subscription-order-report-recovery-LLD.md
  - process/stories/CR138-S08-docs-fixtures-cp7-authorization-runbook-LLD.md
  - process/checks/CP5-CR138-S01-shared-contracts-authorization-audit-LLD-IMPLEMENTABILITY.md
  - process/checks/CP5-CR138-S02-runner-plan-preflight-control-LLD-IMPLEMENTABILITY.md
  - process/checks/CP5-CR138-S03-runner-event-signal-rebalance-tracking-LLD-IMPLEMENTABILITY.md
  - process/checks/CP5-CR138-S04-runner-evidence-review-incident-lifecycle-LLD-IMPLEMENTABILITY.md
  - process/checks/CP5-CR138-S05-gateway-lifecycle-health-rest-contract-LLD-IMPLEMENTABILITY.md
  - process/checks/CP5-CR138-S06-gateway-query-calendar-commission-pnl-LLD-IMPLEMENTABILITY.md
  - process/checks/CP5-CR138-S07-gateway-subscription-order-report-recovery-LLD-IMPLEMENTABILITY.md
  - process/checks/CP5-CR138-S08-docs-fixtures-cp7-authorization-runbook-LLD-IMPLEMENTABILITY.md
  - process/checks/CP5-CR138-RUNNER-QMT-OPERATIONAL-CONTROL-LLD-BATCH.result.json
  - process/context/CP5-CR138-RUNNER-QMT-OPERATIONAL-CONTROL-CONTEXT.yaml
  - process/checkpoints/CP5-CR138-RUNNER-QMT-OPERATIONAL-CONTROL-LLD-BATCH.md
  - process/checks/CP5-CR138-HUMAN-GATE-LAUNCH-MESSAGE.md
  - process/checks/CR138-FOLLOW-UP-CR-COVERAGE-AUDIT-2026-06-24.md
  - process/state/CHECKPOINT-LEDGER.ndjson
  - process/state/READ-EXPANSION-LEDGER.ndjson
authz_policy_refs:
  - "NO_CREDENTIAL_READ"
  - "NO_RUNTIME"
  - "NO_NAS"
  - "NO_TRADING"
  - "NO_PROVIDER_LAKE_CATALOG"
  - "NO_REMOTE_WRITE"
risk_refs:
  - "RISK-CR138-RUNTIME-MISREAD"
  - "RISK-CR138-RUNTIME-NOT-VERIFIED"
cp8_approval_result: "approved"
cp8_approved_by: "user"
cp8_approved_at: "2026-06-24T18:58:00+08:00"
release_decision: "READY_WITH_RISK"
release_context_ref: "process/release/RELEASE-CONTEXT-CR138.yaml"
cp8_checkpoint_ref: "process/checkpoints/CP8-CR138-DELIVERY-READINESS.md"
cr_index_path: "process/changes/CR-INDEX.yaml"
current_requirement_baseline_path: "process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml"
historical_baseline_status: "active"
---

# CR-138 Runner & QMT Gateway Operational Use-Case Baseline

## 变更描述

用户要求读取 `process/context/CR137-CLOSURE-TO-RUNNER-QMT-USE-CASE-CONTEXT-RESET-HANDOFF-2026-06-24.md`，并充分参考两份 QMT 生态源码深度研究报告，推进 `CR138 Runner & QMT Gateway Operational Use-Case Baseline`。

本 CR 的目标不是继续补 Runner 离线小功能，也不是实现 online runtime。目标是先冻结真实用户如何使用 Runner 与 QMT Gateway：

- Runner 负责“策略如何运行、如何观察、如何调整、如何复盘”。
- QMT Gateway 负责“如何安全稳定地触达 QMT / MiniQMT / XtQuant”。
- 后续功能模型、边界设计、HLD、LLD 和实现必须以本 use-case baseline 为输入。

## 参考输入

| 输入 | 用途 | 结论 |
|---|---|---|
| `process/context/CR137-CLOSURE-TO-RUNNER-QMT-USE-CASE-CONTEXT-RESET-HANDOFF-2026-06-24.md` | 恢复当前上下文和用户纠偏 | CR137 已关闭；推荐启动 CR138；本轮不自动授权 runtime / NAS / QMT / credentials / trading。 |
| `process/docs/features/strategy-runner-core/DESIGN.md` | 当前 Runner Core feature authority | 现有 Runner Core 是 offline-first，不等于 operational runtime。 |
| `process/USE-CASES.md` | 既有 UC-28..UC-32 双目标策略交付框架基线 | 旧基线只覆盖策略交付框架和 runner 安装设计，不足以描述日常运营使用方式。 |
| `process/docs/design/BLUEPRINT.md` | FEAT-05 / FEAT-06 / FEAT-09 边界 | FEAT-05 是 QMT gateway / readonly admission，FEAT-06 是 OMS/风控/阶段激活，FEAT-09 是策略交付合同。 |
| `lite-qmt-executor、BulletTrade、EasyXT 源码深度研究报告.md` | Runner / runtime / strategy samples 参考 | Runner 应作为策略运行控制面；要覆盖任务状态机、事件循环、组合上下文、订单队列、风控、恢复和日志审计。 |
| `quant-qmt-proxy 与 qmt-bridge 深度研究报告.md` | QMT gateway 服务化参考 | Gateway 要作为服务化运行层；重点是协议分层、生命周期、会话/账户、订阅、xtdata 串行化、故障恢复和审计。 |

## CR 类型与门禁策略

| 字段 | 内容 |
|---|---|
| CR 类型 | `product-scope` |
| Legacy CR kind | `requirement-change` |
| 生命周期状态 | `active` |
| 就绪状态 | `not_ready` |
| 门禁状态 | `cp5_pending` |
| 门禁模板 | `standard` |

### 门禁分类说明

`meta-flow gate classify` 对包含 `QMT` / `gateway` 的变更会给出 `runtime-high-risk` 建议，并要求 CP2 / CP3 / CP5 / CP8 人工门禁。本 CR 接受该高风险门禁强度，但 CR tracking legacy 索引当前合法 `gate_profile` 值为 `standard` / `compact` / `runtime` / `spike` / `full`，因此索引字段保持 `standard`，避免把 use-case baseline 误登记成 runtime 授权 CR。

本说明不改变本轮执行边界：CR138 CP3 不自动授权 runtime、QMT、MiniQMT、XtQuant、gateway 连接、凭据读取、账户 / 行情 / 订单读取、submit/cancel、simulation/live、NAS、provider/lake/catalog 或 Git remote 写入。后续如确有验证必要，允许按 action scope、运行窗口、脱敏、回滚和审计范围单独发起 `runtime_authorization` gate。

## 结构化权限策略

```yaml
authorization_policy:
  nas:
    access: false
    list: false
    read: false
    write: false
    publish: false
    delete: false
  credentials:
    env_read: false
    secret_read: false
    account_read: false
  runtime:
    qmt: false
    miniqmt: false
    xtquant: false
    gateway: false
  trading:
    submit: false
    cancel: false
    simulation: false
    live: false
  provider_lake_catalog:
    provider_fetch: false
    lake_write: false
    catalog_publish: false
  git_remote:
    commit: false
    push: false
```

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/USE-CASES.md` | 原文档更新 | 保留 UC-01 至 UC-32；新增 UC-33 至 UC-50；新增场景归一化与合并审查；CP2 已批准并标记为 confirmed | `## 修订记录` v1.16 | approved-cp2 |
| `process/REQUIREMENTS.md` | 不变 | 旧 REQ-001 至 REQ-200 保留；CR138 CP2 后再进入 requirement-extraction 增量 | 不适用 | pending |
| `process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml` | 不变 | 当前 baseline 保留；CR138 仅先冻结 use-case，不改变可执行需求真相 | 不适用 | pending |
| `process/docs/features/strategy-runner-core/*` | 不变 | CR128..CR137 offline runner feature authority 保留 | 不适用 | n/a |
| `process/docs/design/BLUEPRINT.md` | 增量更新 | FEAT-05/06/09 边界作为输入；新增 FEAT-11 / FEAT-12、REST-only P0、按需授权和查询服务边界 | `## 修订记录` v1.5 | approved-cp3 |
| `process/docs/design/HLD.md` / `ARCHITECTURE-DECISION.md` | 刷新 current index | 长期设计入口以功能域命名，不使用 CR 命名主文件 | `## 修订记录` HLD v1.4 / ADR v1.5 | approved-cp3 |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| UC-28 QMT / MiniQMT 双目标策略交付框架 | UC-33..UC-50 Runner / QMT Gateway 运营场景 | 原文保留 | CR046 描述策略包交付框架；CR138 描述用户如何日常运行、观察、调整、恢复。 |
| UC-29 QMT 终端策略包交付形态 | UC-46..UC-50 Gateway 查询、交易事件、审计与配置运营 | 原文保留 | 终端策略包不是 Gateway 服务化运营基线；Gateway 需单独描述能力探测、查询、订阅、回报和恢复。 |
| UC-30 MiniQMT Runner 安装设计 | UC-33..UC-43 Runner 运营场景 | 原文保留 | 安装设计只说明如何部署；运营场景说明多因子、事件、信号、再平衡、盘中、盘后、异常和策略变更如何使用 Runner。 |
| UC-31 双目标验证框架 | CR138 验证场景矩阵 | 原文保留 | CR138 不做实现验证，只冻结后续测试矩阵必须覆盖的运营路径。 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否新增或重定义 REQ-* | Runner / QMT Gateway operational baseline | true | 本轮已确认 use-case baseline；REQ 增量留到 CP2 后的 requirement extraction。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | `process/USE-CASES.md`、后续 TEST-MATRIX | true | 新增 UC-33..UC-50 和 TS-138-*。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | 后续功能模型、HLD、Story 拆分 | true | 后续必须先做功能模型与边界设计，不继续按离线小 slice 拆 CR。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | runtime / QMT / credential / trading boundary | true | CP3 不自动授权；真实运行、查询账户、下单撤单等必须按需另起 runtime authorization gate。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | CP2 / CP3 context/check/checkpoint、use-case baseline、CP4 前置 guardrail | true | 写 CP2/CP3 预检和人工确认草稿；CR138-GUARDRAIL-01 已按用户确认修复 hygiene checker 并运行测试。 |

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量文档修改 | false | 场景会改变后续功能模型和运行授权边界。 |
| 修改架构、权限、安全边界或平台安装路径 | true | 触及 QMT Gateway / Runner operational boundary。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | true | 后续将影响 Runner / Gateway / OMS / risk / audit 拆分。 |
| 需要 HLD / LLD 才能解释影响 | true | CP2 后必须进入功能模型 / HLD。 |
| 是否保持 fast-lane | false | 维持 `workflow_mode_after_change=standard`。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：true
- batch_id：`CR138-RUNNER-QMT-OPERATIONAL-CONTROL-BATCH-A`
- 批次范围来源：CP4 Story DAG / Wave / file owner 计划、FEATURE-DESIGN-MATRIX v1.5、FEAT-11 / FEAT-12 设计三件套
- 批次内 Story：S01..S08，全部 `full-lld`
- 当前状态：8 份 full LLD 已生成；8 份 Story 级 CP5 implementability 预检 PASS；批次级 CP5 result PASS；人工门禁待用户确认
- 开发启动条件：仅在 CP5 人工确认 `approve` 后，Story 才可进入后续受控 `dev-ready` 候选；即便 CP5 approve，也不自动授权真实 runtime、QMT / MiniQMT / XtQuant / gateway、凭据、账户 / 行情 / 订单查询、submit/cancel、simulation/live、NAS、provider/lake/catalog 或 Git remote 写入。

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 创建 CR138 并完成冲突预检 | CR137 handoff、用户请求、研究报告 | 本 CR、CR index、state、CP2 context | CR 已登记 | 更新 use-case baseline |
| 2 | `host-orchestrator` | 增量更新 use-case baseline | 研究报告、既有 UC-28..UC-32、Blueprint | `process/USE-CASES.md` v1.16 baseline | CP2 precheck | 发起人工确认 |
| 3 | `host-orchestrator` | 按用户反馈执行 use-case 合并 / 刷新审查 | UC-01..UC-50、旧 QMT / CR046 场景、CR138 baseline | `process/checks/CR138-USE-CASE-NORMALIZATION-AUDIT-2026-06-24.md`、USE-CASES 归一化映射 | CP2 review | 重新发起人工确认 |
| 4 | `host-orchestrator` | 形成 CP2 Decision Brief | context、discussion log、precheck、normalization audit | `process/checkpoints/CP2-CR138-...-REVIEW.md` | 等待用户确认 | `approve` 后进入 requirement extraction / feature model |
| 5 | `meta-se` / `host-orchestrator` | 形成并修订 CP3 solution-design | 已批准 use-case baseline、两份 QMT 生态研究报告、用户 CP3 反馈 | 功能命名 HLD/ADR、BLUEPRINT/DOMAIN/DEPENDENCY 更新、CP3 review | CP3 人工确认 | 用户 `approve` 后进入 CP4 Story planning |
| 6 | `host-orchestrator` | 执行 CP4 前置 hygiene guardrail task | 用户确认“方案 C + 方案 A，不新起 CR，放到 CR138” | `scripts/check_process_artifact_hygiene.py`、`tests/test_cr132_process_artifact_hygiene.py`、`process/checks/CR138-PROCESS-ARTIFACT-HYGIENE-GUARDRAIL-2026-06-24.md` | 测试与 checker 验证通过 | 继续 CP4 Story planning |
| 7 | `host-orchestrator` | 执行 CP4 Story planning | CP3 approved HLD/ADR、FEAT-11 / FEAT-12 边界、用户 CP3 反馈 | `process/docs/design/FEATURE-DESIGN-MATRIX.md`、FEAT-11 / FEAT-12 设计三件套、`process/STORY-BACKLOG-CR138.md`、`process/DEVELOPMENT-PLAN-CR138.yaml`、8 张 Story 卡、`process/checks/CP4-CR138-STORY-DAG-PARALLEL-SAFETY.md` | CP4 自动预检 PASS | 进入全量 LLD 设计证据准备；CP5 前不得实现 |
| 8 | `host-orchestrator` | 准备 CP5 full LLD 设计证据并发起人工门禁 | CP4 PASS、8 张 Story 卡、Feature Matrix、Feature Design | 8 份 Story LLD、8 份 CP5 implementability 预检、CP5 batch result、Context Capsule、manual checkpoint、launch message、READ ledger | CP5 自动预检 PASS；human-gate checker PASS | 等待用户 `approve` / `修改` / `reject` |
| 9 | `host-orchestrator` | 审计原后续 CR 与 CR138 Story 覆盖关系 | CR-INDEX、CURRENT-REQUIREMENT-BASELINE、CR091/098/126/137 follow-up 证据、CR138 Story Plan | `process/checks/CR138-FOLLOW-UP-CR-COVERAGE-AUDIT-2026-06-24.md`、S02/S03 batch 设计增量、CR-INDEX / baseline 后续计划收敛 | 自动检查待重跑 | CP5 仍 pending；等待用户 `approve` / `修改` / `reject` |

## 不授权范围

- 本 CP3 不读取 `.env`、token、secret、账号、账户、资金、持仓、委托、成交、session、cookie 或原始日志；后续账户只读验证必须单独授权并脱敏。
- 不访问、列取、挂载、读取、复制、写入、发布、pull 或删除真实 NAS。
- 本 CP3 不启动、连接、安装或运行 QMT / MiniQMT / XtQuant / gateway runtime。
- 本 CP3 不执行 submit/cancel、buy/sell、simulation/live。
- 不执行 provider fetch、lake write、catalog publish 或 current pointer 修改。
- 不 clone / install / run 外部项目；两份研究报告作为本轮参考输入足够。
- 除 CR138-GUARDRAIL-01 已确认的 `scripts/check_process_artifact_hygiene.py` 与 `tests/test_cr132_process_artifact_hygiene.py` 修复外，不修改源码、测试、依赖、安装脚本或 Git remote。

## 处理结论

- 当前状态：`active / cp5_pending / not_ready`。
- 已完成：CR138 正式启动、冲突预检、use-case baseline、UC-01..UC-50 合并 / 刷新审查、CP2 context、CP2 precheck、CP2 review gate；用户于 2026-06-24T13:20:00+08:00 回复 `approve` 通过 CP2。CP3 solution-design 已按用户反馈修订并由用户于 2026-06-24T15:30:00+08:00 回复 `approve` 通过：长期 HLD/ADR 改为功能域命名，Gateway P0 收敛为 REST-only，runtime policy 改为 CP3 不自动授权但后续可按需授权，交易日历、佣金 / 费用模型、收益 / PnL 查询已纳入 FEAT-12 Query Service。用户随后确认先执行方案 C + 方案 A，不新起 CR；CR138-GUARDRAIL-01 已完成 hygiene checker 修复，验证结果见 `process/checks/CR138-PROCESS-ARTIFACT-HYGIENE-GUARDRAIL-2026-06-24.md`。CP4 Story planning 已完成：FEAT-11 / FEAT-12 设计三件套、Feature Design Matrix、8 个 Story、4 个 Wave、Development Plan 和 CP4 自动预检均已生成，CP4 结论 PASS。CP5 设计证据已补齐：8 份 full LLD、8 份 Story 级 implementability 预检、CP5 batch result、Context Capsule、manual checkpoint 和 launch message 均已生成并通过自动检查。
- 待推进：等待用户审查 `process/checkpoints/CP5-CR138-RUNNER-QMT-OPERATIONAL-CONTROL-LLD-BATCH.md` 并回复 `approve` / `修改: <具体修改点>` / `reject`。
- 推荐下一步：若用户 `approve`，将 S01..S08 从 `lld-ready-for-review` 推进为后续受控 `dev-ready` 候选；仍不得自动启动实现、真实 runtime、QMT、MiniQMT、XtQuant、凭据、账户 / 行情 / 订单读取、submit/cancel、simulation/live、NAS、provider/lake/catalog 或 Git remote 写入；任何真实验证仍需单独 runtime_authorization gate。

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| Parent CR | `CR-137` | Offline Runner Run Registry 已关闭，交接建议启动 CR138。 |
| Context handoff | `process/context/CR137-CLOSURE-TO-RUNNER-QMT-USE-CASE-CONTEXT-RESET-HANDOFF-2026-06-24.md` | 本 CR 的恢复入口。 |
| Source report | `lite-qmt-executor、BulletTrade、EasyXT 源码深度研究报告.md` | Runner / runtime / strategy sample 输入。 |
| Source report | `quant-qmt-proxy 与 qmt-bridge 深度研究报告.md` | QMT Gateway 服务化输入。 |
| Use-case baseline | `process/USE-CASES.md` | CR138 v1.16 confirmed。 |
