---
cr_id: "CR-161"
cr_type: "product-scope"
cr_kind: "requirement-change"
title: "Strategy Admission Evidence Pipeline Hardening"
lifecycle_status: "closed"
readiness_status: "ready_with_risk"
gate_status: "closed"
status: "closed"
gate_profile: "standard-lite"
route_strategy: "standard-governance; CP2 first, no Story/LLD/implementation before scope approval"
route_plan_ref: "process/checks/CP0-CR161.route-plan.json"
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
fast_lane_upgrade_reason: "admission semantics, product baseline, statistical evidence contracts, and no-overclaim boundaries require CP2/CP3/CP8 human gates"
rollback_to: "delivered / no active formal CR baseline after CR160 closure"
approval_result: "approved"
created_at: "2026-07-09T21:39:26+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-07-09T22:19:04+08:00"
source: "user"
linked_issue: ""
parent_cr: ""
source_checkpoint: "user request after CR160 closure"
source_decision_id: ""
follow_up_type: "strategy-admission-methodology-hardening"
risk_class: "product-scope-statistical-methodology-no-runtime"
owner: "host-orchestrator"
revisit_condition: "CP2 approve or user requests scope change; forward research-engine instrumentation requires a later CP2/CP3 decision or follow-up CR"
acceptance_criteria: "CP2 confirms contract-first evidence hardening, typed_unavailable fail-closed behavior, CR155 negative regression criteria, and no-runtime/no-new-real-data authorization boundary."
close_condition: "CP8 approves delivered evidence contracts, admission integration semantics, verification evidence, and follow-up tracking for research-engine instrumentation and economic/capacity extensions."
cr_index_path: "process/changes/CR-INDEX.json"
current_requirement_baseline_path: "process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml"
historical_baseline_status: "reframed"
reframed_by:
  - "CR-162"
reframe_summary: "CR162 refreshed the nine promised product and feature baseline documents after CR161 closure; CR161 design, CP7 evidence, waiver and CP8 READY_WITH_RISK decision remain historical."
goal_ref: "ROADMAP-QUANT-RESEARCH-PRODUCTION"
goal_statement: "Upgrade strategy admission from contract-visible evidence slots to computable or explicitly unavailable fail-closed evidence semantics."
user_goal_impact: "Prevents strategies from passing admission when trial lineage, multiple-testing evidence, OOS evidence, or economic/capacity evidence is missing; preserves CR155 as a blocked negative sample."
approval_focus: "scope|architecture|security|implementation|runtime|risk"
split_rationale: "The change cuts across CR151 statistical admission, CR154 reliability gates, CR155 blocked real-lake sample, and future strategy admission packages. It changes product/admission semantics rather than a single implementation task."
why_not_merge_with_parent: "CR151/CR154/CR155/CR160 are closed foundations or design workflows. CR161 changes the next methodology baseline and needs its own CP2 audit boundary."
why_not_story_or_task: "Before implementation, CP2 must settle evidence availability semantics, typed_unavailable behavior, CR155 negative-regression scope, and whether research-engine trial-lineage instrumentation is current scope or follow-up."
decision_burden: "high"
approve_effect: "Approve CP2 scope and route CR161 to CP3 design for contract-first admission evidence hardening. CP2 approval does not authorize source implementation or research-engine instrumentation unless separately selected."
reject_effect: "Admission remains dependent on existing CR151/CR154 slots and static/fixture semantics; missing trial lineage or multiple-testing evidence may stay as a planned but unscoped methodology gap."
not_authorized_by_approve:
  - "source_or_test_implementation_before_CP5_approval"
  - "research_engine_forward_instrumentation_unless_CP2_selects_it"
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
  - "external_framework_clone_install_run"
  - "git_remote_write"
  - "true_release_execution_or_publish"
product_baseline_refresh_required: true
required_evidence:
  - "admission_package"
  - "oos_walkforward"
  - "typed_unavailable_fail_closed"
  - "cr155_negative_regression"
required_phase: "documentation"
required_agent: "host-orchestrator"
required_gate: "CP8"
block_story_decomposition_until: "CP2-approved"
affected_product_docs:
  - "docs/product/USE-CASES.md"
  - "docs/product/REQUIREMENTS.md"
  - "docs/product/SCENARIOS.yaml"
  - "docs/product/TEST-MATRIX.md"
  - "docs/product/MVP-SCOPE.md"
  - "docs/product/BACKLOG.md"
  - "process/docs/features/factor-research-loop/DESIGN.md"
  - "process/docs/features/factor-research-loop/TEST-PLAN.md"
  - "process/docs/features/factor-research-loop/TASKS.md"
affected_use_cases:
  - "UC-58 multifactor strategy E2E"
  - "UC-59 machine learning strategy E2E"
  - "UC-60 event-driven strategy E2E"
routing_design_ref: "process/checks/CP0-CR161.route-plan.json"
summary_ref: "process/changes/summaries/CR-161.summary.json"
cp0_result_ref: "process/checks/CP0-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-REQUEST-INTAKE.result.json"
cp0_context_ref: "process/context/CP0-CR161.context.json"
cp1_result_ref: "process/checks/CP1-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-USE-CASE-COMPLETENESS.result.json"
cp2_result_ref: "process/checks/CP2-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-SCOPE.result.json"
cp2_context_ref: "process/context/CP2-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-SCOPE-CONTEXT.yaml"
cp2_checkpoint_ref: "process/checkpoints/CP2-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-SCOPE.md"
risk_refs:
  - "R-CR161-TRIAL-LINEAGE-SOURCE"
  - "R-CR161-CR155-HISTORICAL-EVIDENCE-GAP"
  - "R-CR161-STATIC-CONTRACT-MISREAD-AS-COMPUTED-PROOF"
  - "R-CR161-RESEARCH-ENGINE-INSTRUMENTATION-SCOPE-CREEP"
  - "R-CR161-RUNTIME-AUTHORIZATION-LEAK"
authz_policy_refs:
  - "NO_CREDENTIAL_READ"
  - "NO_RUNTIME"
  - "NO_PRODUCTION_WRITE"
  - "NO_TRADING"
  - "NO_REAL_LAKE_READ_OR_WRITE"
  - "NO_NAS_SYNC_OR_WRITE"
  - "NO_PROVIDER_FETCH"
  - "NO_BROKER_WRITE"
  - "NO_EXTERNAL_FRAMEWORK_RUN"
  - "NO_CATALOG_POINTER_WRITE"
---

# CR-161 Strategy Admission Evidence Pipeline Hardening

## 变更描述

用户要求按已确认计划启动新的 CR，补强策略 admission 的统计方法论和经济判别力。范围聚焦 C1-C4 blocker：

- C1 多重检验 / 数据偷窥门：当前已有 CR151 / CR154 slot 和 fail-closed 合同，但缺少从研究实验族到 FDR / WRC-SPA / PBO / DSR 证据的可计算或显式 unavailable 管线。
- C2 Walk-forward / OOS：当前已有 split / purge / embargo refs，但尚未作为一等 admission evidence stage 管理。
- C3 市场冲击模型：当前有 commission / tax / slippage / impact slot 和 no-real-TCA 边界，但没有可审计近似 impact / cost-underestimation evidence。
- C4 容量 / 流动性 sizing：当前有容量 / 流动性字段和 blocked-claim 语义，但没有策略级 capacity curve / alpha decay / capacity-adjusted return 证据。

根据用户对风险的补充，本 CR 的第一实施切片必须采用 **contract-first**：

- 不在 CP2 前改造研究引擎。
- 不从 CR155 历史 evidence 反推或伪造 trial lineage。
- 缺 trial count、parameter search lineage、p-values、fold-level metrics 时，输出 `typed_unavailable` 并 fail-closed。
- CR155 只作为 `blocked_admission_failed` negative regression seed；验收目标是保持 blocked / `paper_candidate=false`，不是要求历史 C1/C2 evidence 全部可计算。

## 目标影响摘要

| 字段 | 内容 |
|---|---|
| 目标引用 | `ROADMAP-QUANT-RESEARCH-PRODUCTION` |
| 整体目标 | 将策略 admission 从 contract-visible / fixture-visible 升级为 evidence-computable 或 typed-unavailable fail-closed 的判别管线。 |
| 用户目标影响 | 缺少 trial lineage、多重检验、OOS、成本或容量证据时，系统不会静默 PASS，也不会把 READY_WITH_RISK / rerun consistency 误读为 paper candidate。 |
| 本 CR 为什么值得独立推进 | 它改变 admission 语义、产品基线、证据 contract 和后续研究引擎边界，跨 CR151/154/155/160，不是单个 Story。 |
| approve 后会发生什么 | 进入 CP3 设计，定义 evidence contracts、availability model、typed_unavailable fail-closed rules、CR155 negative regression、后续 instrumentation follow-up。 |
| reject / 不确认会阻塞什么 | admission 仍停留在现有 slots / refs / fixture 语义，C1-C4 blocker 继续作为生产级量化框架的核心缺口。 |
| 决策负担 | `high`：必须明确当前切片不做研究引擎 instrumentation，不修 CR155，不授权 runtime / real data。 |

## 拆分理由

| 问题 | 结论 |
|---|---|
| 为什么不合并到 parent / active CR | 当前无 active formal CR；CR151/154/155/160 已关闭并提供基础合同 / 样例，但 CR161 是新的方法论基线。 |
| 为什么不是 Story / task / follow-up | CP2 前必须先决定 trial lineage 来源、typed_unavailable 语义、CR155 验收口径、是否改造研究引擎和授权边界。 |
| 触发独立 CR 的边界 | strategy admission evidence pipeline、统计可靠性、OOS evidence、economic/capacity evidence、negative regression、research-engine instrumentation follow-up。 |

## CR 类型与门禁策略

| 字段 | 内容 |
|---|---|
| CR 类型 | `product-scope` |
| Legacy CR kind | `requirement-change` |
| 生命周期状态 | `active` |
| 就绪状态 | `not_ready` |
| 门禁状态 | `cp2_pending` |
| 门禁模板 | `standard-lite`（保留 standard-governance 语义；当前 route planner 历史上接受该 profile） |
| Route plan | `process/checks/CP0-CR161.route-plan.json` |

### CR Trait 与实际路由

| trait 字段 | 值 | 路由含义 |
|---|---|---|
| uses existing evidence only | `false` | 需要新增 evidence contracts 和后续设计；但 CR155 negative regression 只消费既有 evidence。 |
| has new design | `true` | CP3 必须设计 contracts、availability semantics、decision table 和 follow-up boundaries。 |
| has new implementation | `false` | CP2 默认不授权实现；若用户选择当前 CR 包含代码实现，必须重算 CP4/CP5/CP6。 |
| has new verification | `true` | CP7 验证 typed_unavailable fail-closed、CR155 negative regression 和 no-overclaim wording。 |
| requires architecture review | `true` | 影响 CR151 / CR154 / CR155 admission chain 和跨策略证据模型。 |
| requires story decomposition | `false` | 默认设计-first；CP2 可选择后续实现路线，但 CP2 前不得拆 Story。 |
| requires subagent dispatch | `true` | CP3/CP7 应有 meta-se / meta-qa 调度证据或批准的 inline-fallback。 |

## Checkpoint Index

| CP | 状态 | 机器结果 ref | 人工门禁 ref | Context ref | Ledger event ref | 摘要 |
|---|---|---|---|---|---|---|
| CP0 | pass | `process/checks/CP0-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-REQUEST-INTAKE.result.json` | N/A | `process/context/CP0-CR161.context.json` | `process/state/CHECKPOINT-LEDGER.ndjson` | 请求已受理；无 active / blocked formal CR 冲突；CR161 编号确认。 |
| CP1 | pass | `process/checks/CP1-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-USE-CASE-COMPLETENESS.result.json` | N/A | `process/context/CP2-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-SCOPE-CONTEXT.yaml` | `process/state/CHECKPOINT-LEDGER.ndjson` | C1-C4、CR155 negative regression、typed_unavailable 场景覆盖已形成 CP2 输入。 |
| CP2 | approved | `process/checks/CP2-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-SCOPE.result.json` | `process/checkpoints/CP2-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-SCOPE.md` | `process/context/CP2-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-SCOPE-CONTEXT.yaml` | `process/state/GATE-LEDGER.ndjson` | 用户已确认 contract-first、typed_unavailable fail-closed、CR155 negative regression、follow-up split 与不授权边界。 |
| CP3 | approved | `process/checks/CP3-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-HLD-CONSISTENCY.result.json` | `process/checkpoints/CP3-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-HLD-REVIEW.md` | `process/context/CP3-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-DESIGN-CONTEXT.yaml` | `process/state/GATE-LEDGER.ndjson` | 用户已批准 seven-object evidence contracts、claim-tier fail-closed semantics、CR151/154 integration 与 CR155 negative regression。 |
| CP4 | n/a-by-route | `process/checks/CP4-CR161-STORY-DAG-PARALLEL-SAFETY.result.json` | N/A | N/A | `process/state/CHECKPOINT-LEDGER.ndjson` | 批准的 design-only route 无 Story decomposition。 |
| CP5 | n/a-by-route | `process/checks/CP5-CR161-LLD-DESIGN-EVIDENCE.result.json` | N/A | N/A | `process/state/CHECKPOINT-LEDGER.ndjson` | 批准的 design-only route 无 LLD batch。 |
| CP6 | n/a-by-route | `process/checks/CP6-CR161-IMPLEMENTATION-DONE.result.json` | N/A | N/A | `process/state/CHECKPOINT-LEDGER.ndjson` | 当前 CR 未授权实现。 |
| CP7 | pass-with-risk | `process/checks/CP7-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-VERIFICATION.result.json` | N/A | `process/context/CP7-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-VERIFICATION-CONTEXT.yaml` | `process/state/CHECKPOINT-LEDGER.ndjson` | 静态 design-contract 验证通过；独立 QA 未完成，风险显式带入 CP8。 |
| CP8 | approved | `process/checks/CP8-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-RELEASE-READINESS.result.json` | `process/checkpoints/CP8-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-RELEASE-READINESS.md` | `process/context/CP8-CR161-STRATEGY-ADMISSION-EVIDENCE-PIPELINE-HARDENING-CONTEXT.yaml` | `process/state/GATE-LEDGER.ndjson` | 用户批准 `READY_WITH_RISK`；独立性例外仅以到期 waiver 接受，后续高风险工作必须独立复核。 |

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
```

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `docs/product/USE-CASES.md` | 原文档更新 | 既有场景保留；新增 CR161 strategy admission evidence pipeline 场景与 negative regression 场景。 | `## 修订记录` | pending CP2 |
| `docs/product/REQUIREMENTS.md` | 原文档更新 | 既有 REQ 保留；新增 CR161 evidence contract / typed_unavailable / fail-closed / no-overclaim requirements。 | `## 修订记录` | pending CP2 |
| `docs/product/SCENARIOS.yaml` | 原文档更新 | 既有 scenarios 保留；新增 trial lineage missing、CR155 historical evidence gap、typed_unavailable negative cases。 | `revision_history` / 等价修订记录 | pending CP2 |
| `docs/product/TEST-MATRIX.md` | 原文档更新 | 既有矩阵保留；新增 C1-C4 evidence availability 和 CR155 negative regression 覆盖。 | `## 修订记录` | pending CP2 |
| `docs/product/MVP-SCOPE.md` | 原文档更新 | 既有 MVP 保留；明确 CR161 current slice 是 contract-first，不含 engine instrumentation。 | `## 修订记录` | pending CP2 |
| `docs/product/BACKLOG.md` | 原文档更新 | 既有 backlog 保留；新增 follow-up：research-engine trial-lineage instrumentation、impact/capacity implementation。 | `## 修订记录` | pending CP2 |
| `process/docs/features/factor-research-loop/DESIGN.md` | 原文档更新 | 既有 CR151 / factor-research-loop 设计保留；新增 CR161 evidence pipeline 设计引用。 | 修订记录 | pending CP2 |
| `process/docs/features/factor-research-loop/TEST-PLAN.md` | 原文档更新 | 既有测试计划保留；新增 typed_unavailable / fail-closed / CR155 regression 验证。 | 修订记录 | pending CP2 |
| `process/docs/features/factor-research-loop/TASKS.md` | 原文档更新 | 既有任务保留；新增 CR161 后续任务 / follow-up，不抢占 CP2 前实现。 | 修订记录 | pending CP2 |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| CR151 `StrategyAdmissionStatisticalGate` | CR161 evidence availability / typed_unavailable inputs | 原文保留并扩展 | CR161 不替换 CR151 gate；它定义上游 evidence 生成 / 不可得语义。 |
| CR154 Gate 1 / Gate 2 / Gate 4 | CR161 evidence contracts 和 fail-closed policy | 原文保留并扩展 | CR161 约束 CR154 slots 的 evidence quality，不把 slot 存在误读为 proof。 |
| CR155 blocked sample | CR161 negative regression seed | 原 evidence 保留 | CR155 仍 `blocked_admission_failed`，不要求历史 C1/C2 全部可计算。 |
| CR160 observation review | CR161 admission evidence prerequisite | 原设计保留 | CR161 产生更严格的 Stage 3 admission evidence semantics，供未来 observation review 消费。 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增、删除或重定义 REQ-* | `docs/product/REQUIREMENTS.md`、factor research feature docs | true | CP2 通过后增量新增 evidence availability / typed_unavailable / no-overclaim requirements。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | `SCENARIOS.yaml` / `TEST-MATRIX.md` | true | 新增 C1/C2 missing evidence、CR155 negative regression、C3/C4 deferred boundary 场景。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | future development plan / route plan | true | 当前只到 CP2；CP2 决定 CP3 后是否设计-only 或进入 Story planning。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | authorization boundary | false for current slice | 当前不授权 runtime、new real lake、NAS/provider、credential、broker、external framework；任何需要另起授权门。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | HLD / quality / release docs | true | CP3/CP7/CP8 需要生成并验证 design/evidence docs；代码实现另行授权。 |

## 回退决策

- 影响范围：全局 admission semantics，但当前只写 process/product/design 范围。
- 回退到阶段：`delivered / CR160 closed baseline`
- 需要重新确认的对象：CP2 decisions、product baseline docs、future CR161 HLD / route plan。

## 产品基线重整门禁

- 是否需要产品基线重整：`true`
- 必须回到阶段：`requirement-clarification`
- 责任 Agent：`host-orchestrator`
- 必须通过门禁：`CP2`
- Story / LLD / 实现阻断条件：`CP2-approved`
- 受影响产品文档：见 frontmatter `affected_product_docs`
- 受影响 use case：UC-58 / UC-59 / UC-60
- 分流设计引用：`process/checks/CP0-CR161.route-plan.json`

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | false | 改 admission 语义和产品基线。 |
| 修改架构、权限、安全边界或平台安装路径 | true | 改证据合同、安全边界、no-overclaim 语义。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | true | 可能影响 CR151 / CR154 / admission package contracts。 |
| 需要 HLD / LLD 才能解释影响 | true | CP3 必须设计 evidence model 和 route。 |

结论：`workflow_mode_after_change=standard`。

## 风险与开放项

| ID | 状态 | 风险 | 当前处理 |
|---|---|---|---|
| R-CR161-TRIAL-LINEAGE-SOURCE | open | 研究引擎当前不记录 trial count / parameter search lineage。 | CP2 推荐 contract-first；缺失输出 `typed_unavailable` 并 fail-closed。 |
| R-CR161-CR155-HISTORICAL-EVIDENCE-GAP | open | CR155 packaged evidence 缺 p-values / trial lineage / PBO / DSR 原始输入。 | negative regression 不要求历史 C1/C2 全部可计算，只要求保持 blocked。 |
| R-CR161-STATIC-CONTRACT-MISREAD-AS-COMPUTED-PROOF | open | contract 存在可能被误读为 evidence computed。 | 引入 availability status 和 `typed_unavailable` ceiling。 |
| R-CR161-RESEARCH-ENGINE-INSTRUMENTATION-SCOPE-CREEP | open | 自动 trial lineage instrumentation 会扩大到研究引擎改造。 | 作为 follow-up / later wave，不进当前最小实现切片。 |
| R-CR161-RUNTIME-AUTHORIZATION-LEAK | open | admission hardening 可能被误读为 paper/simulation/runtime 授权。 | not_authorized_by_approve 全量阻断。 |

## CP2 待确认摘要

推荐回复：`approve`

如果用户回复 `approve`，表示接受：

1. Wave 1 采用 contract-first，不改研究引擎，缺 trial lineage 输出 `typed_unavailable`。
2. `typed_unavailable` 对 statistical significance / robustness / paper candidate 必须 fail-closed，不得 silent PASS。
3. CR155 是 negative regression seed，验收目标是仍 blocked / `paper_candidate=false`，不是补齐历史 C1/C2。
4. research-engine forward instrumentation、完整 impact/capacity implementation 进入 follow-up / later wave。
5. 本 CP2 不授权代码实现、真实数据访问、runtime、broker、paper/simulation/live 或 publish。
