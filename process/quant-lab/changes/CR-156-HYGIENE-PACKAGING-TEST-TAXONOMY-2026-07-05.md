---
cr_id: "CR-156"
cr_type: "process"
cr_kind: "requirement-change"
title: "Hygiene Consolidation: Packaging to Test Taxonomy"
lifecycle_status: "closed"
readiness_status: "ready_with_risk"
gate_status: "closed"
status: "closed-current-delivery-ready-with-risk"
gate_profile: "compact"
conflict_keys:
  - "final_packaging_hygiene"
  - "test_taxonomy_hygiene"
  - "test_provenance_coverage"
impact_surface:
  - "packaging-hygiene"
  - "test-taxonomy"
  - "test-provenance"
  - "follow-up-closure"
product_baseline_refresh_required: false
product_baseline_refresh_status: "not_required_process_hygiene_only"
required_phase: "delivered"
required_agent: "host-orchestrator"
required_gate: "CP8"
block_story_decomposition_until: "N/A - no story decomposition planned"
affected_product_docs: []
affected_use_cases: []
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
risk_refs:
  - "R-CR156-GIT-REMOTE-WRITE-MISREAD-001"
  - "R-CR156-RESIDUAL-UNRELATED-FULL-PYTEST-FAILURES-001"
created_at: "2026-07-04T23:43:42+08:00"
created_by: "host-orchestrator"
source: "user"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
impact_level: "low"
rollback_to: "delivered / no active formal CR baseline after CR155 closure"
approval_result: "approved"
cr_index_path: "process/changes/CR-INDEX.json"
summary_ref: "process/changes/summaries/CR-156.summary.json"
cp0_result_ref: "process/checks/CP0-CR156-REQUEST-INTAKE.result.json"
cp0_context_ref: "process/context/CP0-CR156.context.json"
cp1_result_ref: "process/checks/CP1-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-SCOPE-COMPLETENESS.result.json"
cp2_result_ref: "process/checks/CP2-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-SCOPE.result.json"
cp2_context_ref: "process/context/CP2-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-SCOPE-CONTEXT.yaml"
cp2_checkpoint_ref: "process/checkpoints/CP2-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-SCOPE.md"
cp3_result_ref: "process/checks/CP3-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-DESIGN-NOT-APPLICABLE.result.json"
cp4_result_ref: "process/checks/CP4-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-STORY-DAG-NOT-APPLICABLE.result.json"
cp5_result_ref: "process/checks/CP5-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-LLD-BATCH-NOT-APPLICABLE.result.json"
cp6_result_ref: "process/checks/CP6-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-IMPLEMENTATION-NOT-APPLICABLE.result.json"
cp7_result_ref: "process/checks/CP7-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-VERIFICATION-NOT-APPLICABLE.result.json"
cp8_result_ref: "process/checks/CP8-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-RELEASE-READINESS.result.json"
cp8_context_ref: "process/context/CP8-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-CLOSURE-CONTEXT.yaml"
cp8_checkpoint_ref: "process/checkpoints/CP8-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-RELEASE-READINESS.md"
release_context_ref: "process/release/RELEASE-CONTEXT-CR156.yaml"
closure_summary_ref: "process/checks/CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-CLOSURE-SUMMARY-2026-07-05.md"
routing_design_ref: ""
---

# CR-156 Hygiene Consolidation: Packaging to Test Taxonomy

## 变更描述

用户请求：“启动 CR156。请先按 meta-flow current state 恢复上下文：读取 AGENTS.md、process/current/CURRENT.json、process/state/STATE.current.json、process/changes/CR-INDEX.yaml，确认当前无 active formal CR 后，从 CP0 开始受理。”

当前已完成上下文恢复和 CR tracking 冲突预检：`process` 路由健康，`STATE.current.json.active_change=null`，`CR-INDEX.json.active_crs=[]`，`meta-flow check cr-tracking` 报告 active formal CRs none / blocked formal CRs none。

用户随后补充：“CR156的目标是 hygiene 收束: FU-CR154-001 (packaging) → FU-CR152-001 (test taxonomy)”。

本 CR 的目标由此确定为本地 hygiene consolidation：收束 `FU-CR154-001` packaging hygiene 与 `FU-CR152-001` test taxonomy / `tests/PROVENANCE.yaml` coverage hygiene。用户评审后确认两项已有 evidence，可并行做只读验证并统一关闭；该变更只处理过程台账、检查证据、局部 packaging/test taxonomy 收束与后续关闭路径，不改变产品需求基线，不授权 Git remote write、true release execution、publish、runtime、真实数据、凭据、NAS/provider、broker 或交易动作。

## 目标影响摘要

| 字段 | 内容 |
|---|---|
| 目标引用 | `CR156-HYGIENE-CONSOLIDATION` |
| 整体目标 | 统一收束 `FU-CR154-001` packaging hygiene 与 `FU-CR152-001` test taxonomy / provenance hygiene。 |
| 用户目标影响 | 降低 CR154 / CR152 CP8 后续事项遗留噪声，形成可关闭的本地 hygiene 证据链。 |
| 本 CR 为什么值得独立推进 | 两个 follow-up 均为局部 hygiene / packaging 收束，且当前无其他 active formal CR；合并到 CR156 可减少碎片化 CR。 |
| approve 后会发生什么 | CP8 已批准关闭；CR156 关闭为 `READY_WITH_RISK`，两个 follow-up 已关闭 under CR156。 |
| reject / 不确认会阻塞什么 | 不确认会保留 `FU-CR154-001` 与 `FU-CR152-001` 作为 open follow-up candidate。 |
| 决策负担 | `low`：范围已由用户指定，主要确认本地 hygiene 和不授权边界。 |

## 拆分理由

| 问题 | 结论 |
|---|---|
| 为什么不合并到 parent / active CR | CR154 / CR152 已关闭或进入后续跟踪；当前无 active / blocked formal CR，CR156 是新的收束外壳。 |
| 为什么不是 Story / task / follow-up | 用户显式启动 CR156，且目标是两个已有 follow-up 的顺序收束，需要一个统一审计闭环。 |
| 触发独立 CR 的边界 | `FU-CR154-001` packaging hygiene 与 `FU-CR152-001` test taxonomy / provenance hygiene 共同进入 CR156；不包含 CR155 admission remediation。 |

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-154` / `CR-152` follow-up candidates |
| 来源检查点 | `CR154 CP8` / `CR152 CP7` follow-up tracking |
| 来源决策 ID | `FU-CR154-001`、`FU-CR152-001` |
| follow-up 类型 | local process hygiene / packaging / test taxonomy |
| 风险等级 | low |
| owner | host-orchestrator |
| 重访条件 | CP1 / CP2 compact scope baseline 发现 scope 或授权边界需要扩大 |
| 验收标准 | `FU-CR154-001` packaging evidence 和 `FU-CR152-001` taxonomy/provenance evidence 均可追溯；不授权边界清晰；CR tracking 无活动冲突 |
| 关闭条件 | CR156 完成 CP8 或用户取消 / supersede |

## CR 类型与门禁策略

| 字段 | 内容 |
|---|---|
| CR 类型 | `process` |
| Legacy CR kind | `requirement-change` |
| 生命周期状态 | `closed` |
| 就绪状态 | `ready_with_risk` |
| 门禁状态 | `closed` |
| 门禁模板 | `compact` |

## Checkpoint Index

| CP | 状态 | 机器结果 ref | 人工门禁 ref | Context ref | Ledger event ref | 摘要 |
|---|---|---|---|---|---|---|
| CP0 | pass | `process/checks/CP0-CR156-REQUEST-INTAKE.result.json` | N/A | `process/context/CP0-CR156.context.json` | `process/state/CHECKPOINT-LEDGER.ndjson` | 目标已补充：`FU-CR154-001` packaging → `FU-CR152-001` test taxonomy。 |
| CP1 | pass | `process/checks/CP1-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-SCOPE-COMPLETENESS.result.json` | N/A | `process/context/CP2-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-SCOPE-CONTEXT.yaml` | `process/state/CHECKPOINT-LEDGER.ndjson` | compact scope completeness 通过。 |
| CP2 | approved | `process/checks/CP2-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-SCOPE.result.json` | `process/checkpoints/CP2-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-SCOPE.md` | `process/context/CP2-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-SCOPE-CONTEXT.yaml` | `process/state/CHECKPOINT-LEDGER.ndjson` / `process/state/GATE-LEDGER.ndjson` | 用户评审同意 3 项决策，并要求 CP3 / CP4 / CP5 明确 N/A、改为直接 CP8 closure route。 |
| CP3 | N/A | `process/checks/CP3-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-DESIGN-NOT-APPLICABLE.result.json` | N/A | `process/context/CP8-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-CLOSURE-CONTEXT.yaml` | `process/state/CHECKPOINT-LEDGER.ndjson` | process hygiene CR 无 HLD / ADR / architecture artifacts required。 |
| CP4 | N/A | `process/checks/CP4-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-STORY-DAG-NOT-APPLICABLE.result.json` | N/A | `process/context/CP8-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-CLOSURE-CONTEXT.yaml` | `process/state/CHECKPOINT-LEDGER.ndjson` | 无 Story decomposition / DAG / parallel-safety surface。 |
| CP5 | N/A | `process/checks/CP5-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-LLD-BATCH-NOT-APPLICABLE.result.json` | N/A | `process/context/CP8-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-CLOSURE-CONTEXT.yaml` | `process/state/CHECKPOINT-LEDGER.ndjson` | 无 LLD batch 或 Story design evidence required。 |
| CP6 | WAIVED | `process/checks/CP6-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-IMPLEMENTATION-NOT-APPLICABLE.result.json` | N/A | `process/context/CP8-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-CLOSURE-CONTEXT.yaml` | `process/state/CHECKPOINT-LEDGER.ndjson` | 本 CR 不执行新实现；消费 2026-07-04 已有 follow-up evidence。 |
| CP7 | WAIVED | `process/checks/CP7-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-VERIFICATION-NOT-APPLICABLE.result.json` | N/A | `process/context/CP8-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-CLOSURE-CONTEXT.yaml` | `process/state/CHECKPOINT-LEDGER.ndjson` | 不重复运行验证；CP8 统一核验既有 evidence 和 residual wording。 |
| CP8 | approved | `process/checks/CP8-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-RELEASE-READINESS.result.json` | `process/checkpoints/CP8-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-RELEASE-READINESS.md` | `process/context/CP8-CR156-HYGIENE-PACKAGING-TEST-TAXONOMY-CLOSURE-CONTEXT.yaml` | `process/state/CHECKPOINT-LEDGER.ndjson` / `process/state/GATE-LEDGER.ndjson` | 用户批准 CR156 关闭为 READY_WITH_RISK。 |

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
| `docs/product/USE-CASES.md` | 不变 | 本 CR 不改变用户场景或产品基线 | N/A | approved-by-scope |
| `docs/product/REQUIREMENTS.md` | 不变 | 本 CR 不新增 / 重定义 REQ | N/A | approved-by-scope |
| `docs/product/SCENARIOS.yaml` | 不变 | 本 CR 复用已有 follow-up 验证证据，不扩展产品场景 | N/A | approved-by-scope |
| `docs/product/TEST-MATRIX.md` | 不变 | test taxonomy hygiene 不重定义产品验收矩阵 | N/A | approved-by-scope |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否新增、删除或重定义 REQ-* | 无 | no product-baseline impact | 不修改 USE-CASES / REQUIREMENTS / SCENARIOS / MVP scope。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | `FU-CR152-001` 既有测试 taxonomy / provenance 证据 | no scenario expansion | 只追溯既有 hygiene 证据，不新增产品测试场景。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | CR156 compact follow-up closure | low impact | 无 Story 拆解；两项 evidence 可并行只读核验并统一 closure。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | 默认全部不授权 | controlled | 禁止 runtime、真实数据、凭据、NAS/provider、broker、trading、Git remote、publish。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | follow-up checks / CR tracking / local packaging evidence | local only | 复用已有 PASS / PASS_WITH_RESIDUAL_UNRELATED_FAILURES 证据并形成关闭路径。 |

## 回退决策

- 影响范围：本地 process hygiene、packaging evidence、test taxonomy / provenance evidence 和 CR tracking。
- 回退到阶段：`delivered / no active formal CR baseline after CR155 closure`。
- 需要重新确认的对象：如 CP1 / CP2 发现需要扩大到代码实现、真实 release、Git remote 或 runtime，则必须新建授权决策或回退为 candidate。

## 产品基线重整门禁

- 是否需要产品基线重整：false。
- 必须回到阶段：`requirement-clarification`（compact CP1/CP2 scope baseline only）。
- 责任 Agent：`host-orchestrator`。
- 必须通过门禁：`CP1` / 如需人工确认则 `CP2`。
- Story / LLD / 实现阻断条件：N/A，本 CR 不计划 Story / LLD。
- 受影响产品文档：无。
- 受影响 use case：无。

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | yes | 本地 follow-up hygiene 收束。 |
| 修改架构、权限、安全边界或平台安装路径 | no | 不扩大权限，不修改平台路径。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | no | 不引入 Story / LLD。 |
| 需要 HLD / LLD 才能解释影响 | no | 既有 follow-up evidence 已说明目标。 |
| 是否保持 fast-lane | no | 工作流仍记录为 standard；本 CR 采用 compact gate profile。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：false。
- batch_id：`CR-156-LLD-BATCH`。
- 批次范围来源：N/A。
- 批次内 Story：N/A。

## 风险与开放项

| ID | 状态 | 问题 | owner | 下一步 |
|---|---|---|---|---|
| R-CR156-GIT-REMOTE-WRITE-MISREAD-001 | OPEN | packaging / commit hygiene 容易被误读为允许 push、publish 或 true release。 | host-orchestrator | CP1 / CP2 和后续关闭文件继续显式声明不授权 Git remote write / true release / publish。 |
| R-CR156-RESIDUAL-UNRELATED-FULL-PYTEST-FAILURES-001 | OPEN | `FU-CR152-001` 证据中 full pytest 有 4 个 unrelated process/design-surface failure，不能被误读为本 CR 新失败。 | host-orchestrator | 在验证和关闭摘要中保留 residual unrelated failure 分类，不声明 full suite green。 |
