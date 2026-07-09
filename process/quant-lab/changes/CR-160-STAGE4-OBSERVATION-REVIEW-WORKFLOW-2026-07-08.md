---
cr_id: "CR-160"
cr_type: "product-scope"
cr_kind: "requirement-change"
title: "Stage 4 Observation Review Workflow"
lifecycle_status: "closed"
readiness_status: "ready_with_risk"
gate_status: "delivered"
status: "closed"
gate_profile: "standard-lite"
route_strategy: "standard-governance; literal gate_profile=standard is not supported by current meta-flow route planner"
route_plan_ref: "process/checks/CP0-CR160.route-plan.json"
cr_trait_uses_existing_evidence_only: false
cr_trait_has_new_design: true
cr_trait_has_new_implementation: false
cr_trait_has_new_verification: true
cr_trait_requires_architecture_review: true
cr_trait_requires_story_decomposition: false
cr_trait_requires_subagent_dispatch: true
cr_trait_verification_waiver_reason: ""
cr_trait_verification_waiver_ref: ""
impact_level: "high"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "product baseline and Stage 4 gate semantics require CP2/CP3/CP8 human gates"
rollback_to: "delivered / no active formal CR baseline after CR159 closure"
approval_result: "approved"
created_at: "2026-07-08T21:55:00+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-07-09T20:18:07+08:00"
source: "user"
linked_issue: ""
parent_cr: "CR-157"
source_checkpoint: "CR157 backlog / MVP deferred item"
source_decision_id: "DF-CR157-003"
follow_up_type: "stage4-observation-review-workflow"
risk_class: "product-scope-process-design-no-runtime"
owner: "host-orchestrator"
revisit_condition: "Mature admission package PASS or user chooses to design Stage 4 observation/simulation authorization gate"
acceptance_criteria: "Define observation_plan_ref content, Stage 4 observation review workflow, gate approval decision table, CR155 blocked sample classification, checklist, and authorization boundary without authorizing runtime."
close_condition: "CP8 approves the workflow/gate contract delivery, product baseline refresh is complete across 6 product docs, and follow-up routes are recorded for strategy remediation, real-data validation, paper authorization, runtime authorization, data-lake governance, or checker/schema implementation."
cr_index_path: "process/changes/CR-INDEX.json"
current_requirement_baseline_path: "process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml"
historical_baseline_status: "active"
reframed_by: []
reframe_summary: ""
goal_ref: "BL-CR157-003"
goal_statement: "Fill CR157 Stage 4 observation review semantic gap without promoting blocked artifacts or authorizing runtime."
user_goal_impact: "Clarifies whether future strategy artifacts can enter observation review, need remediation, need real-data evidence, or require separate simulation/paper/runtime authorization."
approval_focus: "scope|architecture|security|runtime|risk"
split_rationale: "CR157 reserved observation_plan_ref and stage4_observation_gate_approved semantics but did not define their content or workflow; CR158 closed adapter scope and CR159 closed process hygiene, leaving Stage 4 review as the next product baseline decision."
why_not_merge_with_parent: "CR157 is closed and CR158 already consumed event/ML adapter deferred items; CR160 has a distinct Stage 4 review/gate objective and authorization boundary."
why_not_story_or_task: "This changes product scope and gate semantics before any implementation Story can be safely decomposed."
decision_burden: "medium"
approve_effect: "Approve CP0 intake and route CR160 to requirement clarification / CP2 for evidence profile, CR155 seed classification, deliverable shape, authorization boundary, and CP4/CP5 route decisions."
reject_effect: "Stage 4 observation review remains undefined; CR157 observation_plan_ref and stage4_observation_gate_approved stay as unresolved semantic placeholders."
not_authorized_by_approve:
  - "simulation_or_paper_run"
  - "live_or_trading_runtime"
  - "qmt_miniqmt_xtquant_gateway_runtime"
  - "broker_operation_or_order_write"
  - "credential_or_env_read"
  - "new_real_lake_read"
  - "real_lake_write"
  - "nas_read_write_sync_or_metadata_normalization"
  - "provider_fetch"
  - "catalog_store_registry_model_prediction_write"
  - "git_remote_write"
  - "true_release_execution_or_publish"
product_baseline_refresh_required: true
required_phase: "documentation"
required_agent: ""
required_gate: "CP8"
block_story_decomposition_until: "N/A; CR160 design-only route keeps CP4/CP5/CP6 N/A"
affected_product_docs:
  - "docs/product/USE-CASES.md"
  - "docs/product/REQUIREMENTS.md"
  - "docs/product/SCENARIOS.yaml"
  - "docs/product/TEST-MATRIX.md"
  - "docs/product/MVP-SCOPE.md"
  - "docs/product/BACKLOG.md"
affected_use_cases:
  - "UC-58 multifactor strategy E2E"
  - "DF-CR157-003 Stage 4 observation review workflow"
routing_design_ref: "process/checks/CP0-CR160.route-plan.json"
summary_ref: "process/changes/summaries/CR-160.summary.json"
cp0_result_ref: "process/checks/CP0-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-REQUEST-INTAKE.result.json"
cp0_context_ref: "process/context/CP0-CR160.context.json"
cp1_result_ref: "process/checks/CP1-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-USE-CASE-COMPLETENESS.result.json"
cp2_result_ref: "process/checks/CP2-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-SCOPE.result.json"
cp2_context_ref: "process/context/CP2-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-SCOPE-CONTEXT.yaml"
cp2_checkpoint_ref: "process/checkpoints/CP2-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-SCOPE.md"
cp3_result_ref: "process/checks/CP3-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-HLD-CONSISTENCY.result.json"
cp3_context_ref: "process/context/CP3-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-DESIGN-CONTEXT.yaml"
cp3_checkpoint_ref: "process/checkpoints/CP3-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-HLD-REVIEW.md"
cp4_result_ref: ""
cp5_result_ref: ""
cp6_result_ref: ""
cp7_result_ref: "process/checks/CP7-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-VERIFICATION.result.json"
cp7_evidence_ref: "process/evidence/CR160-CP7-VERIFICATION.index.json"
cp7_return_ref: "process/returns/CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW.CP7.return.json"
cp8_context_ref: "process/context/CP8-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-CONTEXT.yaml"
cp8_result_ref: "process/checks/CP8-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-RELEASE-READINESS.result.json"
cp8_checkpoint_ref: "process/checkpoints/CP8-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-RELEASE-READINESS.md"
cp8_launch_message_ref: "process/checks/CP8-CR160-HUMAN-GATE-LAUNCH-MESSAGE.md"
release_context_ref: "process/release/RELEASE-CONTEXT-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW.yaml"
risk_refs:
  - "R-CR160-READY-WITH-RISK-MISREAD"
  - "R-CR160-CR155-BLOCKED-ARTIFACT-PROMOTION"
  - "R-CR160-RUNTIME-AUTHORIZATION-LEAK"
  - "R-CR160-GATE-PROFILE-NAME-MISMATCH"
  - "R-CR160-CP7-DESIGN-ONLY"
  - "R-CR160-FUTURE-CHECKER-DEFERRED"
authz_policy_refs:
  - "NO_CREDENTIAL_READ"
  - "NO_RUNTIME"
  - "NO_PRODUCTION_WRITE"
  - "NO_TRADING"
  - "NO_REAL_LAKE_WRITE"
  - "NO_NAS_SYNC_OR_WRITE"
  - "NO_PROVIDER_FETCH"
  - "NO_BROKER_WRITE"
  - "NO_EXTERNAL_FRAMEWORK_RUN"
  - "NO_CATALOG_POINTER_WRITE"
---

# CR-160 Stage 4 Observation Review Workflow

## 变更描述

用户要求启动 `CR160 CP0 请求受理`，承接 `BL-CR157-003 / DF-CR157-003 Stage 4 observation review workflow`。本 CR 的目标是补齐 CR157 已预留但未定义的 Stage 4 语义：`observation_plan_ref` 指向的 observation plan 内容、`stage4_observation_gate_approved` 的批准条件、observation review workflow、review checklist、authorization gate contract 和 fail-closed 分类规则。

本 CR 不是把 CR155 推入 observation / paper / simulation。CR155 只能作为 `blocked_admission_failed` 的标准反例样本：真实 lake readonly validation 和 rerun consistency 已执行并通过，但 admission package 为 `BLOCKED`、statistical gate 为 `FAIL`、`paper_candidate=false`。

## 目标影响摘要

| 字段 | 内容 |
|---|---|
| 目标引用 | `BL-CR157-003` |
| 整体目标 | 定义 Stage 4 observation review workflow 和前置授权门禁，补齐 CR157 Stage3 handoff 后续语义。 |
| 用户目标影响 | 后续策略 artifact 可以被一致地判定为 `eligible_for_observation_review`、`needs_real_data_evidence`、`needs_strategy_remediation`、`needs_runtime_authorization` 或 `blocked`。 |
| 本 CR 为什么值得独立推进 | 它改变产品范围和 gate semantics，不是单个 Story 或代码任务；同时涉及 READY_WITH_RISK、BLOCKED、paper_candidate=false 和 runtime authorization 的边界。 |
| approve 后会发生什么 | 进入 CP1/CP2 产品澄清；CP2 将确认 evidence profile、CR155 seed classification、deliverable shape、authorization boundary 和 CP4/CP5 route。 |
| reject / 不确认会阻塞什么 | Stage 4 observation review 仍未定义，`observation_plan_ref` 和 `stage4_observation_gate_approved` 只能继续作为占位语义存在。 |
| 决策负担 | `medium`：范围清楚，但需要用户确认证据层级、交付形态和授权边界。 |

## 拆分理由

| 问题 | 结论 |
|---|---|
| 为什么不合并到 parent / active CR | 当前无 active formal CR；CR157、CR158、CR159 均已关闭，CR160 目标与它们的关闭范围不同。 |
| 为什么不是 Story / task / follow-up | CP2 前必须先定义产品范围、证据分级和安全边界；否则 Story 拆解会误把 blocked artifact 推进 observation。 |
| 触发独立 CR 的边界 | Stage 4 gate 语义、observation plan 合同、authorization boundary、CR155 fail-closed 样例、后续 runtime/paper authorization 路由。 |

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-157` |
| 来源检查点 | `CR157 backlog / MVP deferred item` |
| 来源决策 ID | `DF-CR157-003` |
| follow-up 类型 | `stage4-observation-review-workflow` |
| 风险等级 | `product-scope-process-design-no-runtime` |
| owner | `host-orchestrator` |
| 重访条件 | Mature admission package PASS 或用户要求设计 Stage 4 observation / simulation authorization gate。 |
| 验收标准 | 定义 observation plan、review workflow、decision table、checklist、authorization boundary 和 CR155 blocked 样例分类。 |
| 关闭条件 | CP8 确认 CR160 只交付 gate contract / workflow design，产品基线刷新已完成，不授权 runtime/paper/live。 |

## CR 类型与门禁策略

| 字段 | 内容 |
|---|---|
| CR 类型 | `product-scope` |
| Legacy CR kind | `requirement-change` |
| 生命周期状态 | `closed` |
| 就绪状态 | `ready_with_risk` |
| 门禁状态 | `delivered` |
| 门禁模板 | `standard-lite`（当前 route planner 不支持 literal `standard`；本 CR 保留 standard-governance 语义） |
| Route plan | `process/checks/CP0-CR160.route-plan.json` |

### CR Trait 与实际路由

| trait 字段 | 值 | 路由含义 |
|---|---|---|
| uses existing evidence only | `false` | CR160 不是只消费已有证据；它会新增 Stage 4 review/gate design。 |
| has new design | `true` | CP3 必须设计 review workflow 和 gate contract。 |
| has new implementation | `false` | 默认纯设计路径；是否实现 schema/checker 由 CP2 决策。 |
| has new verification | `true` | CP7 必须验证 checklist 可测试性、CR155 fail-closed classification 和 no-overclaim wording。 |
| requires architecture review | `true` | Stage 4 gate 语义影响跨 CR 设计边界，CP3 人工门禁必需。 |
| requires story decomposition | `false` | CP2 选择纯设计路径时无 Story；若选择代码实现，则 route plan 必须升级并重算 CP4/CP5。 |
| requires subagent dispatch | `true` | CP6/CP7 设计审查必须有 meta-dev/meta-qa dispatch 或批准的 inline-fallback 证据。 |

## Checkpoint Index

| CP | 状态 | 机器结果 ref | 人工门禁 ref | Context ref | Ledger event ref | 摘要 |
|---|---|---|---|---|---|---|
| CP0 | pass | `process/checks/CP0-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-REQUEST-INTAKE.result.json` | N/A | `process/context/CP0-CR160.context.json` | `process/state/CHECKPOINT-LEDGER.ndjson` | 请求已受理；route plan 已生成；无 active / blocked formal CR 冲突。 |
| CP1 | pass | `process/checks/CP1-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-USE-CASE-COMPLETENESS.result.json` | N/A | `process/context/CP2-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-SCOPE-CONTEXT.yaml` | `process/state/CHECKPOINT-LEDGER.ndjson` | 产品基线 completeness 预检通过；Stage 4 场景、文档更新面和 no-runtime 边界已覆盖。 |
| CP2 | approved | `process/checks/CP2-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-SCOPE.result.json` | `process/checkpoints/CP2-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-SCOPE.md` | `process/context/CP2-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-SCOPE-CONTEXT.yaml` | `process/state/GATE-LEDGER.ndjson` | 用户已确认 evidence profile、CR155 seed classification、deliverable shape、authorization boundary、CP4/CP5/CP6 route。 |
| CP3 | approved | `process/checks/CP3-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-HLD-CONSISTENCY.result.json` | `process/checkpoints/CP3-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-HLD-REVIEW.md` | `process/context/CP3-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-DESIGN-CONTEXT.yaml` | `process/state/GATE-LEDGER.ndjson` | 用户已批准 HLD、checklist、CR155 seed classification、template/instance 边界、contract-only fail-closed ceiling 和 non-authorization boundary。 |
| CP4 | n/a-by-route | N/A | N/A | N/A | `process/state/CHECKPOINT-LEDGER.ndjson` | 当前纯设计初始 route 不做 Story decomposition；CP2 可改为代码路径。 |
| CP5 | n/a-by-route | N/A | N/A | N/A | `process/state/GATE-LEDGER.ndjson` | 当前纯设计初始 route 不做 LLD batch；CP2 可改为代码路径。 |
| CP6 | n/a-by-route | N/A | N/A | N/A | `process/state/CHECKPOINT-LEDGER.ndjson` | 当前无代码实现；CP6 保持 N/A。设计交付物在 CP7 前作为验证输入记录。 |
| CP7 | pass-with-risk | `process/checks/CP7-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-VERIFICATION.result.json` | N/A | `process/context/CP7-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-VERIFICATION-CONTEXT.yaml` | `process/state/CHECKPOINT-LEDGER.ndjson` | meta-qa 已完成设计静态验证；结论 `PASS_WITH_RISK`，无 blocker，风险进入 CP8 接受项。 |
| CP8 | approved | `process/checks/CP8-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-RELEASE-READINESS.result.json` | `process/checkpoints/CP8-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-RELEASE-READINESS.md` | `process/context/CP8-CR160-STAGE4-OBSERVATION-REVIEW-WORKFLOW-CONTEXT.yaml` | `process/state/GATE-LEDGER.ndjson` | 发布就绪预检 PASS；产品基线刷新已补齐；用户确认 `READY_WITH_RISK`、不授权边界和 follow-up candidate tracking。 |

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
    simulation: false
    paper: false
    live: false
  trading:
    submit: false
    cancel: false
    broker_operation: false
  data:
    new_real_lake_read: false
    real_lake_write: false
    provider_fetch: false
    catalog_pointer_mutation: false
    store_or_registry_write: false
```

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `docs/product/USE-CASES.md` | 原文档更新 | 保留 CR157/CR158 deferred mapping，追加 CR160 Stage 4 review use case | `## 修订记录` | pending CP2 |
| `docs/product/REQUIREMENTS.md` | 原文档更新 | 保留 REQ-CR157/CR158，追加 CR160 observation review requirements | `## 修订记录` | pending CP2 |
| `docs/product/SCENARIOS.yaml` | 原文档更新 | 保留既有场景，追加 contract-only / real-data / blocked artifact / runtime request 场景 | YAML revision section or top metadata | pending CP2 |
| `docs/product/TEST-MATRIX.md` | 原文档更新 | 保留既有测试矩阵，追加 CR160 review workflow checks | `## 修订记录` | pending CP2 |
| `docs/product/MVP-SCOPE.md` | 原文档更新 | 保留 DF-CR157-003，映射到 CR160 scope | `## 修订记录` | pending CP2 |
| `docs/product/BACKLOG.md` | 原文档更新 | 保留 BL-CR157-003 历史行，标记 promoted to CR160 | `## 修订记录` | pending CP2 |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| `DF-CR157-003` | `CR-160 Stage 4 observation review workflow` | 原文保留 + promoted mapping | Stage 4 review 从 deferred idea 转为正式 CR160。 |
| `BL-CR157-003` | `CR-160 Stage 4 observation review workflow` | 原文保留 + promoted mapping | Backlog P1 item 转为正式 CR。 |
| CR157 `observation_plan_ref` | CR160 observation plan content contract | 代码合同保留 | CR160 定义 ref 指向内容，不改 CR157 历史实现。 |
| CR157 `stage4_observation_gate_approved` | CR160 gate approval decision table | 代码合同保留 | CR160 定义 unlock condition 的可审计语义。 |
| CR155 artifact package | CR160 `blocked_admission_failed` sample | 原 evidence 保留 | CR155 不重开、不晋级，只作为 fail-closed 样例。 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增 / 修改产品范围或需求基线 | Stage 4 observation review workflow | true | CP2 前产品澄清；更新 USE-CASES / REQUIREMENTS / MVP-SCOPE / BACKLOG。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | contract-only、real-data、blocked admission、runtime request 场景 | true | 更新 SCENARIOS / TEST-MATRIX。 |
| 计划层 | 是否改变 Phase / Story / gate 路由 | CP2/CP3/CP7/CP8 route；CP4/CP5/CP6 条件路由 | true | CP2 决定纯设计 WAIVED / N/A 还是代码路径升级。 |
| 安全层 | 是否涉及 runtime / data / credential / trading 授权边界 | observation / simulation / paper / runtime gate | true | 明确不授权运行；后续另起 runtime/paper authorization CR。 |
| 交付层 | 是否新增交付物 | HLD、checklist、classification sample、release context | true | CP3/CP7/CP8 交付设计与验证证据。 |

## CP2 待确认决策

| Decision ID | 类型 | 问题 | 推荐方案 | 备选路径 | 影响 |
|---|---|---|---|---|---|
| `DQ-CP2-CR160-EVIDENCE-PROFILE` | scope | Observation review 输入支持 contract-level evidence、real-data evidence，还是双轨？ | 双轨 fail-closed：`contract_only` 只能输出低等级结论；`real_data_validated` 仍需 admission 非阻断才可进入 review。 | 只支持 contract-level；只支持 real-data。 | 决定 observation review 的输入资格和输出状态。 |
| `DQ-CP2-CR160-CR155-SEED-CLASSIFICATION` | scope | CR155 是否作为当前 observation candidate？ | 否。CR155 分类为 `blocked_admission_failed`，只作为 fail-closed 样例。 | 暂不使用 CR155 样例；或误用为候选（不推荐）。 | 防止把 `READY_WITH_RISK` 误读为 admission ready。 |
| `DQ-CP2-CR160-DELIVERABLE-SHAPE` | implementation | CR160 交付设计文档、代码，还是两者？ | 默认设计文档 + checklist + gate contract + classification evidence；轻量 schema/checker optional。 | 强制实现 schema/checker；只写需求不做 HLD。 | 决定是否触发 CP4/CP5/CP6 标准实现路径。 |
| `DQ-CP2-CR160-AUTHORIZATION-BOUNDARY` | security / runtime_authorization | 是否授权 simulation/paper/runtime/real lake 新操作？ | 否。只定义后续授权门禁。 | 单独 runtime authorization CR。 | 决定安全边界。 |
| `DQ-CP2-CR160-CP4-CP5-ROUTE` | workflow | 纯设计路径下如何处理 CP4/CP5？ | 纯设计：CP4/CP5 记录 N/A / WAIVED reason；含代码：route 升级并标准 CP4/CP5。 | 强制所有设计走 Story；或静默跳过（不允许）。 | 决定下游计划和 dispatch 证据要求。 |

## CR155 Seed Classification Baseline

CR155 当前事实必须在 CP2/CP3/CP7 中保持一致：

| 字段 | 值 |
|---|---|
| CR155 closure | `READY_WITH_RISK` for artifact scope |
| admission package status | `BLOCKED` |
| statistical gate status | `FAIL` |
| decision | `ARTIFACT_COMPLETE_ADMISSION_FAILED` |
| paper_candidate | `false` |
| rerun consistency | `PASS` |
| CR160 classification | `blocked_admission_failed` |

## 初始实施计划

| 阶段 | 目标 | 主要动作 | 出口 |
|---|---|---|---|
| CP0 | 请求受理 | 创建 CR160、route plan、CP0 result、context、summary、ledger/index/state refs | CP0 PASS |
| CP1/CP2 | 产品澄清 | 更新 product baseline；确认 5 个 CP2 决策 | CP2 人工批准 |
| CP3 | 设计 | 输出 Stage 4 observation review HLD / gate contract；checklist 必须分层覆盖 Stage 1 数据基础、Stage 2 研究生产、Stage 3 研究机门禁和横切授权 / no-overclaim；`observation_plan_ref` 必须区分 template 与 instance，并定义观察周期、检查频率、跟踪指标和退出条件 | CP3 人工批准 |
| CP4/CP5 | 条件路由 | 纯设计记录 N/A/WAIVED reason；含代码则 Story/LLD | route 与 CP2 一致 |
| CP6/CP7 | 交付与验证 | CP6 N/A；设计交付物作为 CP7 验证输入，由 meta-dev/meta-qa 或批准的 inline-fallback 审查 contract completeness / checklist testability / CR155 fail-closed | CP7 PASS/PASS_WITH_RISK |
| CP8 | 收尾 | 交付 release context，登记后续候选，不授权 runtime | CP8 approved / READY or READY_WITH_RISK |

## CP3 设计约束

| 约束 ID | 来源 | 要求 | 验收口径 |
|---|---|---|---|
| `CP3-FOCUS-CR160-005` | 用户全流程评审 | Checklist 不得只覆盖 Stage 3。必须分层覆盖 Stage 1（PIT / universe / lineage）、Stage 2（因子方法论 / typed_unavailable 处置）、Stage 3（统计门 / OOS / 经济显著性 / 容量 / 冲击 / rerun）和横切层（operation counters / authorization / no-overclaim）。 | `docs/quality/OBSERVATION-REVIEW-CHECKLIST.md` 有四层结构，CP7 可逐层复核。 |
| `CP3-FOCUS-CR160-006` | 用户全流程评审 | `observation_plan_ref` 必须拆分为 CR160 定义的 `observation_plan_template` 和 Stage 3 未来产出的 `observation_plan_instance`。 | HLD 明确 template / instance 的生成方、审查方、合规关系和缺失时 fail-closed 行为。 |
| `CP3-FOCUS-CR160-007` | 用户全流程评审 | Observation plan template 必须包含时间维度：观察周期、检查点频率、跟踪指标、退出条件、暂停 / 修复触发。 | Decision table 不允许缺少时间维度的 plan instance 通过 observation review。 |
| `CP3-FOCUS-CR160-008` | 用户全流程评审 | 审查主体必须明确：checklist 是人工审查工具，decision table 是审查后的分类框架。 | 不把 checklist 写成自动 runtime gate；自动 schema/checker 只作为后续 follow-up CR 候选。 |

## 风险

| 风险 ID | 等级 | 背景 | 缓解 |
|---|---|---|---|
| `R-CR160-READY-WITH-RISK-MISREAD` | high | 上游 CR 的 READY_WITH_RISK 可能被误读为 admission ready。 | CP2/CP3 decision table 明确 closure readiness 与 admission readiness 分离。 |
| `R-CR160-CR155-BLOCKED-ARTIFACT-PROMOTION` | high | CR155 有真实验证但 package `BLOCKED`、paper_candidate=false。 | CR155 只能作为 blocked sample，不可作为 observation candidate。 |
| `R-CR160-RUNTIME-AUTHORIZATION-LEAK` | high | Observation / simulation / paper 语义容易越界到 runtime。 | CR160 不授权任何 runtime；后续运行必须独立 CR。 |
| `R-CR160-GATE-PROFILE-NAME-MISMATCH` | medium | 用户评审使用 `standard`，当前 route planner 不支持该 literal profile。 | 机器使用 `standard-lite`，CR 记录 `route_strategy=standard-governance`；若必须 literal `standard`，需另起 tooling CR。 |

## 当前结论

CR160 已在 CP8 关闭为 `READY_WITH_RISK / delivered`。Stage 4 observation review workflow、分层 checklist、CR155 fail-closed sample、产品基线刷新和后续候选路由均已记录；本 CR 仍不授权代码实现、runtime/paper/simulation/live、交易、真实数据新读取、NAS/provider/credential 访问、Git remote write、deployment 或 publish。
