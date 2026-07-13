---
story_id: "CR166-S04-existing-consumer-projections"
title: "Existing-consumer projections and CR155 regression"
story_slug: "existing-consumer-projections"
status: "done"
priority: "P0"
wave: "CR166-W4-PROJECTIONS"
depends_on: ["CR166-S01-evidence-envelope-contracts", "CR166-S02-fold-validator-adapters", "CR166-S03-deterministic-c2-producer"]
dependency_types: ["contract", "contract", "runtime"]
feature_design_refs: ["docs/features/walk-forward-oos-projections/DESIGN.md", "docs/features/walk-forward-oos-projections/TEST-PLAN.md", "docs/features/walk-forward-oos-projections/TASKS.md", "docs/features/walk-forward-oos-producer/DESIGN.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["cross-module-integration", "backward-compatibility", "worse-state-merge", "claim-ceiling"], rationale: "Three existing policy owners must consume one identity without a parallel gate.", evidence_path: "process/stories/STORY-CR166-S04-existing-consumer-projections-LLD.md"}
file_ownership: {primary: ["engine/walk_forward_oos_projections.py", "engine/strategy_admission_statistical_gate.py", "engine/cross_strategy_reliability_gates.py", "engine/strategy_admission_package.py", "tests/research/test_walk_forward_oos_projections.py", "tests/research/test_statistical_evidence_cr155_regression.py"], shared: [], merge_owner: "CR166-S04"}
lld_gate: {design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR166-S04-existing-consumer-projections-LLD.md", status: "approved"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true, cp6_result_ref: "process/checks/CP6-CR166-S04.result.json", cp7_result_ref: "process/checks/CP7-CR166-S04.result.json", verification_status: "PASS"}
task_count: 5
created_at: "2026-07-13T12:23:00+08:00"
updated_at: "2026-07-13T15:05:04+08:00"
---

## 目标

将同一 C2 ref/hash/availability/reasons 保守投影到 CR151、CR154 Gate 2 与 StrategyAdmissionPackage，保持 policy owner、runtime flags 和 CR155 blocked 状态。

## 量化验收标准

consumer=3/3；identity difference=0；status improvement=0；parallel gate=0；CR155 `paper_candidate` promotion=0；raw fold recomputation=0。

## 技术说明

只增加薄 projection/attach；present+pass 仅 eligible，不是整体或运行授权。CP5 已批准，等待上游 runtime/contract 门控满足后实现。
