---
cr_id: "CR-155"
cr_type: "product-scope"
cr_kind: "requirement-change"
title: "Daily Multifactor Baseline Strategy Artifact"
lifecycle_status: "closed"
readiness_status: "ready_with_risk"
gate_status: "closed"
status: "closed-current-delivery-ready-with-risk"
gate_profile: "standard"
conflict_keys:
  - "daily-multifactor-baseline-strategy-artifact"
  - "daily-multifactor-baseline"
  - "multifactor-strategy-artifact"
  - "strategy-admission-baseline-artifact"
impact_surface:
  - "requirements"
  - "scope"
  - "strategy-framework"
  - "multifactor"
  - "backtest-validation"
  - "research-admission"
  - "artifact-contract"
  - "human-gate"
product_baseline_refresh_required: true
product_baseline_refresh_status: "completed"
required_phase: "delivered"
required_agent: ""
required_gate: ""
block_story_decomposition_until: "completed"
affected_product_docs:
  - "process/USE-CASES.md"
  - "docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP.md"
  - "docs/design/STRATEGY-FRAMEWORK-IMPLEMENTATION-ASSESSMENT-AND-REMEDIATION-PLAN-2026-07-01.md"
  - "docs/design/STRATEGY-E2E-FRAMEWORK-REVIEW-2026-07-01.md"
affected_use_cases:
  - "UC-58 multifactor strategy E2E"
authz_policy_refs:
  - "NO_CREDENTIAL_READ"
  - "NO_RUNTIME"
  - "NO_PRODUCTION_WRITE"
  - "NO_TRADING"
  - "REAL_LAKE_READONLY_APPROVED_FOR_CR155_CP2"
  - "NO_REAL_LAKE_WRITE"
  - "NO_NAS_SYNC_OR_WRITE"
  - "NO_PROVIDER_FETCH"
  - "NO_BROKER_WRITE"
  - "NO_EXTERNAL_FRAMEWORK_RUN"
  - "NO_CATALOG_POINTER_WRITE"
risk_refs:
  - "R-CR155-ARTIFACT-OVERCLAIM-001"
  - "R-CR155-REAL-DATA-RUNTIME-BOUNDARY-001"
created_at: "2026-07-04T13:46:34+08:00"
created_by: "host-orchestrator"
source: "user"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
impact_level: "medium"
rollback_to: "delivered / no active formal CR baseline after CR154 closure"
approval_result: "approved / CP2"
cr_index_path: "process/changes/CR-INDEX.yaml"
summary_ref: "process/changes/summaries/CR-155.summary.json"
cp0_result_ref: "process/checks/CP0-CR155-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT-REQUEST-INTAKE.result.json"
cp0_context_ref: "process/context/CP0-CR155.context.json"
cp1_result_ref: "process/checks/CP1-CR155-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT-USE-CASE-COMPLETENESS.result.json"
cp2_context_ref: "process/context/CP2-CR155-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT-SCOPE-CONTEXT.yaml"
cp2_result_ref: "process/checks/CP2-CR155-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT-SCOPE.result.json"
cp2_checkpoint_ref: "process/checkpoints/CP2-CR155-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT-SCOPE.md"
cp3_context_ref: "process/context/CP3-CR155-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT-HLD-CONTEXT.yaml"
cp3_hld_ref: "docs/design/HLD-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT.md"
cp3_adr_ref: "docs/design/ARCHITECTURE-DECISION-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT.md"
cp3_result_ref: "process/checks/CP3-CR155-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT-HLD-CONSISTENCY.result.json"
cp3_checkpoint_ref: "process/checkpoints/CP3-CR155-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT-HLD-REVIEW.md"
cp4_result_ref: "process/checks/CP4-CR155-STORY-DAG-PARALLEL-SAFETY.result.json"
cp5_context_ref: "process/context/CP5-CR155-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT-LLD-CONTEXT.yaml"
cp5_result_ref: "process/checks/CP5-CR155-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT-LLD-BATCH.result.json"
cp5_checkpoint_ref: "process/checkpoints/CP5-CR155-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT-LLD-BATCH.md"
cp6_result_ref: "process/checks/CP6-CR155-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT-IMPLEMENTATION.result.json"
cp6_evidence_ref: "process/evidence/CR155-CP6-IMPLEMENTATION.index.json"
cp7_result_ref: "process/checks/CP7-CR155-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT-VERIFICATION.result.json"
cp7_evidence_ref: "process/evidence/CR155-CP7-VERIFICATION.index.json"
cp8_result_ref: "process/checks/CP8-CR155-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT-RELEASE-READINESS.result.json"
cp8_checkpoint_ref: "process/checkpoints/CP8-CR155-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT-RELEASE-READINESS.md"
release_context_ref: "process/release/RELEASE-CONTEXT-CR155.yaml"
real_lake_validation_ref: "process/evidence/CR155-REAL-LAKE-VALIDATION-20260704.json"
real_lake_artifact_package_ref: "process/evidence/CR155-DAILY-MULTIFACTOR-BASELINE-REAL-LAKE-ARTIFACT-PACKAGE.json"
story_backlog_ref: "process/STORY-BACKLOG-CR155.md"
story_status_ref: "process/STORY-STATUS-CR155.md"
development_plan_ref: "process/DEVELOPMENT-PLAN-CR155.yaml"
feature_design_matrix_ref: "docs/design/FEATURE-DESIGN-MATRIX.md#cr155-cp4-增量daily-multifactor-baseline-strategy-artifact"
routing_design_ref: "docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP.md"
---

# CR-155 Daily Multifactor Baseline Strategy Artifact

## 变更描述

启动 CR155：日频多因子 baseline 策略 artifact。

本 CR 的目标是把既有多因子研究、统一回测 / experiment foundation、statistical admission gate 与 cross-strategy reliability gates 收敛成一个可审计的日频多因子 baseline strategy artifact。CR155 已通过 CP6 implementation、CP7 verification 和 CP8 release-readiness 自动检查；用户授权后已补做真实本地 governed lake readonly historical/OOS/rerun validation。用户已批准 CP8 `READY_WITH_RISK`：artifact 完整、真实 lake readonly 验证已执行、rerun 一致，但 admission package 为 `BLOCKED`，`paper_candidate=false`。CR155 当前已关闭。本轮仍不授权 lake write、runtime、trading、broker、store / registry、publish 或 production deployment。

## 用户目标

用户希望推进一个可被后续研究、回测、admission 和交付证据引用的日频多因子 baseline 策略 artifact。该 artifact 需要清楚表达策略标识、输入 / 输出引用、回测与实验基线、统计准入、可靠性门控、报告 / evidence 链接和不授权边界。

## 当前基线

| 基线 | 当前事实 | 证据 |
|---|---|---|
| CR148 unified backtest / experiment foundation | 已闭环统一 backtest run spec、report pack、cost/risk/attribution pack 和 daily multifactor baseline scope checks。 | `process/changes/CR-148-UNIFIED-BACKTEST-EXPERIMENT-FOUNDATION-2026-07-01.md`、`process/evidence/CR148-PHASE3-BACKTEST-COST-RISK-ATTRIBUTION-2026-07-01.json` |
| CR151 multifactor statistical admission | 已闭环 local/static/fixture 多因子 statistical admission gate；不代表真实数据或 runtime readiness。 | `process/changes/CR-151.md`、`process/changes/summaries/CR-151.summary.json` |
| CR154 cross-strategy reliability gates | 已闭环 local/static/fixture 横切可靠性门控；CP8 以 READY_WITH_RISK 收尾。 | `process/changes/CR-154.md`、`process/changes/summaries/CR-154.summary.json` |
| 当前 CR tracking | 当前无 active / blocked formal CR；CR155 可成为新的 active formal CR。 | `uv run meta-flow check cr-tracking --project-root .` |

## 范围

### In Scope

1. 产出一条日频多因子 baseline 策略：简单因子组合、非最优策略、非收益承诺。
2. 产出 strategy artifact，至少包含 `strategy_id`、universe ref、factor spec、signal spec、portfolio policy、rebalance policy、cost / slippage policy、backtest refs、experiment refs、report refs 和 evidence refs。
3. 产出完整历史回测结果，基于 CP2 明确授权后的本地 governed lake snapshot / current truth 只读输入；若 CP2 不授权真实 lake readonly，则本项降级为 fixture/static 并不得声明真实数据基线。
4. 产出 OOS / walk-forward 验证结果，记录 split、purge / embargo、rerun id 和主要 metrics。
5. 产出 statistical admission gate 评估结果，状态为 `pass` / `fail` / `needs_review`，并给出可审计理由。
6. 产出 cross-strategy reliability gate 评估结果，覆盖 CR154 已建立的回测陷阱、OOS / walk-forward、PIT / survivorship、capacity / impact 和 admission default policy 相关门控。
7. 产出 admission package，包含 `paper_candidate=true|false`、理由、阻断项、风险项和不授权边界。
8. 产出两次 rerun 核心 metrics 一致性证据，至少覆盖组合收益、回撤、换手、成本、capacity / liquidity 相关摘要和 admission status。
9. 明确与 CR148 / CR151 / CR154 的关系：CR155 消费其已交付的 local/static/fixture contract 能力，但本 CR 的核心新增物是具体 daily multifactor baseline strategy artifact。

### Deferred / Later Wave Candidate

1. 超出 CP2 授权范围的真实 lake 读取；任何 lake 写入。
2. catalog pointer、feature store、label store、model registry、event store 或 prediction store 写入。
3. QMT / MiniQMT / xtquant / gateway runtime、simulation、paper、live、trading 或 broker 操作。
4. 真实 TCA、真实成交回放、真实 reconciliation 或 production deployment。
5. 多策略组合、ML strategy 或 event-driven strategy 的真实 baseline artifact 扩展。

## 不授权范围

- credentials / secret / account read
- `.env` read
- real lake write
- real lake readonly before CP2 approval
- NAS read / write / sync / restore / chmod / chgrp / metadata normalization
- provider fetch
- QMT / MiniQMT / xtquant / gateway runtime
- simulation / paper / live / trading runtime
- broker read / write / submit / cancel / real account query
- external framework clone / install / run
- Git remote write
- catalog pointer mutation
- feature store / label store / event store / model registry / prediction store writes
- true release execution / production deployment / publish

## 启动约束

- CR155 已通过 CP8 人工审批并关闭为 `READY_WITH_RISK`。
- CP2 已批准 CR155 范围内本地 governed lake snapshot / current truth readonly，仅用于后续 baseline artifact 的历史回测、OOS / walk-forward 和 rerun consistency。
- CP6 / CP7 已完成 refs-only contract artifact 实现、fixture/static verification 和用户授权后的真实 governed lake readonly historical/OOS/rerun validation；结果为 rerun consistency PASS，但 admission BLOCKED、`paper_candidate=false`。
- 若 CP2 需要改变 UC-58、roadmap 或 strategy framework remediation plan，必须采用增量更新并保留旧基线。
- 当前批准范围不授权 lake write、NAS/provider/credential/runtime/trading/broker/catalog/store/registry/publish。

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否新增或重定义多因子 baseline strategy artifact 需求 | UC-58 / strategy framework roadmap | true | 回到 requirement-clarification，CP2 确认 artifact 范围和验收边界。 |
| 场景层 | 是否改变多因子 E2E 验收口径 | research admission、backtest validation、artifact evidence | true | CP2 明确建议范围：以本地 governed lake snapshot / current truth readonly 为推荐输入；若不授权则降级 fixture/static 且不得声明真实数据 artifact。 |
| 计划层 | 是否改变后续 Story / Wave / 任务依赖 | CR148 / CR151 / CR154 后续实现计划 | true | CP2 通过后进入 CP3/HLD，再决定 Story 拆分和 CP5 设计批次。 |
| 安全层 | 是否引入真实运行、凭据或外部系统权限 | lake、NAS、provider、QMT、broker、credential、catalog/store/registry | true | 本地 governed lake readonly 是 CP2 决策项；lake write、NAS/provider/credential/runtime/trading/broker/catalog/store/registry 全部保持不授权。 |
| 交付层 | 是否需要新增 artifact contract、测试、证据和 release readiness | strategy artifact、tests、process evidence、release context | true | CP2/CP3/CP5 后才允许实施，CP6/CP7/CP8 证明。 |

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/USE-CASES.md` | 原文档更新或不变待 CP2 决定 | UC-58 既有基线保留，CR155 只追加 baseline artifact 场景或建立映射 | `## 修订记录` / 不适用 | pending |
| `docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP.md` | 原文档更新或不变待 CP2 决定 | 保留既有 roadmap，必要时追加 CR155 位置 | `## 修订记录` / 不适用 | pending |
| `docs/design/STRATEGY-FRAMEWORK-IMPLEMENTATION-ASSESSMENT-AND-REMEDIATION-PLAN-2026-07-01.md` | 原文档更新或不变待 CP2 决定 | 保留 CR148 / CR151 / CR154 基线，必要时追加 CR155 artifact 切片 | `## 修订记录` / 不适用 | pending |
| `docs/design/STRATEGY-E2E-FRAMEWORK-REVIEW-2026-07-01.md` | 不变或引用待 CP2 决定 | 作为既有评审输入，不整体替换 | 既有修订记录 | pending |
| `process/changes/CR-155-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT-2026-07-04.md` | 新增 | N/A | 本文件 frontmatter / 正文 | active |

## 回退决策

- 回退到阶段：`delivered` / no active formal CR baseline after CR154 closure。
- 若 CP8 不通过：CR155 保持 active，并按用户要求做策略 remediation、重跑真实数据验证或修改 release readiness 包。
- 若 CP8 缩小范围：只关闭 artifact contract / evidence 子集，其余进入 follow-up 或 later CR。
- 若用户要求 lake write、NAS/provider/runtime/trading/broker/credential/catalog/store/registry 能力：不并入 CR155 first wave，另起 runtime_authorization 或正式 CR。

## 产品基线重整门禁

- 是否需要产品基线重整：true
- 完成状态：`completed by CP2`
- 下一阶段：`documentation / CP8 human review`
- 责任 Agent：`meta-se`
- 必须通过门禁：`CP8`
- Story / LLD / 实现阻断条件：`CP3-approved`
- 受影响产品文档：`process/USE-CASES.md`、roadmap / remediation / E2E review 设计输入
- 受影响 use case：`UC-58 multifactor strategy E2E`

## CP2 建议决策项

| 决策 ID | 类型 | 推荐方案 | 备选方案 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|
| DQ-CP2-CR155-ARTIFACT-SCOPE | scope | 批准 CR155 产出一条日频多因子 baseline 策略 artifact，包含 strategy id、universe、factor spec、signal、portfolio policy、完整历史回测、OOS / walk-forward、statistical admission、cross-strategy reliability gate、admission package 和 rerun consistency。 | 只做 artifact schema，不生成具体 baseline artifact；或只做 fixture artifact。 | 推荐方案能产生可审计基线，但需要真实 lake readonly 决策和更完整验证；备选范围更低风险但价值不足。 | 若 CP2 不授权真实 lake readonly，则降级 fixture/static artifact，并禁止真实数据相关声明。 |
| DQ-CP2-CR155-REAL-LAKE-READONLY | runtime_authorization | 授权首次本地 governed lake snapshot / current truth readonly，用于 CR155 baseline artifact 的历史回测、OOS / walk-forward 和 rerun consistency；不授权任何 lake write、catalog pointer mutation、NAS/provider/credential/runtime/trading/broker。 | 不授权真实 lake readonly，仅允许 fixture/static 数据。 | 推荐方案使 artifact 可声明“基于本地 governed lake snapshot / current truth readonly”；主要风险是边界误读为写入或生产就绪。 | 任一任务需要写 lake、改 catalog pointer、读凭据、provider fetch、NAS 操作或 runtime 时立即阻断并另起授权门。 |
| DQ-CP2-CR155-ADMISSION-AND-RERUN-CRITERIA | implementation | admission package 必须包含 `paper_candidate=true|false` 和理由；两次 rerun 核心 metrics 必须一致或解释差异，核心 metrics 覆盖收益、回撤、换手、成本、capacity / liquidity 摘要和 gate status。 | 只要求单次回测和 admission status，不要求 rerun。 | 推荐方案提高 artifact 可审计性；备选更快但难以排查非确定性。 | 若 rerun metrics 不一致且无法解释，则 admission package 必须 `paper_candidate=false` 或 `needs_review`。 |

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | false | artifact contract 会影响策略框架入口和后续验收。 |
| 修改架构、权限、安全边界或平台安装路径 | true | 需要明确 no-runtime/no-real-data/no-store/catalog 边界。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | true | 可能新增 artifact contract、tests、evidence 和 docs。 |
| 需要 HLD / LLD 才能解释影响 | true | CP2 后需要 CP3/CP5 决定具体 contract 和 Story。 |
| 是否保持 fast-lane | false | 使用 standard。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：待 CP3 / CP4 确认
- batch_id：`CR-155-LLD-BATCH`
- 批次范围来源：CP2/CP3 后的 Story planning
- 开发启动条件：
  - [ ] CP2 范围基线 approved
  - [ ] CP3 HLD / ADR approved
  - [ ] CP4 Story / DAG / parallel safety PASS
  - [ ] 批次内全部 Story 设计证据已输出
  - [ ] 批次内全部 Story CP5 自动预检已通过
  - [ ] CP5 批次人工确认结论为 `approved`

## 当前状态

- CP0：PASS，CR155 已受理。
- CP1：PASS，UC-58 / roadmap / remediation / E2E review completeness 已检查。
- CP2：approved，用户批准 artifact scope、CR155-scoped governed lake/current truth readonly 和 admission/rerun criteria。
- CP3：approved，用户批准 artifact contract、readonly validation pipeline、rerun consistency policy 和 no-runtime/no-write boundary。
- CP4：PASS，Story DAG / parallel safety / lld_policy 自动预检通过。
- CP5：approved，用户批准 5 个 full LLD、readonly implementation boundary 和 strict rerun tolerance policy。
- CP6：PASS，新增 CR155 refs-only baseline artifact contract module、fixture tests 和 CP6 evidence。
- CP7：PASS_WITH_RISK，验证 refs-only contract correctness、fixture tests、fail-closed behavior、no-overclaim wording、no-runtime/no-write boundary，并补做真实本地 governed lake readonly validation。真实验证两次 rerun 核心 metrics 一致，但 `economic_significance` 和 `out_of_sample_validation` blocked。
- CP8：approved / READY_WITH_RISK，用户接受 `DEC-CR155-CP8-001`，关闭 CR155 artifact scope。
- 当前阶段：delivered。
- 当前允许动作：无 active formal CR；`FU-CR155-001` 仅作为后续候选项。继续禁止 lake write、NAS/provider/credential/runtime/trading/broker/catalog/store/registry/publish。

## CP4 Story Planning 结果

| Story | 范围 | CP5 设计证据 |
|---|---|---|
| `CR155-S01-baseline-artifact-contract` | Strategy artifact contract and claim boundary | full-lld |
| `CR155-S02-readonly-data-provenance-adapter` | Readonly local governed lake/current truth provenance and guardrails | full-lld |
| `CR155-S03-backtest-oos-walkforward-validation` | Historical backtest and OOS / walk-forward validation flow | full-lld |
| `CR155-S04-admission-gate-composition-package` | CR151 / CR154 gate composition and admission package | full-lld |
| `CR155-S05-rerun-consistency-release-evidence` | Two-run consistency and release wording | full-lld |

CP4 does not authorize LLD approval, source implementation, test implementation, lake write, catalog pointer mutation, NAS/provider/credential access, runtime, trading, broker operation, external framework execution, store/registry write, publish or production deployment.

## CP5 待确认点

| 决策 ID | 待确认内容 |
|---|---|
| DQ-CP5-CR155-LLD-BATCH-APPROVAL | 是否批准 5 个 CR155 full LLD 进入实现。 |
| DQ-CP5-CR155-READONLY-IMPLEMENTATION-BOUNDARY | 是否确认 CP6 继续使用 CP2 已批准的 CR155-scoped readonly 边界，且不授权写/runtime/trading。 |
| DQ-CP5-CR155-RERUN-TOLERANCE-POLICY | 是否批准 rerun 默认严格 tolerance policy。 |

## CP3 已确认点

| 决策 ID | 确认内容 |
|---|---|
| CP3-DQ-CR155-ARTIFACT-CONTRACT | 批准 standalone artifact contract plus admission package consumer relationship。 |
| CP3-DQ-CR155-READONLY-PIPELINE | 批准 isolated CR155-scoped readonly validation pipeline。 |
| CP3-DQ-CR155-RERUN-POLICY | 批准两次 rerun consistency evidence 作为一级验收对象。 |
| CP3-DQ-CR155-NO-RUNTIME | 确认 CP3 不授权 Story/LLD/implementation、lake write、NAS/provider/credential/runtime/trading/broker/publish。 |

## CP3 历史决策清单

| 决策 ID | 已确认内容 |
|---|---|
| CP3-DQ-CR155-ARTIFACT-CONTRACT | 是否批准 standalone artifact contract plus admission package consumer relationship。 |
| CP3-DQ-CR155-READONLY-PIPELINE | 是否批准 isolated CR155-scoped readonly validation pipeline。 |
| CP3-DQ-CR155-RERUN-POLICY | 是否批准两次 rerun consistency evidence 作为一级验收对象。 |
| CP3-DQ-CR155-NO-RUNTIME | 是否确认 CP3 不授权 Story/LLD/implementation、lake write、NAS/provider/credential/runtime/trading/broker/publish。 |

## CP2 已确认点

| 决策 ID | 确认内容 |
|---|---|
| DQ-CP2-CR155-ARTIFACT-SCOPE | 批准 CR155 产出一条具体日频多因子 baseline strategy artifact，包含 strategy_id、universe、factor spec、signal、portfolio policy、完整历史回测、OOS / walk-forward、statistical admission、cross-strategy reliability gate、admission package 和 rerun consistency。 |
| DQ-CP2-CR155-REAL-LAKE-READONLY | 授权 CR155 范围内本地 governed lake snapshot / current truth readonly，用于 historical backtest、OOS / walk-forward 和 rerun consistency；不授权 lake write、catalog pointer、NAS/provider/credential/runtime/trading/broker。 |
| DQ-CP2-CR155-ADMISSION-AND-RERUN-CRITERIA | admission package 必须包含 `paper_candidate=true|false` 和理由；两次 rerun 核心 metrics 必须一致或解释差异。 |

用户于 2026-07-04T18:15:00+08:00 回复“批准当前CP，继续推进项目”，按 CP2 `approved` 处理。
