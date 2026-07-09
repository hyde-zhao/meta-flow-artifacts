---
story_id: CR158-S04-evidence-handoff-typed-refs
title: Evidence index typed refs and Stage 2/3 handoff refs
story_slug: evidence-handoff-typed-refs
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
- docs/features/factor-research-loop/DESIGN.md
- docs/features/factor-research-loop/TEST-PLAN.md
- docs/features/factor-research-loop/TASKS.md
lld_policy:
  required_level: full-lld
  trigger_reasons:
  - evidence-index
  - handoff-contract
  - refs-only
  - cross-story-contract
  rationale: Extends CR157 refs-only evidence baseline for event/ML typed refs and
    Stage 2/3 handoff.
  waiver_reason: ''
  revisit_condition: If evidence requires large body embedding, return to CP5 or split
    a follow-up evidence CR.
  evidence_path: process/stories/CR158-S04-evidence-handoff-typed-refs-LLD.md
file_ownership:
  primary:
  - engine/strategy_type_adapters.py
  - tests/research/test_strategy_adapter_evidence_refs.py
  shared:
  - process/evidence/*.index.json
  merge_owner: CR158-S04-evidence-handoff-typed-refs
  forbidden:
  - .env
  - data/**
  - /home/hyde/data/**
lld_gate:
  design_evidence_type: full-lld
  design_evidence_path: process/stories/CR158-S04-evidence-handoff-typed-refs-LLD.md
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

把 event/ML adapter typed refs 接入 evidence index 和 Stage 2/3 handoff，保持 refs-only，不复制报告正文、event payload、模型二进制、diff 或 transcript。

## 技术说明

| 项目 | 内容 |
|---|---|
| 设计证据类型 | full-lld (see LLD file) |
| 设计依据 | HLD §5 Evidence extension、ADR-CR158-002、CR157 refs-only baseline |
| 文件影响 | planned evidence helper/tests; no large evidence body writes |
| 接口 / 数据 / 权限变化 | 新增 typed evidence refs；不新增 storage/publish 权限。 |
| 异常、失败与回退 | body_copy_count > 0 必须 blocked。 |
| 测试入口 | planned `tests/research/test_strategy_adapter_evidence_refs.py` |
| 风险与重访条件 | 若 consumer 需要正文，另行设计 evidence artifact ref，不写入 index body。 |

## 量化验收标准（acceptance_criteria）

- [ ] event evidence refs traceability coverage == 100% for P0 refs.
- [ ] ML evidence refs traceability coverage == 100% for P0 refs.
- [ ] body_copy_count == 0.
- [ ] handoff consumer reads public refs/status, not private extension payload.
