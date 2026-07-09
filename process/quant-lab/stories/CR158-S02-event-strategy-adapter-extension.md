---
story_id: CR158-S02-event-strategy-adapter-extension
title: Event strategy adapter typed extension
story_slug: event-strategy-adapter-extension
status: verified-with-risk
priority: P0
wave: CR158-W2-TYPED-EXTENSIONS
depends_on:
- CR158-S01-shared-adapter-core-contract
dependency_contracts:
- upstream: CR158-S01-shared-adapter-core-contract
  type: contract
  required_status_for_dev: lld-approved
feature_design_refs:
- docs/features/factor-research-loop/DESIGN.md
- docs/features/factor-research-loop/TEST-PLAN.md
- docs/features/factor-research-loop/TASKS.md
- docs/features/runtime-authorization-safety/DESIGN.md
lld_policy:
  required_level: full-lld
  trigger_reasons:
  - typed-extension
  - event-contract
  - security-boundary
  - failure-model
  rationale: Defines event-only typed refs and fail-closed behavior without real feed/listener/runtime.
  waiver_reason: ''
  revisit_condition: If event fields leak into shared core or ML extension, return
    to CP5 design clarification.
  evidence_path: process/stories/CR158-S02-event-strategy-adapter-extension-LLD.md
file_ownership:
  primary:
  - engine/strategy_type_adapters.py
  - tests/research/test_event_strategy_adapter.py
  shared:
  - tests/research/test_strategy_type_adapter_core.py
  merge_owner: CR158-S02-event-strategy-adapter-extension
  forbidden:
  - .env
  - data/**
  - /home/hyde/data/**
lld_gate:
  design_evidence_type: full-lld
  design_evidence_path: process/stories/CR158-S02-event-strategy-adapter-extension-LLD.md
  status: approved
  confirmed: true
  confirmed_at: '2026-07-05T18:05:00+08:00'
dev_gate:
  design_evidence_confirmed: true
  lld_confirmed: true
  dependencies_satisfied: true
  file_conflict_free: true
  implementation_allowed: local_static_fixture_only
created_at: '2026-07-05T17:35:00+08:00'
updated_at: '2026-07-05T17:55:00+08:00'
implementation_allowed: local_static_fixture_only
cp5_approved_at: '2026-07-05T18:05:00+08:00'
cp5_approved_by: user
cp6_result_ref: process/checks/CP6-CR158-EVENT-ML-STRATEGY-ADAPTER-IMPLEMENTATION.result.json
cp7_result_ref: process/checks/CP7-CR158-EVENT-ML-STRATEGY-ADAPTER-VERIFICATION.result.json
release_gate_ref: process/checkpoints/CP8-CR158-EVENT-ML-STRATEGY-ADAPTER-RELEASE-READINESS.md
verification_gate:
  status: pass_with_risk
  decision: PASS_WITH_RISK
  ref: process/checks/CP7-CR158-EVENT-ML-STRATEGY-ADAPTER-VERIFICATION.result.json
closure_status: closed-current-delivery-ready-with-risk
cp8_result_ref: process/checks/CP8-CR158-EVENT-ML-STRATEGY-ADAPTER-RELEASE-READINESS.result.json
closed_at: '2026-07-05T19:20:00+08:00'
---

## 目标

定义 event strategy adapter typed extension，覆盖 event source/time/payload schema/alignment policy/signal output/blocked reason refs，并保持 no-feed/no-listener/no-runtime。

## 开发上下文（dev_context）

- 输入文件：CR158 HLD、ADR-CR158-001/003、REQ-CR158-002、SC-CR158-P01/N01/N02
- 输出文件：planned `engine/strategy_type_adapters.py` event extension、planned `tests/research/test_event_strategy_adapter.py`
- 设计约束：真实 event feed、live listener、provider fetch、gateway call 计数必须为 0。

### 依赖与并行门控

| 上游 Story | 类型 | LLD 门控 | 开发门控 | 说明 |
|---|---|---|---|---|
| CR158-S01-shared-adapter-core-contract | contract | upstream_lld_or_story_contract_declared | upstream_contract_frozen | 消费 shared core extension slot。 |

## 技术说明

| 项目 | 内容 |
|---|---|
| 设计证据类型 | full-lld (see LLD file) |
| 设计依据 | HLD §5 EventAdapterExtension、FEATURE-DESIGN-MATRIX CR158 |
| 文件影响 | planned adapter contract module and event adapter tests |
| 接口 / 数据 / 权限变化 | 新增 event typed refs；不新增真实 event feed/runtime 权限。 |
| 异常、失败与回退 | 缺 event source ref、alignment policy ref 或 signal output ref 必须 blocked。 |
| 测试入口 | planned `tests/research/test_event_strategy_adapter.py` |
| 风险与重访条件 | 若需要真实 event feed，另起 runtime authorization CR。 |

## 量化验收标准（acceptance_criteria）

- [ ] Event extension 字段类别 >= 6。
- [ ] real_event_feed_call_count == 0。
- [ ] live_listener_count == 0。
- [ ] missing P0 event refs produce blocked status.
