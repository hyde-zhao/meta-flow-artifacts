---
story_id: CR158-S02-event-strategy-adapter-extension
title: Event Strategy Adapter Typed Extension
story_slug: event-strategy-adapter-extension
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

# LLD: CR158-S02 - Event Strategy Adapter Typed Extension

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
| --- | --- | --- |
| HLD | docs/design/HLD-EVENT-ML-STRATEGY-ADAPTER.md | CP3 approved thin shared core + typed extensions, refs-only evidence and no-runtime boundary. |
| ADR | docs/design/ARCHITECTURE-DECISION-EVENT-ML-STRATEGY-ADAPTER.md | ADR-CR158-001..003 accepted at CP3. |
| Feature Matrix | process/docs/design/FEATURE-DESIGN-MATRIX.md#cr158-cp4-增量event--ml-strategy-adapter-unified-implementation | This Story requires full-lld and belongs to CR158 CP5 batch. |
| Development Plan | process/DEVELOPMENT-PLAN-CR158-EVENT-ML-STRATEGY-ADAPTER.yaml | Defines wave, dependencies, owner files, merge order and no-runtime authorization boundary. |
| Feature DESIGN | docs/features/factor-research-loop/DESIGN.md / docs/features/runtime-authorization-safety/DESIGN.md | FEAT-03 owns adapter contract; FEAT-07 owns no-runtime guard semantics. |

## 1. Goal

Define the EventAdapterExtension typed ref contract without adding live feed, listener, provider, gateway, runtime or event-store behavior.

## 2. Requirements（Functional / Non-Functional）

### 2.1 Functional

| ID | Requirement | Acceptance |
| --- | --- | --- |
| S02-R1 | Represent event source, event time, payload schema, alignment policy, signal output and blocked reason refs. | Event extension exposes at least six event-only ref groups. |
| S02-R2 | Validate missing P0 event refs as BLOCKED. | Missing source/alignment/signal refs never degrade to warning. |
| S02-R3 | Keep no-feed/no-listener boundary machine-verifiable. | real_event_feed and live_listener counters remain 0 in accepted fixture/static paths. |

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
| EventAdapterExtension | Event-only typed refs envelope. | Consumes S01 core extension slot. |
| validate_event_adapter_extension | Validate event P0 refs and operation counters. | Returns AdapterValidationResult. |
| event_adapter_summary | Public refs/status summary for evidence/handoff. | Does not expose event private payload body. |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
| --- | --- | --- |
| 修改 | engine/strategy_type_adapters.py | Add EventAdapterExtension, event required refs and event validator. |
| 创建 | tests/test_cr158_event_strategy_adapter.py | Add event extension positive, missing-ref, no-feed and isolation tests. |

## 5. 数据模型与持久化设计

| 对象 / 字段 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| EventAdapterExtension.event_source_ref | Mapping[str, Any] | required | Static event source metadata ref. |
| event_time_ref | Mapping[str, Any] | required | Event timestamp / availability time ref. |
| payload_schema_ref | Mapping[str, Any] | required | Schema ref only, no payload body. |
| alignment_policy_ref | Mapping[str, Any] | required | Time alignment policy ref. |
| signal_output_ref | Mapping[str, Any] | required | Output signal ref. |
| blocked_reason_refs | tuple[Mapping[str, Any], ...] | required when blocked | Machine-readable missing/forbidden reasons. |

无新增持久化、无真实数据读写、无 catalog / store / registry / publish 写入。

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
| --- | --- | --- | --- | --- |
| validate_event_adapter_extension(core, extension, operation_counts=None) | StrategyTypeAdapterCore + EventAdapterExtension | AdapterValidationResult | S02 tests / S04 evidence | Fails closed on missing event refs, wrong strategy_type or nonzero feed/listener counters. |

第 10 节为每个接口提供至少一条测试入口。

## 7. 核心处理流程

1. Validate S01 core first.
2. Confirm `strategy_type == event`.
3. Validate event source/time/schema/alignment/signal refs are nonblank ref objects.
4. Normalize feed/listener/provider/gateway counters and block any nonzero value.
5. Return refs-only event adapter summary for S04.

## 8. 技术设计细节

- Event payload data is never embedded; only `payload_schema_ref` and optional short metadata are allowed.
- Do not import or call `engine.event_strategy_admission_gate` at validation time; adapter extension is a lower-level typed ref contract.
- Tests may use static dictionaries only; no event feed or fixture file read is needed for the validator itself.

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
| complete event fixture | Core event + all event refs + counters zero. | Validate extension. | PASS. | unit |
| missing alignment policy | alignment_policy_ref absent. | Validate extension. | BLOCKED with field alignment_policy_ref. | unit |
| live listener counter | live_event_listener=1. | Validate extension. | BLOCKED. | unit |
| ML field leakage | event extension contains model_artifact_ref. | Validate extension. | BLOCKED or ignored with explicit reason per core policy. | unit |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
| --- | --- | --- | --- | --- |
| CR158-S02-T1 | 修改 | engine/strategy_type_adapters.py | Add event extension dataclass and validator. | tests/test_cr158_event_strategy_adapter.py |
| CR158-S02-T2 | 创建 | tests/test_cr158_event_strategy_adapter.py | Cover event P0 refs and no-feed counters. | pytest tests/test_cr158_event_strategy_adapter.py |

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 选项与推荐 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
| --- | --- | --- | --- | --- | --- | --- |
| LCQ-CR158-S02-001 | No blocking clarification item for this Story. | Use approved CP3/CP4 defaults; any new runtime/data need becomes separate CR. | N/A - no user answer required. | 接口 / 文件 owner / 测试 / 安全 / 文档 / 跨 Story 契约 | process/context/CP5-CR158-EVENT-ML-STRATEGY-ADAPTER-LLD-CONTEXT.yaml | If implementation requires real operation or new shared schema field. |

| 风险 / 难点 | 影响 | 缓解措施 / 预研建议 |
| --- | --- | --- |
| Event refs may be mistaken for real feed subscriptions. | Could weaken CP3/CP4 boundary if unchecked. | Name fields as refs, keep no-feed counters, release wording guard in S06. |

### OPEN / Spike 跟踪

| ID | 类型（OPEN / Spike） | 问题 | 下一动作 | 责任方 |
| --- | --- | --- | --- | --- |
| N/A | OPEN | No open or spike item blocks CP5. | N/A | host-orchestrator |

## 13. 回滚与发布策略

- 发布方式：CP5 approval only records design acceptance; CP6 may later implement local/static/fixture source and tests.
- 回滚触发条件：CP5 changes_requested / reject, CP6 design infeasibility, CP7 violation of no-runtime boundary, or evidence overclaim.
- 回滚动作：Remove EventAdapterExtension and tests; keep S01 core available for ML/evidence redesign.
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
