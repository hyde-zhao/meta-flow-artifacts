---
cr_id: CR-158
cr_type: product-scope
cr_kind: requirement-change
title: Event + ML Strategy Adapter Unified Implementation
lifecycle_status: closed-current-delivery
readiness_status: ready_with_risk
gate_status: closed
status: closed-current-delivery-ready-with-risk
gate_profile: architecture-major
route_plan_ref: process/checks/CP0-CR158.route-plan.json
cr_trait_uses_existing_evidence_only: false
cr_trait_has_new_design: true
cr_trait_has_new_implementation: true
cr_trait_has_new_verification: true
cr_trait_requires_architecture_review: true
cr_trait_requires_story_decomposition: true
cr_trait_requires_subagent_dispatch: true
cr_trait_verification_waiver_reason: ''
cr_trait_verification_waiver_ref: ''
impact_level: high
workflow_mode_before: standard
workflow_mode_after_change: standard
fast_lane_upgrade_reason: New event + ML adapter scope changes product baseline, adapter
  contracts, implementation and verification; fast-lane is not appropriate.
rollback_to: delivered / no active formal CR baseline after CR157 closure
approval_result: cp8_approved_ready_with_risk
created_at: '2026-07-05T15:58:00+08:00'
created_by: host-orchestrator
approved_by: user
approved_at: '2026-07-05T16:55:00+08:00'
source: cp8-follow-up
linked_issue: DF-CR157-001, DF-CR157-002
parent_cr: CR-157
source_checkpoint: process/checkpoints/CP8-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-RELEASE-READINESS.md
source_decision_id: DQ-CP8-CR157-002
follow_up_type: deferred-adapter-implementation
risk_class: medium
owner: host-orchestrator
revisit_condition: CR157 CP8 is closed and the user explicitly starts the deferred
  event + ML strategy adapter scope as one CR.
acceptance_criteria: CP2 confirms unified event + ML adapter scope and authorization
  boundary; CP3/CP5 approve adapter contract and Story design batch; CP6/CP7 prove
  local/static/fixture implementation and verification without runtime or publish.
close_condition: All approved CR158 Story implementation and verification evidence
  passes CP6/CP7, and CP8 records release readiness or explicit closure decision.
cr_index_path: process/changes/CR-INDEX.json
current_requirement_baseline_path: process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml
historical_baseline_status: active
reframed_by: []
reframe_summary: ''
goal_ref: EVENT-ML-STRATEGY-ADAPTER-UNIFICATION
goal_statement: Promote CR157 deferred event and ML strategy adapters into one unified
  adapter CR with shared contracts, implementation path and no-runtime validation
  boundary.
user_goal_impact: Turns two deferred CR157 backlog items into a single governed delivery
  stream so event and ML strategy adapter semantics are designed together instead
  of diverging.
approval_focus: scope
split_rationale: The event adapter and ML adapter share adapter contract, evidence
  index, handoff and no-runtime validation concerns; one CR avoids duplicated CP2/CP3/CP5
  gates and conflicting schema decisions.
why_not_merge_with_parent: CR157 is closed as READY_WITH_RISK and explicitly deferred
  adapter implementation out of its first slice; reopening CR157 would blur the approved
  closure boundary.
why_not_story_or_task: The adapter work changes product scope, architecture contracts,
  Story decomposition and verification matrix; it needs CP2 before HLD/Story/LLD/implementation.
decision_burden: medium
approve_effect: 'Current gate target is CP5: approving the future CP5 LLD batch would
  authorize CP6 local/static/fixture implementation only; runtime, real data, registry,
  publish and trading remain blocked.'
reject_effect: If the future CP5 LLD batch is not approved, CR158 remains in design
  rework; no adapter implementation, tests implementation, runtime or release claim
  starts.
not_authorized_by_approve:
- real event feed or live listener
- real ML model training, external model service call, model registry write or promotion
- real lake/NAS/provider access or credential/env/session read
- catalog/store/registry/model/prediction write
- QMT/MiniQMT/xtquant/gateway runtime
- simulation/paper/live/trading runtime or broker operation
- external framework clone/install/run
- Git remote write
- true release execution, publish, production deployment or runtime readiness claim
product_baseline_refresh_required: true
product_baseline_refresh_status: approved_cp2
required_phase: delivered
required_agent: host-orchestrator
required_gate: none
block_story_decomposition_until: completed_cp3_approved
block_implementation_until: completed_cp5_approved
affected_product_docs:
- docs/product/USE-CASES.md
- docs/product/REQUIREMENTS.md
- docs/product/SCENARIOS.yaml
- docs/product/TEST-MATRIX.md
- docs/product/STORY-MAP.md
- docs/product/MVP-SCOPE.md
- docs/product/RELEASE-SLICES.md
- docs/product/BACKLOG.md
- docs/design/HLD-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md
- docs/design/ARCHITECTURE-DECISION-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md
- docs/components/MULTIFACTOR-RESEARCH.md
affected_use_cases:
- UC-58-CR157
impact_surface:
- product-baseline
- requirements
- scenarios
- test-matrix
- story-map
- mvp-scope
- adapter-contract
- event-strategy
- ml-strategy
- evidence-index
- stage2-stage3-handoff
conflict_keys:
- strategy_adapter_contract
- event_strategy_adapter
- ml_strategy_adapter
- stage2_multifactor_framework
- research_evidence_index
- no_runtime_no_publish_boundary
authz_policy_refs:
- NO_CREDENTIAL_READ
- NO_RUNTIME
- NO_PRODUCTION_WRITE
- NO_TRADING
- NO_REAL_LAKE_WRITE
- NO_NAS_SYNC_OR_WRITE
- NO_PROVIDER_FETCH
- NO_BROKER_WRITE
- NO_EXTERNAL_FRAMEWORK_RUN
- NO_CATALOG_POINTER_WRITE
risk_refs:
- R-CR158-ADAPTER-CONTRACT-COUPLING
- R-CR158-EVIDENCE-SEMANTIC-OVERFIT
- R-CR158-RUNTIME-OVERCLAIM
routing_design_ref: process/changes/CR-157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-2026-07-05.md#CP8-Follow-up-来源
summary_ref: process/changes/summaries/CR-158.summary.json
cp0_result_ref: process/checks/CP0-CR158-EVENT-ML-STRATEGY-ADAPTER-REQUEST-INTAKE.result.json
cp0_context_ref: process/context/CP0-CR158.context.json
cp1_result_ref: process/checks/CP1-CR158-EVENT-ML-STRATEGY-ADAPTER-USE-CASE-COMPLETENESS.result.json
cp2_result_ref: process/checks/CP2-CR158-EVENT-ML-STRATEGY-ADAPTER-SCOPE.result.json
cp2_context_ref: process/context/CP2-CR158-EVENT-ML-STRATEGY-ADAPTER-CONTEXT.yaml
cp2_checkpoint_ref: process/checkpoints/CP2-CR158-EVENT-ML-STRATEGY-ADAPTER-SCOPE.md
cp2_approved_by: user
cp2_approved_at: '2026-07-05T16:55:00+08:00'
cp3_context_ref: process/context/CP3-CR158-EVENT-ML-STRATEGY-ADAPTER-DESIGN-CONTEXT.yaml
cp3_result_ref: process/checks/CP3-CR158-EVENT-ML-STRATEGY-ADAPTER-HLD-CONSISTENCY.result.json
cp3_checkpoint_ref: process/checkpoints/CP3-CR158-EVENT-ML-STRATEGY-ADAPTER-HLD-REVIEW.md
cp3_hld_ref: docs/design/HLD-EVENT-ML-STRATEGY-ADAPTER.md
cp3_adr_ref: docs/design/ARCHITECTURE-DECISION-EVENT-ML-STRATEGY-ADAPTER.md
cp3_approved_by: user
cp3_approved_at: '2026-07-05T17:35:00+08:00'
cp4_context_ref: process/context/CP4-CR158-EVENT-ML-STRATEGY-ADAPTER-STORY-PLANNING-CONTEXT.yaml
cp4_result_ref: process/checks/CP4-CR158-EVENT-ML-STRATEGY-ADAPTER-STORY-DAG-PARALLEL-SAFETY.result.json
cp5_context_ref: process/context/CP5-CR158-EVENT-ML-STRATEGY-ADAPTER-LLD-CONTEXT.yaml
development_plan_ref: process/DEVELOPMENT-PLAN-CR158-EVENT-ML-STRATEGY-ADAPTER.yaml
story_status_ref: process/STORY-STATUS-CR158-EVENT-ML-STRATEGY-ADAPTER.md
cp5_result_ref: process/checks/CP5-CR158-EVENT-ML-STRATEGY-ADAPTER-LLD-IMPLEMENTABILITY.result.json
cp5_checkpoint_ref: process/checkpoints/CP5-CR158-EVENT-ML-STRATEGY-ADAPTER-LLD-BATCH.md
cp5_launch_message_ref: process/checkpoints/CP5-CR158-EVENT-ML-STRATEGY-ADAPTER-LLD-BATCH.launch.md
cp5_opened_at: '2026-07-05T17:55:00+08:00'
release_decision: READY_WITH_RISK
closed_at: '2026-07-05T19:20:00+08:00'
closed_by: user
cp6_result_ref: process/checks/CP6-CR158-EVENT-ML-STRATEGY-ADAPTER-IMPLEMENTATION.result.json
cp7_result_ref: process/checks/CP7-CR158-EVENT-ML-STRATEGY-ADAPTER-VERIFICATION.result.json
cp8_context_ref: process/context/CP8-CR158-EVENT-ML-STRATEGY-ADAPTER-RELEASE-CONTEXT.yaml
cp8_result_ref: process/checks/CP8-CR158-EVENT-ML-STRATEGY-ADAPTER-RELEASE-READINESS.result.json
cp8_checkpoint_ref: process/checkpoints/CP8-CR158-EVENT-ML-STRATEGY-ADAPTER-RELEASE-READINESS.md
cp8_approved_by: user
cp8_approved_at: '2026-07-05T19:20:00+08:00'
release_context_ref: process/release/RELEASE-CONTEXT-CR158-EVENT-ML-STRATEGY-ADAPTER.yaml
post_closure_audit_ref: process/checks/CR158-METAFLOW-PROCESS-SELF-AUDIT.md
token_retrospective_ref: process/checks/CR158-TOKEN-RETROSPECTIVE.md
post_closure_audit_status: complete
post_closure_audit_at: '2026-07-05T19:40:00+08:00'
---

# CR-158 Event + ML Strategy Adapter Unified Implementation

## 变更描述

用户请求：“启动 event + ML strategy adapter CR，把 DF-CR157-001 和 DF-CR157-002 合并为一个 CR”。

本 CR 将 CR157 已确认延后的两个 follow-up 统一转为一个正式 CR：

| Deferred ID | 原标题 | 来源 | 本 CR 处理 |
|---|---|---|---|
| `DF-CR157-001` | Event strategy adapter implementation | `SGQ-CR157-001` / CR157 CP8 deferred follow-up | 合并进入 CR158 统一 adapter scope。 |
| `DF-CR157-002` | ML strategy adapter implementation | `SGQ-CR157-001` / CR157 CP8 deferred follow-up | 合并进入 CR158 统一 adapter scope。 |

CR158 的目标不是立即实现单个 adapter task，而是先确认 event + ML strategy adapter 的统一产品范围、共享 adapter contract、证据索引语义、Stage 2/Stage 3 handoff 影响、Story 拆分和 no-runtime 验证边界。CP2 未批准前，不得启动 HLD、Story decomposition、LLD 或实现。

## 目标影响摘要

| 字段 | 内容 |
|---|---|
| 目标引用 | `EVENT-ML-STRATEGY-ADAPTER-UNIFICATION` |
| 整体目标 | 将 CR157 延后的 event adapter 与 ML adapter 合并为一个统一 adapter CR，先确认产品范围和架构契约，再进入 Story/LLD/实现。 |
| 用户目标影响 | 避免 event / ML 两条 adapter 线分别推进时产生证据 schema、handoff、validation 和 no-runtime 边界分裂。 |
| 本 CR 为什么值得独立推进 | CR157 已关闭且明确把 adapter implementation 延后；当前无 active formal CR，两个 deferred item 共享目标、审批人、风险边界、交付节奏和回滚策略。 |
| approve 后会发生什么 | CP2 approve 后进入 solution-design，由 meta-se 输出 HLD/ADR 和 Story decomposition；CP5 全量设计证据通过前不允许实现。 |
| reject / 不确认会阻塞什么 | 两个 deferred adapter 仍保留为后续候选；不会改变 CR157 已关闭状态。 |
| 决策负担 | `medium`：需确认统一 scope、共同 contract 和不授权边界。 |

## 拆分理由

| 问题 | 结论 |
|---|---|
| 为什么不合并到 parent / active CR | CR157 已 CP8 关闭，且 `DF-CR157-001/002` 是 CR157 明确延期项；当前无 active / blocked formal CR。 |
| 为什么不是 Story / task / follow-up | event + ML adapter 会影响产品基线、接口契约、证据索引和验证矩阵，必须先过 CP2/CP3/CP5。 |
| 触发独立 CR 的边界 | 同一目标、同一 user journey、共享 HLD、同一 release value、相同 no-runtime 授权边界；因此合并成一个 CR，而不是拆两个 CR。 |

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-157` |
| 来源检查点 | `process/checkpoints/CP8-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-RELEASE-READINESS.md` |
| 来源决策 ID | `DQ-CP8-CR157-002` / deferred adapter follow-up |
| follow-up 类型 | deferred adapter implementation |
| 风险等级 | medium |
| owner | host-orchestrator |
| 重访条件 | CR157 已关闭，用户明确启动 event + ML adapter 统一 CR。 |
| 验收标准 | CP2/CP3/CP5 明确 adapter 范围、共享 contract、Story batch、verification matrix 和 no-runtime boundary。 |
| 关闭条件 | CP6/CP7/CP8 完成 approved scope 的本地实现、验证和交付就绪，或用户明确取消 / supersede。 |

## CR 类型与门禁策略

| 字段 | 内容 |
|---|---|
| CR 类型 | `product-scope` |
| Legacy CR kind | `requirement-change` |
| 生命周期状态 | `active` |
| 就绪状态 | `not_ready` |
| 门禁状态 | `cp5_pending` |
| 门禁模板 | `architecture-major` |
| Route plan | `process/checks/CP0-CR158.route-plan.json` |

### CR Trait 与实际路由

| trait 字段 | 值 | 路由含义 |
|---|---|---|
| uses existing evidence only | false | 需要新增设计、实现和验证。 |
| has new design | true | CP3 / CP4 / CP5 适用。 |
| has new implementation | true | CP6 适用。 |
| has new verification | true | CP7 适用。 |
| requires architecture review | true | CP3 必须 standard + human gate。 |
| requires story decomposition | true | CP4 / CP5 适用。 |
| requires subagent dispatch | true | CP2 后需按阶段调度 meta-pm / meta-se / meta-dev / meta-qa，或记录经批准的 inline fallback。 |

## Checkpoint Index

| CP | 状态 | 机器结果 ref | 人工门禁 ref | Context ref | Ledger event ref | 摘要 |
|---|---|---|---|---|---|---|
| CP0 | pass | `process/checks/CP0-CR158-EVENT-ML-STRATEGY-ADAPTER-REQUEST-INTAKE.result.json` | N/A | `process/context/CP0-CR158.context.json` | `process/state/CHECKPOINT-LEDGER.ndjson` | CR intake 完成；两个 DF item 合并为一个正式 CR。 |
| CP1 | pass | `process/checks/CP1-CR158-EVENT-ML-STRATEGY-ADAPTER-USE-CASE-COMPLETENESS.result.json` | N/A | `process/context/CP2-CR158-EVENT-ML-STRATEGY-ADAPTER-CONTEXT.yaml` | `process/state/CHECKPOINT-LEDGER.ndjson` | CR158 增量场景 / 需求 / TEST-MATRIX / MVP scope 已更新，自动检查通过。 |
| CP2 | approved | `process/checks/CP2-CR158-EVENT-ML-STRATEGY-ADAPTER-SCOPE.result.json` | `process/checkpoints/CP2-CR158-EVENT-ML-STRATEGY-ADAPTER-SCOPE.md` | `process/context/CP2-CR158-EVENT-ML-STRATEGY-ADAPTER-CONTEXT.yaml` | `process/state/GATE-LEDGER.ndjson` | 用户已批准统一 scope、authorization boundary 和 gate sequence；不授权 runtime / real data / publish / implementation before CP5。 |
| CP3 | approved | `process/checks/CP3-CR158-EVENT-ML-STRATEGY-ADAPTER-HLD-CONSISTENCY.result.json` | `process/checkpoints/CP3-CR158-EVENT-ML-STRATEGY-ADAPTER-HLD-REVIEW.md` | `process/context/CP3-CR158-EVENT-ML-STRATEGY-ADAPTER-DESIGN-CONTEXT.yaml` | `process/state/GATE-LEDGER.ndjson` | 用户已接受 thin shared core + typed extensions、refs-only evidence 和 no-runtime guard 推荐决策；不授权 runtime / real data / publish / implementation before CP5。 |
| CP4 | pass | `process/checks/CP4-CR158-EVENT-ML-STRATEGY-ADAPTER-STORY-DAG-PARALLEL-SAFETY.result.json` | N/A | `process/context/CP4-CR158-EVENT-ML-STRATEGY-ADAPTER-STORY-PLANNING-CONTEXT.yaml` | `process/state/CHECKPOINT-LEDGER.ndjson` | Story DAG、Feature Design Matrix、场景覆盖和并行安全自动预检通过；汇入 CP5。 |
| CP5 | approved | `process/checks/CP5-CR158-EVENT-ML-STRATEGY-ADAPTER-LLD-IMPLEMENTABILITY.result.json` | `process/checkpoints/CP5-CR158-EVENT-ML-STRATEGY-ADAPTER-LLD-BATCH.md` | `process/context/CP5-CR158-EVENT-ML-STRATEGY-ADAPTER-LLD-CONTEXT.yaml` | `process/state/GATE-LEDGER.ndjson` | CP5 自动预检 PASS，人工门禁已发起；全量 Story design evidence 待用户统一确认。 |
| CP6 | PASS | `process/checks/CP6-CR158-EVENT-ML-STRATEGY-ADAPTER-IMPLEMENTATION.result.json` | N/A | `process/context/stories/*.CP6.work-packet.json` | `process/state/CHECKPOINT-LEDGER.ndjson` | CP5 通过后才可实现。 |
| CP7 | PASS_WITH_RISK | `process/checks/CP7-CR158-EVENT-ML-STRATEGY-ADAPTER-VERIFICATION.result.json` | N/A | `process/context/stories/*.CP7.verify-packet.json` | `process/state/CHECKPOINT-LEDGER.ndjson` | 必须证明 local/static/fixture no-runtime boundary。 |
| CP8 | approved / READY_WITH_RISK | `process/checks/CP8-CR158-EVENT-ML-STRATEGY-ADAPTER-RELEASE-READINESS.result.json` | `process/checkpoints/CP8-CR158-EVENT-ML-STRATEGY-ADAPTER-RELEASE-READINESS.md` | `process/context/CP8-CR158-EVENT-ML-STRATEGY-ADAPTER-RELEASE-CONTEXT.yaml` | `process/state/GATE-LEDGER.ndjson` | 发布就绪 / ready-with-risk / closure 决策。 |

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
| `docs/product/USE-CASES.md` | 原文档更新 | 保留 CR157 `Deferred Ideas` 历史行，追加 CR158 启动/合并映射 | `## 修订记录` | approved CP2 |
| `docs/product/REQUIREMENTS.md` | 原文档更新 | 保留 REQ-CR157-006 作为历史 deferred boundary，新增 CR158 requirements | `## 修订记录` | approved CP2 |
| `docs/product/SCENARIOS.yaml` | 原文档更新 | 保留 CR157 no-runtime first-slice scenarios，新增 CR158 adapter positive/negative/boundary scenarios | `revision_history` | approved CP2 |
| `docs/product/TEST-MATRIX.md` | 原文档更新 | 保留 CR157 static evidence matrix，新增 CR158 adapter coverage rows | `## 修订记录` | approved CP2 |
| `docs/product/STORY-MAP.md` | 原文档更新 | 保留 CR157 first slice，新增 CR158 adapter journey / Story candidates | `## 修订记录` | approved CP2 |
| `docs/product/MVP-SCOPE.md` | 原文档更新 | 保留 CR157 out-of-scope decision，明确 CR158 in-scope / out-of-scope | `## 修订记录` | approved CP2 |
| `docs/product/RELEASE-SLICES.md` | 原文档更新 | 保留 CR157 release slice，新增 CR158 candidate slices after CP2 | `## 修订记录` | approved CP2 |
| `docs/product/BACKLOG.md` | 原文档更新 | 将 `DF-CR157-001/002` 标记为 promoted to CR158，不删除旧追溯 | `## 修订记录` | approved CP2 |
| `docs/design/HLD-EVENT-ML-STRATEGY-ADAPTER.md` | 新增 CR 专属 HLD | 不覆盖 CR157 HLD；用 companion docs 引用 CR158 蓝图 / ADR | `## 修订记录` | approved CP3 |
| `docs/design/ARCHITECTURE-DECISION-EVENT-ML-STRATEGY-ADAPTER.md` | 新增 CR 专属 ADR | 不覆盖 CR157 ADR；接受 ADR-CR158-001..003 | `## 修订记录` | approved CP3 |
| `docs/design/BLUEPRINT.md` / `DOMAIN-MAP.md` / `DEPENDENCY-MAP.md` | 原文档更新 | 追加 CR158 增量小节，不替换项目级蓝图 | 对应 CR158 小节 | approved CP3 |
| `process/docs/design/FEATURE-DESIGN-MATRIX.md` | 原文档更新 | 追加 CR158 CP4 Story / lld_policy 增量小节 | `## 修订记录` | pass CP4 |
| `process/DEVELOPMENT-PLAN-CR158-EVENT-ML-STRATEGY-ADAPTER.yaml` | 新增 CR 专属计划 | 作为 CR158 Story 管理机器真相源，不替换全局 legacy 计划 | YAML frontmatter | pass CP4 |
| `process/stories/CR158-S*.md` | 新增 Story 卡片 | 六个 Story 均为 `lld-ready`，CP5 前不得实现 | Story frontmatter | pass CP4 / pending CP5 |
| `docs/components/MULTIFACTOR-RESEARCH.md` | 原文档更新 | 保留 CR157 Stage 2 framework wording，新增 adapter implementation status only after CP6/CP7 | `## 修订记录` | pending downstream |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| `DF-CR157-001` | `CR158 Event adapter scope` | 原文保留 + CR158 摘录映射 | 从 deferred idea promoted to active formal CR scope，不删除 CR157 历史。 |
| `DF-CR157-002` | `CR158 ML adapter scope` | 原文保留 + CR158 摘录映射 | 从 deferred idea promoted to active formal CR scope，不删除 CR157 历史。 |
| `REQ-CR157-006` | `REQ-CR158-*` | 历史需求保留 | CR157 的 “adapter backlog boundary” 仍表示当时 first slice 不包含实现；CR158 是后续正式启动。 |
| `SC-CR157-B01` | `SC-CR158-*` | 历史场景保留 | CR157 backlog review 场景不替代 CR158 adapter engineering scenarios。 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否新增、删除或重定义 REQ-* | `DF-CR157-001/002`, `REQ-CR157-006`, CR158 product docs | true | 回到 requirement-clarification，由 meta-pm 生成增量需求和 CP2 Decision Brief。 |
| 场景层 | 是否改变验证场景、测试矩阵或 MVP scope | event adapter scenario, ML adapter scenario, no-runtime negative scenario | true | 更新 SCENARIOS / TEST-MATRIX / MVP-SCOPE，CP2 前不得 Story split。 |
| 计划层 | 是否改变 Phase、Wave、Story / 文件所有权 | adapter contract, event path, ML path, evidence index, handoff | true | CP2 后由 meta-se 拆 Story；CP5 前不得实现。 |
| 安全层 | 是否引入真实数据、runtime、credential、publish 或 trading 权限 | no-runtime authz boundary | false for new authorization, true for boundary enforcement | 默认全部不授权；任何真实 runtime 或 provider access 需另起 runtime authorization gate。 |
| 交付层 | 是否新增 release、migration、rollback、verification evidence | CR158 release docs, CP6/CP7 evidence, rollback plan | true | CP8 前生成 release context / release notes / rollback / feedback，且不得 overclaim production readiness。 |

## 回退决策

- 影响范围：CR158 产品基线、adapter contract、Story plan、implementation / verification evidence and release readiness.
- 回退到阶段：`delivered / no active formal CR baseline after CR157 closure`.
- 回退方式：若 CP2 未通过，关闭或取消 CR158，保留 `DF-CR157-001/002` 为 deferred; 不改写 CR157 closure。
- 需要重新确认的对象：若用户要求只做 event 或只做 ML，应在 CP2 Decision Brief 中作为备选方案确认；若要求真实 runtime / provider / model registry / publish，必须新增 runtime authorization decision 或拆出独立 runtime CR。

## 产品基线重整门禁

- 是否需要产品基线重整：true。
- 必须回到阶段：`requirement-clarification`。
- 责任 Agent：`meta-pm`。
- 必须通过门禁：`CP2`。
- Story / LLD / 实现阻断条件：`block_story_decomposition_until=CP2-approved`; CP5 通过前不得实现。
- 受影响产品文档：见 frontmatter `affected_product_docs`。
- 受影响 use case：`UC-58-CR157` and CR158 incremental adapter use cases.

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | no | 涉及产品范围、adapter contract、implementation 和 verification。 |
| 修改架构、权限、安全边界或平台安装路径 | architecture yes / authz no | 修改架构契约；不扩大授权边界。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | yes | event + ML adapter likely produces multiple Stories and shared contract files。 |
| 需要 HLD / LLD 才能解释影响 | yes | 需要 CP3 HLD/ADR 和 CP5 Story design batch。 |
| 是否保持 fast-lane | no | 保持 `standard` workflow with `architecture-major` gate profile。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：true。
- batch_id：`CR158-EVENT-ML-STRATEGY-ADAPTER-BATCH-A`。
- 批次范围来源：CP2 product baseline + CP3 HLD / ADR + CP4 development plan。
- 批次内 Story：`CR158-S01-shared-adapter-core-contract`、`CR158-S02-event-strategy-adapter-extension`、`CR158-S03-ml-strategy-adapter-extension`、`CR158-S04-evidence-handoff-typed-refs`、`CR158-S05-no-runtime-guard-counters`、`CR158-S06-verification-release-boundary`。
- 统一确认门禁：全部 Story 设计证据和 CP4 自动预检完成后，统一进入 CP5 人工确认；不得逐 Story 确认后逐 Story 开发。

## 初始风险

| 风险 ID | 严重度 | 状态 | 描述 | 缓解 |
|---|---|---|---|---|
| `R-CR158-ADAPTER-CONTRACT-COUPLING` | medium | open | event and ML adapter semantics may be forced into an over-general contract. | CP2/CP3 must decide shared core vs type-specific extension points. |
| `R-CR158-EVIDENCE-SEMANTIC-OVERFIT` | medium | open | Evidence index may overfit event or ML semantics and break existing CR157 refs-only contract. | Keep CR157 refs-only baseline; add typed extensions only after CP3 ADR. |
| `R-CR158-RUNTIME-OVERCLAIM` | high | open | Adapter implementation wording may imply real event feed, model training, registry, provider or live readiness. | Enforce `not_authorized_by_approve`; CP7 must verify static/fixture boundary and release docs must avoid runtime readiness claims. |

## 后续收敛路径

1. CP1/CP2: meta-pm performs incremental use-case discovery and requirement extraction for unified event + ML adapter scope.
2. CP2 gate: user confirms unified adapter scope, alternatives and no-runtime authorization boundary.
3. CP3: meta-se prepares HLD/ADR for shared adapter contract and type-specific extension points.
4. CP4/CP5: meta-se/meta-dev prepare Story DAG and full Story design batch; user confirms CP5 before implementation.
5. CP6/CP7: meta-dev/meta-qa implement and verify local/static/fixture adapter behavior only.
6. CP8: release readiness records what is ready, what remains not authorized and what follow-ups remain.


## Closure Summary

| 字段 | 内容 |
|---|---|
| 关闭状态 | `closed-current-delivery / READY_WITH_RISK` |
| 关闭时间 | `2026-07-05T19:20:00+08:00` |
| 审批人 | user |
| CP8 checkpoint | `process/checkpoints/CP8-CR158-EVENT-ML-STRATEGY-ADAPTER-RELEASE-READINESS.md` |
| CP8 result | `process/checks/CP8-CR158-EVENT-ML-STRATEGY-ADAPTER-RELEASE-READINESS.result.json` |
| release context | `process/release/RELEASE-CONTEXT-CR158-EVENT-ML-STRATEGY-ADAPTER.yaml` |
| 接受风险 | `R-CR158-CP7-STATIC-FIXTURE-ONLY` |
| 不授权范围 | 真实发布、Git remote write、真实 event feed/listener、真实 ML training/model registry、数据湖 / NAS / provider / credential、QMT/gateway runtime、simulation/paper/live/trading/broker、catalog/store/registry/model/feature/label/prediction 写入、external framework run、production readiness claim。 |

CR158 当前交付关闭为 local/static/fixture event + ML adapter slice。后续真实 feed、真实 ML training、runtime、registry、publish 或 trading 验证只能通过独立 gate / CR 启动。


## Post-Closure Audit

| 字段 | 内容 |
|---|---|
| Meta-flow process self-audit | `process/checks/CR158-METAFLOW-PROCESS-SELF-AUDIT.md` |
| Token / redundancy retrospective | `process/checks/CR158-TOKEN-RETROSPECTIVE.md` |
| 结论 | CR158 最终合规关闭；发现的 CP2 brief omission、CP7 return enum、CP8 checker wording 和状态漂移均已在关闭前或关闭时修复。 |
| 主要优化方向 | human-gate renderer、batch-lld for homogeneous adapter stories、status-sync command、minimal-internal release profile。 |
