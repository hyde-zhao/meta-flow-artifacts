---
story_id: "CR166-S05-independent-static-verification"
title: "Independent fixture/static verification and claim audit"
story_slug: "independent-static-verification"
status: "done"
priority: "P0"
wave: "CR166-W5-VERIFICATION"
depends_on: ["CR166-S01-evidence-envelope-contracts", "CR166-S02-fold-validator-adapters", "CR166-S03-deterministic-c2-producer", "CR166-S04-existing-consumer-projections"]
dependency_types: ["runtime", "runtime", "runtime", "runtime"]
feature_design_refs: ["docs/features/walk-forward-oos-verification/DESIGN.md", "docs/features/walk-forward-oos-verification/TEST-PLAN.md", "docs/features/walk-forward-oos-verification/TASKS.md", "docs/features/strategy-evidence-envelope/TEST-PLAN.md", "docs/features/walk-forward-oos-validation/TEST-PLAN.md", "docs/features/walk-forward-oos-producer/TEST-PLAN.md", "docs/features/walk-forward-oos-projections/TEST-PLAN.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["cross-story-verification", "authorization-audit", "historical-failure-attribution", "Stage-claim-ceiling"], rationale: "CP7 needs one independent evidence plan spanning 12 QAC and all four production Features.", evidence_path: "process/stories/STORY-CR166-S05-independent-static-verification-LLD.md"}
file_ownership: {primary: ["tests/research/test_walk_forward_oos_qac.py", "tests/research/test_walk_forward_oos_authorization.py", "tests/research/test_walk_forward_oos_cr155_regression.py", "tests/fixtures/walk_forward_oos/"], shared: [], merge_owner: "CR166-S05", read_only: ["engine/strategy_evidence.py", "engine/walk_forward_oos_evidence.py", "engine/walk_forward_oos_projections.py"]}
lld_gate: {design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR166-S05-independent-static-verification-LLD.md", status: "approved"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true, cp6_result_ref: "process/checks/CP6-CR166-S05.result.json", cp7_result_ref: "process/checks/CP7-CR166-S05.result.json", verification_status: "PASS"}
task_count: 5
created_at: "2026-07-13T12:24:00+08:00"
updated_at: "2026-07-13T15:05:04+08:00"
---

## 目标

以独立 fixture/static lane 覆盖 11/11 scenarios、12/12 QAC、8/8 fail-closed、permission/Stage claim、CR155 regression 和 CR165 历史失败逐项归因。

## 量化验收标准

QAC=12/12；daily/ML=2/2；event N/A=1/1 且 producer=0；forbidden operations=0；CR166 新路径 failure=0；触及 CR165 历史失败归因率=100%。

## 技术说明

本 Story 不拥有生产接口，不运行真实数据、runtime 或外部系统。CP5 已批准 synthetic fixture/static 验证，等待 S01-S04 全部 verified 后执行。
