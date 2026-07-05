---
story_id: CR158-S05-no-runtime-guard-counters
title: No-runtime guard counters and failure reasons
story_slug: no-runtime-guard-counters
status: verified-with-risk
priority: P0
wave: CR158-W3-EVIDENCE-SAFETY
depends_on:
- CR158-S01-shared-adapter-core-contract
- CR158-S02-event-strategy-adapter-extension
- CR158-S03-ml-strategy-adapter-extension
dependency_contracts:
- upstream: CR158-S01-shared-adapter-core-contract
  type: contract
  required_status_for_dev: lld-approved
- upstream: CR158-S02-event-strategy-adapter-extension
  type: contract
  required_status_for_dev: lld-approved
- upstream: CR158-S03-ml-strategy-adapter-extension
  type: contract
  required_status_for_dev: lld-approved
feature_design_refs:
- docs/features/runtime-authorization-safety/DESIGN.md
- docs/features/runtime-authorization-safety/TEST-PLAN.md
- docs/features/runtime-authorization-safety/TASKS.md
lld_policy:
  required_level: full-lld
  trigger_reasons:
  - security-boundary
  - runtime-authorization
  - forbidden-counter
  - failure-model
  rationale: Defines adapter forbidden operation counter report and fail-closed no-runtime
    semantics.
  waiver_reason: ''
  revisit_condition: If any real operation is required, create a separate runtime
    authorization CR.
  evidence_path: process/stories/CR158-S05-no-runtime-guard-counters-LLD.md
file_ownership:
  primary:
  - engine/strategy_type_adapters.py
  - tests/test_cr158_adapter_no_runtime_guard.py
  shared:
  - docs/features/runtime-authorization-safety/DESIGN.md
  merge_owner: CR158-S05-no-runtime-guard-counters
  forbidden:
  - .env
  - data/**
  - /home/hyde/data/**
lld_gate:
  design_evidence_type: full-lld
  design_evidence_path: process/stories/CR158-S05-no-runtime-guard-counters-LLD.md
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

定义 CR158 adapter no-runtime guard counter report，覆盖真实 feed、训练、provider/lake/NAS/credential、runtime、trading、registry、publish、external framework 和 Git remote write 的 fail-closed 行为。

## 技术说明

| 项目 | 内容 |
|---|---|
| 设计证据类型 | full-lld (see LLD file) |
| 设计依据 | ADR-CR158-003、FEAT-07 CR158 Adapter No-Runtime Guard |
| 文件影响 | planned adapter safety counters and tests |
| 接口 / 数据 / 权限变化 | 新增 local counter report；不新增任何真实操作授权。 |
| 异常、失败与回退 | 任一 forbidden counter 非 0 -> blocked。 |
| 测试入口 | planned `tests/test_cr158_adapter_no_runtime_guard.py` |
| 风险与重访条件 | 用户需要真实验证时回退到 runtime authorization CR。 |

## 量化验收标准（acceptance_criteria）

- [ ] forbidden counter set covers at least 12 operation classes.
- [ ] all forbidden counters == 0 for accepted fixture/static paths.
- [ ] any forbidden counter > 0 blocks adapter readiness.
- [ ] CP7/CP8 wording states no runtime authorization.
