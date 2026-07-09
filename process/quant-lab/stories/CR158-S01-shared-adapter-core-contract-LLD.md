---
story_id: CR158-S01-shared-adapter-core-contract
title: Shared Adapter Core and Validation Result Contract
story_slug: shared-adapter-core-contract
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

# LLD: CR158-S01 - Shared Adapter Core and Validation Result Contract

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
| --- | --- | --- |
| HLD | docs/design/HLD-EVENT-ML-STRATEGY-ADAPTER.md | CP3 approved thin shared core + typed extensions, refs-only evidence and no-runtime boundary. |
| ADR | docs/design/ARCHITECTURE-DECISION-EVENT-ML-STRATEGY-ADAPTER.md | ADR-CR158-001..003 accepted at CP3. |
| Feature Matrix | process/docs/design/FEATURE-DESIGN-MATRIX.md#cr158-cp4-增量event--ml-strategy-adapter-unified-implementation | This Story requires full-lld and belongs to CR158 CP5 batch. |
| Development Plan | process/DEVELOPMENT-PLAN-CR158-EVENT-ML-STRATEGY-ADAPTER.yaml | Defines wave, dependencies, owner files, merge order and no-runtime authorization boundary. |
| Feature DESIGN | docs/features/factor-research-loop/DESIGN.md / docs/features/runtime-authorization-safety/DESIGN.md | FEAT-03 owns adapter contract; FEAT-07 owns no-runtime guard semantics. |

## 1. Goal

Define the thin shared StrategyTypeAdapterCore and AdapterValidationResult contract consumed by all CR158 event, ML, evidence, guard and release stories.

## 2. Requirements（Functional / Non-Functional）

### 2.1 Functional

| ID | Requirement | Acceptance |
| --- | --- | --- |
| S01-R1 | Represent at least seven shared core field groups: adapter id/type, input refs, output signal refs, evidence refs, blocked reason refs, authorization flags and handoff refs. | Shared core schema includes all seven groups and excludes event-only / ML-only private fields. |
| S01-R2 | Expose a validation result that can represent PASS, BLOCKED, missing-ref and forbidden-operation reasons. | AdapterValidationResult always carries status, blocked reasons, evidence refs and operation counters. |
| S01-R3 | Keep shared core local/static/fixture only. | No filesystem, credential, provider, lake, runtime, training, registry, publish or trading operation is introduced. |

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
| StrategyTypeAdapterCore | Dataclass / mapping contract for shared refs and authorization flags. | S01 owns the root contract; S02/S03 consume extension slots. |
| AdapterValidationResult | Typed validation status, blocked reason refs, evidence refs and counter report linkage. | Unknown status fails closed to BLOCKED. |
| AdapterBlockedReason | Machine-readable reason object for missing refs, invalid type, forbidden operation or overclaim. | Used by S02-S05. |
| normalize_adapter_core | Normalize dataclass/mapping fixture payloads into a JSON-safe core object. | No I/O; pure mapping normalization. |
| validate_strategy_type_adapter_core | Validate required shared groups and extension isolation. | Blocks event-only / ML-only leakage into core. |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
| --- | --- | --- |
| 创建 | engine/strategy_type_adapters.py | Add CR158 shared dataclasses, status enum, normalization and core validation helpers. |
| 创建 | tests/research/test_strategy_type_adapter_core.py | Add contract tests for required groups, extension isolation, blocked reasons and zero default counters. |

## 5. 数据模型与持久化设计

| 对象 / 字段 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| StrategyTypeAdapterCore.adapter_id | str | nonblank | Stable adapter identifier. |
| StrategyTypeAdapterCore.strategy_type | Literal[event, ml] | required | Controls which typed extension is valid. |
| input_refs/output_signal_refs/evidence_refs/handoff_refs | tuple[Mapping[str, Any], ...] | refs-only | Refs must point to fixture/static evidence, not embedded body. |
| authorization_flags | Mapping[str, bool] | all runtime flags false by default | Must include no_runtime, no_feed, no_training, no_registry, no_publish. |
| AdapterValidationResult.status | str | PASS|FAIL|NEEDS_REVIEW|BLOCKED | Unknown status coerces to BLOCKED. |
| AdapterValidationResult.operation_counts | Mapping[str, int] | all forbidden counters normalized | Consumed by S05. |

无新增持久化、无真实数据读写、无 catalog / store / registry / publish 写入。

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
| --- | --- | --- | --- | --- |
| validate_strategy_type_adapter_core(core, extension=None, operation_counts=None) | core mapping/dataclass, optional extension and counters | AdapterValidationResult | S01/S02/S03 tests | Fails closed on missing shared refs, private-field leakage or nonzero counters. |
| adapter_core_summary(result_or_core) | core/result mapping | JSON-safe summary | S04/S06 | Returns refs/status/counters without private extension body. |

第 10 节为每个接口提供至少一条测试入口。

## 7. 核心处理流程

1. Normalize incoming fixture/static core payload.
2. Verify required shared field groups are present.
3. Reject event-only or ML-only private keys at shared-core level.
4. Normalize operation counters through CR158 counter set with default zero values.
5. Return AdapterValidationResult with explicit blocked reasons or PASS.

## 8. 技术设计细节

- Use frozen dataclasses with `slots=True`, matching existing engine contract style.
- Reuse `engine.serialization.json_safe` / `as_mapping` if available; do not create another serialization helper.
- Keep extension payload opaque at core level except for `strategy_type` compatibility.
- Expose tuple constants for required fields and private-field denylist so S02/S03 can test against the same source of truth.

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
| complete core fixture | All seven shared groups present, event/ML private keys absent. | Call validator. | PASS with zero forbidden counters. | unit |
| event private key in core | Core includes event_payload_schema_ref. | Call validator. | BLOCKED with field name. | unit |
| ML private key in core | Core includes model_artifact_ref. | Call validator. | BLOCKED with field name. | unit |
| nonzero counter | Core is otherwise complete, `real_event_feed=1`. | Call validator. | BLOCKED via S05 counter policy. | unit |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
| --- | --- | --- | --- | --- |
| CR158-S01-T1 | 创建 | engine/strategy_type_adapters.py | Add status enum, blocked reason, core dataclass, validation result and zero counter constants. | tests/research/test_strategy_type_adapter_core.py |
| CR158-S01-T2 | 创建 | tests/research/test_strategy_type_adapter_core.py | Cover complete, missing, private-field leakage and nonzero-counter paths. | pytest tests/research/test_strategy_type_adapter_core.py |

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 选项与推荐 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
| --- | --- | --- | --- | --- | --- | --- |
| LCQ-CR158-S01-001 | No blocking clarification item for this Story. | Use approved CP3/CP4 defaults; any new runtime/data need becomes separate CR. | N/A - no user answer required. | 接口 / 文件 owner / 测试 / 安全 / 文档 / 跨 Story 契约 | process/context/CP5-CR158-EVENT-ML-STRATEGY-ADAPTER-LLD-CONTEXT.yaml | If implementation requires real operation or new shared schema field. |

| 风险 / 难点 | 影响 | 缓解措施 / 预研建议 |
| --- | --- | --- |
| Core can become a fat schema if extension fields are added for convenience. | Could weaken CP3/CP4 boundary if unchecked. | Keep private-field denylist in tests and CP5 decision; any added shared field needs CP5 design delta. |

### OPEN / Spike 跟踪

| ID | 类型（OPEN / Spike） | 问题 | 下一动作 | 责任方 |
| --- | --- | --- | --- | --- |
| N/A | OPEN | No open or spike item blocks CP5. | N/A | host-orchestrator |

## 13. 回滚与发布策略

- 发布方式：CP5 approval only records design acceptance; CP6 may later implement local/static/fixture source and tests.
- 回滚触发条件：CP5 changes_requested / reject, CP6 design infeasibility, CP7 violation of no-runtime boundary, or evidence overclaim.
- 回滚动作：Remove `engine/strategy_type_adapters.py` and S01 tests; downstream S02-S05 remain blocked until a replacement core LLD is approved.
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
