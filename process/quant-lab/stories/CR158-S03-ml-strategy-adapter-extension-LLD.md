---
story_id: CR158-S03-ml-strategy-adapter-extension
title: ML Strategy Adapter Typed Extension
story_slug: ml-strategy-adapter-extension
lld_version: '1.0'
tier: M
status: dev-ready
confirmed: false
created_by: host-orchestrator-inline
created_at: '2026-07-05T17:55:00+08:00'
confirmed_by: ''
confirmed_at: ''
shared_fragments: []
feature_design_refs:
- docs/features/factor-research-loop/DESIGN.md
- docs/features/runtime-authorization-safety/DESIGN.md
lld_policy:
  required_level: full-lld
  trigger_reasons: []
  rationale: ''
open_items: 0
implementation_allowed: local_static_fixture_only
cp5_review_required: true
source_hld: docs/design/HLD-EVENT-ML-STRATEGY-ADAPTER.md
source_adr: docs/design/ARCHITECTURE-DECISION-EVENT-ML-STRATEGY-ADAPTER.md
cp5_approved_at: '2026-07-05T18:05:00+08:00'
cp5_approved_by: user
lld_gate:
  status: approved
  confirmed: true
  confirmed_at: '2026-07-05T18:05:00+08:00'
dev_gate:
  design_evidence_confirmed: true
  lld_confirmed: true
  dependencies_satisfied: true
  implementation_allowed: local_static_fixture_only
---

# LLD: CR158-S03 - ML Strategy Adapter Typed Extension

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
| --- | --- | --- |
| HLD | docs/design/HLD-EVENT-ML-STRATEGY-ADAPTER.md | CP3 approved thin shared core + typed extensions, refs-only evidence and no-runtime boundary. |
| ADR | docs/design/ARCHITECTURE-DECISION-EVENT-ML-STRATEGY-ADAPTER.md | ADR-CR158-001..003 accepted at CP3. |
| Feature Matrix | process/docs/design/FEATURE-DESIGN-MATRIX.md#cr158-cp4-增量event--ml-strategy-adapter-unified-implementation | This Story requires full-lld and belongs to CR158 CP5 batch. |
| Development Plan | process/DEVELOPMENT-PLAN-CR158-EVENT-ML-STRATEGY-ADAPTER.yaml | Defines wave, dependencies, owner files, merge order and no-runtime authorization boundary. |
| Feature DESIGN | docs/features/factor-research-loop/DESIGN.md / docs/features/runtime-authorization-safety/DESIGN.md | FEAT-03 owns adapter contract; FEAT-07 owns no-runtime guard semantics. |

## 1. Goal

Define the MLAdapterExtension typed ref contract without real model training, external model service calls, registry writes or prediction-store writes.

## 2. Requirements（Functional / Non-Functional）

### 2.1 Functional

| ID | Requirement | Acceptance |
| --- | --- | --- |
| S03-R1 | Represent training snapshot, feature set, label policy, model artifact, validation report, prediction signal and blocked reason refs. | ML extension exposes at least seven ML-only ref groups. |
| S03-R2 | Validate missing P0 ML refs as BLOCKED. | Missing artifact/report/prediction refs never degrade to warning. |
| S03-R3 | Keep no-training/no-registry boundary machine-verifiable. | real_training and model_registry_write counters remain 0 in accepted fixture/static paths. |

### 2.2 Non-Functional

| 维度 | 约束 | 验收 |
| --- | --- | --- |
| Safety | No real feed/training/provider/lake/NAS/credential/QMT/runtime/simulation/paper/live/trading/broker/store/catalog/registry/publish/external-framework/Git remote action is authorized. | Forbidden counters and blocked reasons are machine-readable. |
| Traceability | All public outputs are refs/status/short metadata. | P0 refs are traceable from adapter to evidence/handoff. |
| Compatibility | Follow existing dataclass + mapping + `to_dict` engine contract style. | Existing event/ML admission modules are not broken. |
| Testability | Every interface has unit fixture tests. | Tests use static mappings and no runtime operations. |

## 3. 模块拆分与职责

| 模块 / 文件组 | 职责 | 说明 |
| --- | --- | --- |
| MLAdapterExtension | ML-only typed refs envelope. | Consumes S01 core extension slot. |
| validate_ml_adapter_extension | Validate ML P0 refs and operation counters. | Returns AdapterValidationResult. |
| ml_adapter_summary | Public refs/status summary for evidence/handoff. | Does not expose model binary or private metadata body. |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
| --- | --- | --- |
| 修改 | engine/strategy_type_adapters.py | Add MLAdapterExtension, ML required refs and ML validator. |
| 创建 | tests/research/test_ml_strategy_adapter.py | Add ML extension positive, missing-ref, no-training and isolation tests. |

## 5. 数据模型与持久化设计

| 对象 / 字段 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| MLAdapterExtension.training_snapshot_ref | Mapping[str, Any] | required | Static training snapshot ref; not a training action. |
| feature_set_ref | Mapping[str, Any] | required | Feature set metadata ref. |
| label_policy_ref | Mapping[str, Any] | required | Label policy ref. |
| model_artifact_ref | Mapping[str, Any] | required | Artifact ref only, no binary body. |
| validation_report_ref | Mapping[str, Any] | required | Validation report ref. |
| prediction_signal_ref | Mapping[str, Any] | required | Prediction signal ref. |
| blocked_reason_refs | tuple[Mapping[str, Any], ...] | required when blocked | Machine-readable missing/forbidden reasons. |

无新增持久化、无真实数据读写、无 catalog / store / registry / publish 写入。

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
| --- | --- | --- | --- | --- |
| validate_ml_adapter_extension(core, extension, operation_counts=None) | StrategyTypeAdapterCore + MLAdapterExtension | AdapterValidationResult | S03 tests / S04 evidence | Fails closed on missing ML refs, wrong strategy_type or nonzero training/registry counters. |

第 10 节为每个接口提供至少一条测试入口。

## 7. 核心处理流程

1. Validate S01 core first.
2. Confirm `strategy_type == ml`.
3. Validate training/feature/label/artifact/report/prediction refs.
4. Normalize training/service/registry/store counters and block any nonzero value.
5. Return refs-only ML adapter summary for S04.

## 8. 技术设计细节

- `model_artifact_ref` is a local/static ref, not a registry entry or promotion proof.
- Do not call `evaluate_ml_strategy_admission_gate`; adapter extension is a typed ref layer feeding later evidence.
- Triple-barrier or meta-label behavior remains governed by existing ML gate; CR158 does not add training algorithms.

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
| --- | --- | --- |
| 安全 | No real feed/training/provider/lake/NAS/credential/QMT/runtime/simulation/paper/live/trading/broker/store/catalog/registry/publish/external-framework/Git remote action is authorized. | CP5 result / CP7 tests / CP8 wording guard. |
| 权限 | CP5 approval may only authorize later local source/test implementation; runtime authorization remains false. | GATE-LEDGER and CP5 Decision Brief. |
| 性能 | Pure mapping/dataclass validation; O(number of refs + counters). | No dataframe, network or filesystem operations in validator. |
| 可观测性 | Validation result exposes status, blocked reasons, evidence refs and operation counts. | Unit tests inspect summary payloads. |

## 10. 测试设计

| 测试场景 | 前置条件 | 操作 | 预期结果 | 验证方式 |
| --- | --- | --- | --- | --- |
| complete ML fixture | Core ml + all ML refs + counters zero. | Validate extension. | PASS. | unit |
| missing validation report | validation_report_ref absent. | Validate extension. | BLOCKED. | unit |
| registry counter | model_registry_write=1. | Validate extension. | BLOCKED. | unit |
| event field leakage | ML extension contains event_source_ref. | Validate extension. | BLOCKED or explicit private-field reason. | unit |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
| --- | --- | --- | --- | --- |
| CR158-S03-T1 | 修改 | engine/strategy_type_adapters.py | Add ML extension dataclass and validator. | tests/research/test_ml_strategy_adapter.py |
| CR158-S03-T2 | 创建 | tests/research/test_ml_strategy_adapter.py | Cover ML P0 refs and no-training counters. | pytest tests/research/test_ml_strategy_adapter.py |

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 选项与推荐 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
| --- | --- | --- | --- | --- | --- | --- |
| LCQ-CR158-S03-001 | No blocking clarification item for this Story. | Use approved CP3/CP4 defaults; any new runtime/data need becomes separate CR. | N/A - no user answer required. | 接口 / 文件 owner / 测试 / 安全 / 文档 / 跨 Story 契约 | process/context/CP5-CR158-EVENT-ML-STRATEGY-ADAPTER-LLD-CONTEXT.yaml | If implementation requires real operation or new shared schema field. |

| 风险 / 难点 | 影响 | 缓解措施 / 预研建议 |
| --- | --- | --- |
| ML artifact refs may be overclaimed as trained or registry-promoted models. | Could weaken CP3/CP4 boundary if unchecked. | Validation and release wording must state fixture/static artifact refs only. |

### OPEN / Spike 跟踪

| ID | 类型（OPEN / Spike） | 问题 | 下一动作 | 责任方 |
| --- | --- | --- | --- | --- |
| N/A | OPEN | No open or spike item blocks CP5. | N/A | host-orchestrator |

## 13. 回滚与发布策略

- 发布方式：CP5 approval only records design acceptance; CP6 may later implement local/static/fixture source and tests.
- 回滚触发条件：CP5 changes_requested / reject, CP6 design infeasibility, CP7 violation of no-runtime boundary, or evidence overclaim.
- 回滚动作：Remove MLAdapterExtension and tests; keep S01/S02 unaffected.
- 不授权项：No real feed/training/provider/lake/NAS/credential/QMT/runtime/simulation/paper/live/trading/broker/store/catalog/registry/publish/external-framework/Git remote action is authorized.

## 14. Definition of Done

- [ ] 14 个章节全部填写完成。
- [ ] 文件影响范围、接口、测试与实施步骤可直接指导编码。
- [ ] 实现灰区与取舍记录已回填，或显式写明无阻断项。
- [ ] `confirmed=false` 时不进入实现。
- [ ] CP5 人工确认意见已收敛后才允许更新 Story dev gate。
- [ ] OPEN / Spike 已清点或显式写“无”。

## 人工确认区

CP5 reviewer should confirm this Story design evidence as part of `process/checkpoints/CP5-CR158-EVENT-ML-STRATEGY-ADAPTER-LLD-BATCH.md`. A reply of `approve` to the CP5 gate accepts this LLD as implementation guidance, but does not authorize runtime, real data, registry, publish, trading or Git remote operations.
