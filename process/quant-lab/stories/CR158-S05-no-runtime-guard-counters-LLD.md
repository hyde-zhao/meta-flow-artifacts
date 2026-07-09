---
story_id: CR158-S05-no-runtime-guard-counters
title: No-Runtime Guard Counters and Failure Reasons
story_slug: no-runtime-guard-counters
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

# LLD: CR158-S05 - No-Runtime Guard Counters and Failure Reasons

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
| --- | --- | --- |
| HLD | docs/design/HLD-EVENT-ML-STRATEGY-ADAPTER.md | CP3 approved thin shared core + typed extensions, refs-only evidence and no-runtime boundary. |
| ADR | docs/design/ARCHITECTURE-DECISION-EVENT-ML-STRATEGY-ADAPTER.md | ADR-CR158-001..003 accepted at CP3. |
| Feature Matrix | process/docs/design/FEATURE-DESIGN-MATRIX.md#cr158-cp4-增量event--ml-strategy-adapter-unified-implementation | This Story requires full-lld and belongs to CR158 CP5 batch. |
| Development Plan | process/DEVELOPMENT-PLAN-CR158-EVENT-ML-STRATEGY-ADAPTER.yaml | Defines wave, dependencies, owner files, merge order and no-runtime authorization boundary. |
| Feature DESIGN | docs/features/factor-research-loop/DESIGN.md / docs/features/runtime-authorization-safety/DESIGN.md | FEAT-03 owns adapter contract; FEAT-07 owns no-runtime guard semantics. |

## 1. Goal

Define the CR158 adapter forbidden operation counter report and fail-closed blocked reasons for feed, training, provider, lake, NAS, credential, runtime, trading, registry, publish, external framework and Git remote signals.

## 2. Requirements（Functional / Non-Functional）

### 2.1 Functional

| ID | Requirement | Acceptance |
| --- | --- | --- |
| S05-R1 | Cover at least twelve forbidden operation classes. | Counter tuple includes feed/listener/training/service/provider/lake/NAS/credential/runtime/trading/registry/publish/external framework/Git remote families. |
| S05-R2 | Block any nonzero forbidden counter. | Adapter readiness and handoff readiness become BLOCKED. |
| S05-R3 | Keep CP7/CP8 wording aligned with no-runtime boundary. | Validation result exposes blocked reason refs and limitation strings for docs/release review. |

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
| CR158_FORBIDDEN_OPERATION_COUNTERS | Canonical counter names for adapter scope. | S05 owns the tuple. |
| AdapterOperationCounterReport | Counter report dataclass/mapping. | All names normalized to int. |
| zero_adapter_operation_counts | Factory for all-zero report. | Used by S01-S03 tests. |
| validate_adapter_operation_counters | Fail-closed counter validator. | Blocks any nonzero value. |
| adapter_no_runtime_limitations | Canonical limitation strings for CP7/CP8 wording. | Consumed by S06. |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
| --- | --- | --- |
| 修改 | engine/strategy_type_adapters.py | Add CR158 counter tuple, report dataclass, zero factory and validation helper. |
| 创建 | tests/research/test_strategy_adapter_no_runtime_guard.py | Add zero, nonzero, missing/default and limitation wording tests. |

## 5. 数据模型与持久化设计

| 对象 / 字段 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| AdapterOperationCounterReport.operation_counts | Mapping[str, int] | all counter names present after normalization | Missing values default to 0 only through normalizer. |
| nonzero_counters | Mapping[str, int] | derived | Any nonzero is blocker. |
| blocked_reason_refs | tuple[Mapping[str, Any], ...] | required when nonzero | Names counter and unlock condition. |
| limitations | tuple[str, ...] | required | no_feed, no_training, no_registry, no_runtime, no_publish, no_trading. |

无新增持久化、无真实数据读写、无 catalog / store / registry / publish 写入。

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
| --- | --- | --- | --- | --- |
| zero_adapter_operation_counts() | none | dict[str, int] | S01-S04 tests | All counters zero. |
| validate_adapter_operation_counters(operation_counts) | mapping | AdapterValidationResult | S01-S04 validators | Blocks nonzero counters. |
| adapter_no_runtime_summary(report) | counter report | Mapping[str, Any] | S06 docs/release guard | Public limitation summary. |

第 10 节为每个接口提供至少一条测试入口。

## 7. 核心处理流程

1. Normalize all known CR158 counter names from caller input.
2. Default missing counters to 0 in normalizer, but keep test coverage that required names are present in summary.
3. Collect every nonzero counter into blocked reason refs.
4. Return PASS only when nonzero counter count is 0.
5. Expose limitation strings to CP7/CP8 wording checks.

## 8. 技术设计细节

- Counter names should reuse existing FEAT-07 / CR152 / CR153 names where present to reduce drift.
- Do not inspect `.env`, filesystem, provider sessions or runtime objects to infer counters.
- The validator proves only no forbidden operation was reported in fixture/static path; it is not runtime authorization.

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
| zero counters | Use zero factory. | Validate counters. | PASS and nonzero count 0. | unit |
| single nonzero feed | real_event_feed=1. | Validate counters. | BLOCKED with unlock condition. | unit |
| single nonzero registry | model_registry_write=1. | Validate counters. | BLOCKED. | unit |
| coverage count | Inspect counter tuple. | Count operation families. | >= 12 classes covered. | unit |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
| --- | --- | --- | --- | --- |
| CR158-S05-T1 | 修改 | engine/strategy_type_adapters.py | Add counter report, zero factory and validator. | tests/research/test_strategy_adapter_no_runtime_guard.py |
| CR158-S05-T2 | 创建 | tests/research/test_strategy_adapter_no_runtime_guard.py | Cover forbidden counter semantics and limitation summary. | pytest tests/research/test_strategy_adapter_no_runtime_guard.py |

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 选项与推荐 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
| --- | --- | --- | --- | --- | --- | --- |
| LCQ-CR158-S05-001 | No blocking clarification item for this Story. | Use approved CP3/CP4 defaults; any new runtime/data need becomes separate CR. | N/A - no user answer required. | 接口 / 文件 owner / 测试 / 安全 / 文档 / 跨 Story 契约 | process/context/CP5-CR158-EVENT-ML-STRATEGY-ADAPTER-LLD-CONTEXT.yaml | If implementation requires real operation or new shared schema field. |

| 风险 / 难点 | 影响 | 缓解措施 / 预研建议 |
| --- | --- | --- |
| A future reviewer may interpret zero counters as proof that runtime validation happened. | Could weaken CP3/CP4 boundary if unchecked. | LLD, CP5, CP7 and CP8 wording must state counters prove fixture/static no-runtime boundary only. |

### OPEN / Spike 跟踪

| ID | 类型（OPEN / Spike） | 问题 | 下一动作 | 责任方 |
| --- | --- | --- | --- | --- |
| N/A | OPEN | No open or spike item blocks CP5. | N/A | host-orchestrator |

## 13. 回滚与发布策略

- 发布方式：CP5 approval only records design acceptance; CP6 may later implement local/static/fixture source and tests.
- 回滚触发条件：CP5 changes_requested / reject, CP6 design infeasibility, CP7 violation of no-runtime boundary, or evidence overclaim.
- 回滚动作：Remove S05 counter helpers; S01-S04 cannot claim adapter readiness until replacement guard is approved.
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
