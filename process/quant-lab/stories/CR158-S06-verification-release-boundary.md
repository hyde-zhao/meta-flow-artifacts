---
story_id: CR158-S06-verification-release-boundary
title: Verification matrix and release wording boundary
story_slug: verification-release-boundary
status: verified-with-risk
priority: P1
wave: CR158-W4-VERIFICATION-RELEASE
depends_on:
- CR158-S04-evidence-handoff-typed-refs
- CR158-S05-no-runtime-guard-counters
dependency_contracts:
- upstream: CR158-S04-evidence-handoff-typed-refs
  type: contract
  required_status_for_dev: lld-approved
- upstream: CR158-S05-no-runtime-guard-counters
  type: contract
  required_status_for_dev: lld-approved
feature_design_refs:
- docs/features/factor-research-loop/TEST-PLAN.md
- docs/features/runtime-authorization-safety/TEST-PLAN.md
- docs/features/runtime-authorization-safety/TASKS.md
lld_policy:
  required_level: technical-note
  trigger_reasons:
  - docs-release
  - verification-traceability
  - low-code-risk
  rationale: Documentation and verification wording Story consumes S04/S05 contracts;
    no new runtime or core schema.
  waiver_reason: ''
  revisit_condition: Upgrade to full-lld if release wording changes runtime authorization
    or adds new evidence schema.
  evidence_path: process/stories/CR158-S06-verification-release-boundary.md#技术说明
file_ownership:
  primary:
  - docs/product/TEST-MATRIX.md
  - docs/quality/VERIFICATION-REPORT.md
  - docs/release/RELEASE-NOTES.md
  shared:
  - docs/features/runtime-authorization-safety/TEST-PLAN.md
  merge_owner: CR158-S06-verification-release-boundary
  forbidden:
  - .env
  - data/**
  - /home/hyde/data/**
lld_gate:
  design_evidence_type: technical-note
  design_evidence_path: process/stories/CR158-S06-verification-release-boundary.md#技术说明
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

收口 CR158 TEST-MATRIX、verification 和 release wording，确保 local/static/fixture adapter readiness 不被写成 production/runtime/model-registry/trading readiness。

## 技术说明

| 项目 | 内容 |
|---|---|
| 设计证据类型 | technical-note |
| 设计依据 | REQ-CR158-007、HLD §12、FEAT-07 CR158 release overclaim guard、S04 evidence/handoff refs LLD、S05 no-runtime guard LLD |
| 文件影响 | CP6 后可修改 `docs/product/TEST-MATRIX.md`、`docs/quality/VERIFICATION-REPORT.md`、`docs/release/RELEASE-NOTES.md` 和 runtime safety TEST-PLAN wording；CP5 本身不修改 source/test implementation。 |
| 接口 / 数据 / 权限变化 | 无新增 runtime、registry、provider、lake、credential、trading、publish 或 Git remote 权限；只收口验证追踪矩阵和发布措辞。 |
| 异常、失败与回退 | 若 TEST-MATRIX、verification 或 release wording 声称 runtime-ready、registry-ready、publish-ready、paper/live-ready 或 trading-ready，则 CP7/CP8 blocked；回退到 S06 文案修正或 runtime authorization CR。 |
| 测试入口 | CP7 文档审查、CP8 release readiness、planned wording guard；核对 CR158 P0 rows 全部映射到 S01-S06，overclaim count == 0。 |
| 风险与重访条件 | R-CR158-RUNTIME-OVERCLAIM；若用户要求真实运行声明，必须先完成 runtime authorization CR 或重走 CP2/CP3/CP5。 |
| 偏离记录 | 当前无偏离；若 CP6/CP7 新增证据路径，需在 `process/design-deltas/` 或 CP7 evidence index 记录。 |

### Technical Note Details

| 项 | 设计 |
|---|---|
| TEST-MATRIX alignment | `SC-CR158-P01/P02/N01/N02/B01/A01` 均必须映射到至少一个 Story、一个 planned test 或 CP7 review item。 |
| Verification wording | 只允许写 `local/static/fixture adapter contract readiness`，不得写 production/runtime readiness。 |
| Release notes wording | 明确 CR158 不包含 real feed、real training、model registry promotion、QMT/runtime、paper/live/trading、publish 或 deployment。 |
| Failure trigger | 任一 forbidden operation counter 非 0、body_copy_count > 0、或 wording overclaim count > 0 均阻断 CP7/CP8。 |
| Merge order | S06 在 S04 evidence refs 和 S05 counter guard 设计确认后收口，不反向修改 S01-S05 contract。 |

### OPEN / Spike 跟踪

| ID | 类型 | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| N/A | OPEN | No open or spike item blocks CP5. | N/A | host-orchestrator |


## 量化验收标准（acceptance_criteria）

- [ ] TEST-MATRIX CR158 P0 rows all map to at least one Story.
- [ ] release/runtime readiness overclaim count == 0.
- [ ] non-authorized item list includes feed/training/registry/runtime/trading/publish.
- [ ] CP8 wording says fixture/static/local only.
