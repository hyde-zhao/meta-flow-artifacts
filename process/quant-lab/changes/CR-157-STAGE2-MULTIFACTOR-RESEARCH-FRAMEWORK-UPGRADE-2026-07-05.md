---
cr_id: "CR-157"
cr_type: "product-scope"
cr_kind: "requirement-change"
title: "Stage 2 Multifactor Research Framework Upgrade"
lifecycle_status: "closed"
readiness_status: "ready_with_risk"
gate_status: "closed"
cp2_status: "approved"
cp2_approved_at: "2026-07-05T12:01:53+08:00"
cp3_status: "approved"
cp3_approved_at: "2026-07-05T12:48:20+08:00"
gate_profile: "architecture-major"
route_plan_ref: ""
cr_trait_uses_existing_evidence_only: false
cr_trait_has_new_design: true
cr_trait_has_new_implementation: true
cr_trait_has_new_verification: true
cr_trait_requires_architecture_review: true
cr_trait_requires_story_decomposition: true
cr_trait_requires_subagent_dispatch: false
status: "closed-current-delivery-ready-with-risk"
impact_level: "medium"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "Stage 2 changes product/design baseline, framework contracts and likely multiple Story boundaries."
rollback_to: "CR156 closed state plus pre-CR157 Stage 2 baseline"
approval_result: "approved"
created_at: "2026-07-05T11:23:42+08:00"
created_by: "host-orchestrator"
source: "user"
parent_cr: "ROADMAP-QUANT-RESEARCH-PRODUCTION"
source_checkpoint: "process/checks/CR156-META-FLOW-PROCESS-RETROSPECTIVE-2026-07-05.md"
source_decision_id: "USER-20260705-NEXT-STAGE2-AFTER-CR156"
goal_ref: "STAGE2-MULTIFACTOR-FRAMEWORK-UPGRADE"
goal_statement: "Advance Stage 2 multifactor research framework deepening after CR156 closure."
user_goal_impact: "Move from strategy framework foundation closure into the next research-framework upgrade slice while preserving no-runtime and no-publish boundaries."
approval_focus: "scope"
split_rationale: "FU-CR154-001 is already closed under CR156; Stage 2 is a new product/framework increment with different scope, design, implementation and verification surfaces."
why_not_merge_with_parent: "The roadmap is an umbrella baseline, while CR157 is the next executable Stage 2 slice."
why_not_story_or_task: "The requested next step changes framework scope and design baseline before implementation; it must pass CP2 before Story decomposition."
decision_burden: "medium"
approve_effect: "Approve Stage 2 scope discovery and route CR157 to requirement-clarification / meta-pm / CP2 before HLD, Story split or implementation."
reject_effect: "Keep current delivered state after CR156 and do not start Stage 2 framework upgrade work."
not_authorized_by_approve:
  - "real lake write"
  - "NAS read/write/sync/restore or metadata normalization"
  - "provider fetch"
  - "credential or env read"
  - "QMT/MiniQMT/xtquant/gateway runtime"
  - "simulation/paper/live/trading runtime"
  - "broker write/submit/cancel/real account query"
  - "catalog/store/registry/model/prediction write"
  - "Git remote write"
  - "true release execution or publish"
  - "external framework clone/install/run"
product_baseline_refresh_required: false
product_baseline_refresh_status: "approved"
required_phase: "delivered"
required_agent: ""
required_gate: ""
cp6_status: "pass"
cp6_completed_at: "2026-07-05T13:50:00+08:00"
cp6_result_ref: "process/checks/CP6-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-IMPLEMENTATION.result.json"
cp6_return_ref: "process/returns/CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.CP6.return.json"
cp6_evidence_ref: "process/evidence/CR157-CP6-IMPLEMENTATION.index.json"
cp7_status: "pass_with_risk"
cp7_context_ref: "process/context/CP7-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-VERIFICATION-CONTEXT.yaml"
cp7_completed_at: "2026-07-05T14:20:00+08:00"
cp7_result_ref: "process/checks/CP7-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-VERIFICATION.result.json"
cp7_return_ref: "process/returns/CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.CP7.return.json"
cp7_evidence_ref: "process/evidence/CR157-CP7-VERIFICATION.index.json"
cp7_verification_report_ref: "docs/quality/VERIFICATION-REPORT-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md"
cp8_status: "approved"
cp8_checkpoint_ref: "process/checkpoints/CP8-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-RELEASE-READINESS.md"
cp8_context_ref: "process/context/CP8-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-RELEASE-CONTEXT.yaml"
cp8_result_ref: "process/checks/CP8-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-RELEASE-READINESS.result.json"
cp8_approved_at: "2026-07-05T14:30:00+08:00"
cp8_approved_by: "user"
release_context_ref: "process/release/RELEASE-CONTEXT-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.yaml"
release_decision: "READY_WITH_RISK"
cp5_status: "approved"
cp5_ready_at: "2026-07-05T13:30:00+08:00"
cp5_approved_at: "2026-07-05T13:31:00+08:00"
cp5_approved_by: "user"
cp5_result_ref: "process/checks/CP5-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-LLD-IMPLEMENTABILITY.result.json"
cp5_checkpoint_ref: "process/checkpoints/CP5-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-LLD-BATCH.md"
cp5_context_ref: "process/context/CP5-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-LLD-CONTEXT.yaml"
block_story_decomposition_until: "CP3-approved"
affected_product_docs:
  - "docs/product/USE-CASES.md"
  - "docs/product/REQUIREMENTS.md"
  - "docs/product/SCENARIOS.yaml"
  - "docs/product/TEST-MATRIX.md"
  - "docs/product/STORY-MAP.md"
  - "docs/product/MVP-SCOPE.md"
  - "docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP-2026-07-01.md"
  - "docs/components/MULTIFACTOR-RESEARCH.md"
affected_use_cases:
  - "UC-58 multifactor strategy E2E"
routing_design_ref: "process/checks/STAGE2-MULTIFACTOR-FRAMEWORK-NO-LAKE-INITIAL-SLICE-2026-06-26.md"
cr_index_path: "process/changes/CR-INDEX.json"
---

# CR-157 Stage 2 Multifactor Research Framework Upgrade

## 变更描述

用户确认 CR156 关闭后的下一步应先评审 `process/checks/CR156-META-FLOW-PROCESS-RETROSPECTIVE-2026-07-05.md` 的流程审计发现和优化机会，然后进入 Stage 2 多因子研究框架升级，或收束 `FU-CR154-001` 最终打包 hygiene。

本次状态核对结论：

- `FU-CR154-001` 已在 CR156 CP8 下关闭，证据为 `process/checks/FU-CR154-001-FINAL-PACKAGING-HYGIENE-2026-07-04.md`，不再作为下一步 active 候选。
- CR156 流程复盘无阻断项，主要建议是把 existing-evidence hygiene closure、CR index canonical path、N/A / WAIVED templates、quiet JSON validation 和 state source_refs slimming 产品化。
- Stage 2 已有 no-lake initial slice 证据，下一片应围绕成熟 admission package builder、Stage 2/Stage 3 handoff、框架合同深化与后续 Story 拆分做产品基线确认。

## 目标影响摘要

| 字段 | 内容 |
|---|---|
| 目标引用 | `STAGE2-MULTIFACTOR-FRAMEWORK-UPGRADE` |
| 整体目标 | 从策略框架基础 CR 收敛点推进到 Stage 2 多因子研究框架深化。 |
| 用户目标影响 | 将当前工作从 hygiene / closure 转入研究框架能力升级，避免继续处理已关闭的 FU-CR154-001。 |
| 本 CR 为什么值得独立推进 | Stage 2 影响产品范围、设计边界、框架合同、Story 拆分和验证策略，不能作为 CR156 后续 hygiene 或单个 task 处理。 |
| approve 后会发生什么 | 已批准。CR157 关闭为 `closed-current-delivery-ready-with-risk`；仅确认本地/static/fixture slice 交付就绪，不授权真实发布、真实数据或 runtime 操作。 |
| reject / 不确认会阻塞什么 | N/A；CP8 已批准。 |
| 决策负担 | medium |

## 拆分理由

| 问题 | 结论 |
|---|---|
| 为什么不合并到 parent / active CR | 当前无 active formal CR；CR156 已关闭且只覆盖 hygiene consolidation。 |
| 为什么不是 Story / task / follow-up | Stage 2 下一片尚需确认产品范围和设计边界，CP2 未通过前不得拆 Story 或实现。 |
| 触发独立 CR 的边界 | 新的 Stage 2 framework upgrade，包含成熟 admission package builder / framework contract / handoff /验证策略，但不授权 runtime、真实数据写入、NAS/provider、publish 或交易。 |

## CR 类型与门禁策略

| 字段 | 内容 |
|---|---|
| CR 类型 | `product-scope` |
| Legacy CR kind | `requirement-change` |
| 生命周期状态 | `active` |
| 就绪状态 | `ready` |
| 门禁状态 | `closed` |
| 门禁模板 | `architecture-major` |
| Route plan | 待 CP0 route plan 固化 |

## Checkpoint Index

| CP | 状态 | 机器结果 ref | 人工门禁 ref | Context ref | Ledger event ref | 摘要 |
|---|---|---|---|---|---|---|
| CP0 | pass | `process/checks/CP0-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-REQUEST-INTAKE.result.json` | N/A | `process/context/CP0-CR157.context.json` | `process/state/CHECKPOINT-LEDGER.ndjson` | Stage 2 scope captured; FU-CR154-001 confirmed closed. |
| CP1 | pass | `process/checks/CP1-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-USE-CASE-COMPLETENESS.result.json` | N/A | `process/context/CP2-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-CONTEXT.yaml` | `process/state/CHECKPOINT-LEDGER.ndjson` | Stage 2 product baseline drafts and SGQ routing are complete enough for CP2. |
| CP2 | approved | `process/checks/CP2-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-SCOPE.result.json` | `process/checkpoints/CP2-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-SCOPE.md` | `process/context/CP2-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-CONTEXT.yaml` | `process/state/GATE-LEDGER.ndjson` | User approved `DQ-CP2-CR157-FIRST-SLICE`; route to CP3 solution-design. |
| CP3 | approved | `process/checks/CP3-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-HLD-CONSISTENCY.result.json` | `process/checkpoints/CP3-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-HLD-REVIEW.md` | `process/context/CP3-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-HLD-CONTEXT.yaml` | `process/state/GATE-LEDGER.ndjson` | User approved 5 CP3 decisions; route to CP4 Story planning only. |
| CP4 | pass | `process/checks/CP4-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-STORY-DAG-PARALLEL-SAFETY.result.json` | N/A | `process/context/CP4-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-STORY-PLANNING-CONTEXT.yaml` | `process/state/CHECKPOINT-LEDGER.ndjson` | Story DAG and parallel safety passed; 5 Story CP5 batch prepared. |
| CP5 | approved | `process/checks/CP5-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-LLD-IMPLEMENTABILITY.result.json` | `process/checkpoints/CP5-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-LLD-BATCH.md` | `process/context/CP5-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-LLD-CONTEXT.yaml` | `process/state/GATE-LEDGER.ndjson` | User approved all four CP5 decisions; route to CP6 local source/test implementation. |
| CP6 | pass | `process/checks/CP6-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-IMPLEMENTATION.result.json` | N/A | `process/context/CP6-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-IMPLEMENTATION-CONTEXT.yaml` | `process/state/CHECKPOINT-LEDGER.ndjson` | Local source/test/docs implementation passed; return and evidence index generated. |
| CP7 | pass_with_risk | `process/checks/CP7-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-VERIFICATION.result.json` | N/A | `process/context/CP7-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-VERIFICATION-CONTEXT.yaml` | `process/state/CHECKPOINT-LEDGER.ndjson` | meta-qa verification passed with static/fixture-only residual risk. |
| CP8 | approved / READY_WITH_RISK | `process/checks/CP8-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-RELEASE-READINESS.result.json` | `process/checkpoints/CP8-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-RELEASE-READINESS.md` | `process/context/CP8-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-RELEASE-CONTEXT.yaml` | `process/state/GATE-LEDGER.ndjson` | User accepted static/fixture-only residual risk and CR157 closed as READY_WITH_RISK. |

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
  external_framework:
    clone: false
    install: false
    run: false
  repository:
    git_remote_write: false
    publish: false
```

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `docs/product/USE-CASES.md` | 原文档更新 | 保留既有 UC-58 / research journey 基线，追加 Stage 2 增量场景 | `## 修订记录` | approved by CP2 |
| `docs/product/REQUIREMENTS.md` | 原文档更新 | 保留旧 REQ，追加 Stage 2 mature admission / handoff 需求 | `## 修订记录` | approved by CP2 |
| `docs/product/SCENARIOS.yaml` | 原文档更新 | 保留旧场景，追加 Stage 2 正/负/边界/权限场景 | 修订记录或 CR ref | approved by CP2 |
| `docs/product/TEST-MATRIX.md` | 原文档更新 | 保留现有矩阵，追加 Stage 2 覆盖项 | 修订记录或 CR ref | approved by CP2 |
| `docs/product/STORY-MAP.md` | 原文档更新 | 保留既有 story map，追加 Stage 2 release slice | `## 修订记录` | approved by CP2 |
| `docs/product/MVP-SCOPE.md` | 原文档更新 | 保留 CR155 前后 scope，追加 CR157 MVP / out of scope | `## 修订记录` | approved by CP2 |
| `docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP-2026-07-01.md` | 原文档更新 | 保留 v0.1 / review notes，追加 Stage 2 v0.2 routing delta | 文档修订记录 | CP6 implemented in canonical `docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP.md` |
| `docs/components/MULTIFACTOR-RESEARCH.md` | 原文档更新 | 保留 no-lake initial slice，追加下一片合同边界 | 文档修订记录 | CP6 implemented; pending CP7 verification |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| CR150 Multifactor Framework Completion | CR157 Stage 2 framework upgrade | 原文保留 | CR150 metadata-only completion map 是输入，不被重写。 |
| CR155 Daily Multifactor Baseline Strategy Artifact | CR157 Stage 2 mature framework deepening | 原文保留 | CR155 baseline artifact 的 admission BLOCKED 结论是 Stage 2 后续补齐输入。 |
| `process/checks/STAGE2-MULTIFACTOR-FRAMEWORK-NO-LAKE-INITIAL-SLICE-2026-06-26.md` | CR157 next slice | 原文保留 | initial slice 是既有证据，CR157 定义下一片范围。 |

## Stage 2 Exit Criteria Mapping

| Stage 2 退出条件对象 | 当前覆盖来源 | CR157 是否关闭 | 说明 |
|---|---|---|---|
| `FactorSpec` | CR150 / `docs/components/MULTIFACTOR-RESEARCH.md` | no | 作为既有合同输入，不在 CR157 重新设计。 |
| `FactorRunSpec` | CR150 / `docs/components/MULTIFACTOR-RESEARCH.md` | no | 作为既有合同输入，不在 CR157 重新设计。 |
| factor panel | CR150 / CR151 / component baseline | no | CR157 只要求 evidence refs 可追溯，不重新实现 panel。 |
| label window | CR150 / CR151 / component baseline | no | CR157 只要求 validation/evidence refs，不重开 label 设计。 |
| evaluation | CR151 statistical gate / CR154 reliability gate | no | CR157 消费 validation report refs，不重开统计门控。 |
| portfolio / risk | CR154 reliability gate / `PortfolioRiskPolicy` component baseline | partial | CR157 要求准入包引用 risk policy，不重新定义完整组合优化。 |
| admission package | CR157 | yes | 本轮 first slice 关闭 mature admission package builder scope / contract / fail-closed 校验。 |
| evidence index | CR157 | yes | 本轮 first slice 关闭 research evidence index traceability。 |
| Stage 2 / Stage 3 handoff | CR157 | yes | 本轮 first slice 关闭 handoff hardening 和 no-runtime/no-publish 边界。 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增、删除或重定义 REQ-* | `REQUIREMENTS.md`、Stage 2 scope | true | 回到 requirement-clarification，由 meta-pm 做增量需求确认并通过 CP2。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | `SCENARIOS.yaml` / `TEST-MATRIX.md` | true | 增加 mature admission package builder、handoff、no-runtime guard、fixture/static validation 场景。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | `STORY-MAP.md`、`MVP-SCOPE.md`、后续 `DEVELOPMENT-PLAN-CR157.yaml` | true | CP2 后进入 HLD / Story 拆分；CP2 前不得实现。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | 授权边界 / no-runtime policy | false | 明确不授权 runtime、真实 lake/NAS/provider、凭据、交易、publish、Git remote write。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | Stage 2 docs、component docs、tests/evidence | true | CP3/CP5 后按 Story 输出实现和验证证据。 |

## 回退决策

- 影响范围：Stage 2 产品 / 设计 / 实现计划局部范围。
- 回退到阶段：`CR156 closed state plus pre-CR157 Stage 2 baseline`。
- 需要重新确认的对象：CP2 scope、CP3 HLD、CP5 Story design batch、CP8 delivery readiness。

## 产品基线重整门禁

- 是否需要产品基线重整：`true`
- 必须回到阶段：`requirement-clarification`
- 责任 Agent：`meta-pm`
- 已通过门禁：`CP2`
- 下一门禁：`CP7`
- Story / LLD / 实现阻断条件：`CP6-passed; CP7 requires meta-qa dispatch evidence or explicit inline-fallback authorization`
- 受影响产品文档：见 frontmatter `affected_product_docs`
- 受影响 use case：`UC-58 multifactor strategy E2E`
- 分流设计引用：`process/checks/STAGE2-MULTIFACTOR-FRAMEWORK-NO-LAKE-INITIAL-SLICE-2026-06-26.md`

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | false | Stage 2 影响产品和设计基线。 |
| 修改架构、权限、安全边界或平台安装路径 | true | 需要确认 no-runtime / no-provider / no-store write 边界和架构合同。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | true | 预期涉及 mature admission package builder、handoff 和 framework contracts。 |
| 需要 HLD / LLD 才能解释影响 | true | 必须走 standard/architecture-major。 |

## 下一步

1. 发起 CP7 verification-execution，消费 `process/context/CP7-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-VERIFICATION-CONTEXT.yaml`。
2. 由 `meta-qa` 或用户明确批准的 inline-fallback 输出 CP7 verification report、return packet、evidence index 和 CP7 result。
3. 仍不得执行真实 lake/NAS/provider/credential/QMT/runtime/simulation/paper/live/trading/broker/catalog/store/registry/publish/external framework/Git remote 操作。
