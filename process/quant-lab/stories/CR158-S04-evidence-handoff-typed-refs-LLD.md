---
story_id: CR158-S04-evidence-handoff-typed-refs
title: Evidence Index Typed Refs and Stage 2/3 Handoff Refs
story_slug: evidence-handoff-typed-refs
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

# LLD: CR158-S04 - Evidence Index Typed Refs and Stage 2/3 Handoff Refs

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
| --- | --- | --- |
| HLD | docs/design/HLD-EVENT-ML-STRATEGY-ADAPTER.md | CP3 approved thin shared core + typed extensions, refs-only evidence and no-runtime boundary. |
| ADR | docs/design/ARCHITECTURE-DECISION-EVENT-ML-STRATEGY-ADAPTER.md | ADR-CR158-001..003 accepted at CP3. |
| Feature Matrix | process/docs/design/FEATURE-DESIGN-MATRIX.md#cr158-cp4-增量event--ml-strategy-adapter-unified-implementation | This Story requires full-lld and belongs to CR158 CP5 batch. |
| Development Plan | process/DEVELOPMENT-PLAN-CR158-EVENT-ML-STRATEGY-ADAPTER.yaml | Defines wave, dependencies, owner files, merge order and no-runtime authorization boundary. |
| Feature DESIGN | docs/features/factor-research-loop/DESIGN.md / docs/features/runtime-authorization-safety/DESIGN.md | FEAT-03 owns adapter contract; FEAT-07 owns no-runtime guard semantics. |

## 1. Goal

Connect event/ML adapter validation summaries to refs-only evidence index records and Stage 2/3 handoff refs without copying payloads, reports, binaries, diffs or transcripts.

## 2. Requirements（Functional / Non-Functional）

### 2.1 Functional

| ID | Requirement | Acceptance |
| --- | --- | --- |
| S04-R1 | Represent event and ML adapter evidence as typed refs only. | P0 event/ML refs are traceable and body_copy_count stays 0. |
| S04-R2 | Expose public handoff refs and status for Stage 2/3 consumers. | Consumers read refs/status/blocked reasons, not private extension payloads. |
| S04-R3 | Fail closed if evidence body copying is detected. | Any body_copy_count > 0 returns BLOCKED. |

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
| AdapterTypedEvidenceRef | Evidence item shape for adapter validation summaries. | Refs-only; short metadata only. |
| build_adapter_evidence_refs | Create evidence refs from S02/S03 validation results. | No file write; returns mappings for CP6 evidence. |
| adapter_handoff_summary | Public Stage 2/3 handoff summary. | Contains adapter_id, strategy_type, status, evidence refs and blocked reason refs. |
| validate_adapter_evidence_refs | Check body_copy_count and required refs. | Blocks embedded body attempts. |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
| --- | --- | --- |
| 修改 | engine/strategy_type_adapters.py | Add typed evidence ref, evidence builder, handoff summary and refs-only validator. |
| 创建 | tests/research/test_strategy_adapter_evidence_refs.py | Add refs-only, body-copy blocked and handoff public surface tests. |

## 5. 数据模型与持久化设计

| 对象 / 字段 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| AdapterTypedEvidenceRef.ref_id | str | nonblank | Stable evidence ref id. |
| kind | Literal[event_adapter, ml_adapter, adapter_validation, adapter_handoff] | required | Typed evidence category. |
| path_or_id | str | required | Ref path or object id only. |
| hash | str | optional but recommended | Short checksum / provenance marker. |
| status | str | PASS|FAIL|NEEDS_REVIEW|BLOCKED | Mirrors validation status. |
| body_copy_count | int | must equal 0 | Any nonzero value blocks. |
| private_payload_included | bool | must be false | Prevents event payload/model body embedding. |

无新增持久化、无真实数据读写、无 catalog / store / registry / publish 写入。

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
| --- | --- | --- | --- | --- |
| build_adapter_evidence_refs(validation_results) | event/ML AdapterValidationResult sequence | tuple[AdapterTypedEvidenceRef, ...] | CP6 evidence builder | Returns refs only; no persistence. |
| adapter_handoff_summary(core, evidence_refs) | core + typed evidence refs | Mapping[str, Any] | Stage 2/3 handoff consumer | Public refs/status only. |
| validate_adapter_evidence_refs(refs) | typed evidence refs | AdapterValidationResult | S04 tests | Blocks missing refs or body_copy_count > 0. |

第 10 节为每个接口提供至少一条测试入口。

## 7. 核心处理流程

1. Consume S02/S03 validation result summaries.
2. Create typed evidence refs for adapter validation, output signal refs and blocked reason refs.
3. Validate body_copy_count is 0 and no private payload/model body fields are present.
4. Build public handoff summary with adapter status and evidence refs.
5. Return blocked if a consumer attempts to require private extension payload.

## 8. 技术设计细节

- Evidence index remains metadata-only; full reports stay behind artifact refs.
- Stage 2/3 handoff reads `adapter_handoff_summary`, not EventAdapterExtension / MLAdapterExtension internals.
- If a downstream consumer needs body content, create a separate artifact ref and retention policy; do not store it in evidence index.

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
| event evidence refs | Event validation result has P0 refs. | Build evidence refs. | All P0 refs present, body_copy_count 0. | unit |
| ML evidence refs | ML validation result has P0 refs. | Build evidence refs. | All P0 refs present, body_copy_count 0. | unit |
| body copy attempt | typed evidence includes body_copy_count=1. | Validate refs. | BLOCKED. | unit |
| handoff hides private payload | Handoff summary built from event/ML results. | Inspect keys. | No event payload body or model binary fields. | unit |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
| --- | --- | --- | --- | --- |
| CR158-S04-T1 | 修改 | engine/strategy_type_adapters.py | Add typed evidence ref and handoff summary helpers. | tests/research/test_strategy_adapter_evidence_refs.py |
| CR158-S04-T2 | 创建 | tests/research/test_strategy_adapter_evidence_refs.py | Cover refs-only and handoff public surface. | pytest tests/research/test_strategy_adapter_evidence_refs.py |

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 选项与推荐 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
| --- | --- | --- | --- | --- | --- | --- |
| LCQ-CR158-S04-001 | No blocking clarification item for this Story. | Use approved CP3/CP4 defaults; any new runtime/data need becomes separate CR. | N/A - no user answer required. | 接口 / 文件 owner / 测试 / 安全 / 文档 / 跨 Story 契约 | process/context/CP5-CR158-EVENT-ML-STRATEGY-ADAPTER-LLD-CONTEXT.yaml | If implementation requires real operation or new shared schema field. |

| 风险 / 难点 | 影响 | 缓解措施 / 预研建议 |
| --- | --- | --- |
| Evidence index can become a hidden report store if body fields are allowed. | Could weaken CP3/CP4 boundary if unchecked. | body_copy_count and private_payload_included checks are hard blockers. |

### OPEN / Spike 跟踪

| ID | 类型（OPEN / Spike） | 问题 | 下一动作 | 责任方 |
| --- | --- | --- | --- | --- |
| N/A | OPEN | No open or spike item blocks CP5. | N/A | host-orchestrator |

## 13. 回滚与发布策略

- 发布方式：CP5 approval only records design acceptance; CP6 may later implement local/static/fixture source and tests.
- 回滚触发条件：CP5 changes_requested / reject, CP6 design infeasibility, CP7 violation of no-runtime boundary, or evidence overclaim.
- 回滚动作：Remove S04 evidence/handoff helpers; evidence consumers must wait for redesigned refs-only contract.
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
