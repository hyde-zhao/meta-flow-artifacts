---
story_id: "CR166-S03-deterministic-c2-producer"
title: "Deterministic walk-forward/OOS C2 producer"
story_slug: "deterministic-c2-producer"
status: "done"
priority: "P0"
wave: "CR166-W3-PRODUCER"
depends_on: ["CR166-S01-evidence-envelope-contracts", "CR166-S02-fold-validator-adapters"]
dependency_types: ["contract", "contract"]
feature_design_refs: ["docs/features/walk-forward-oos-producer/DESIGN.md", "docs/features/walk-forward-oos-producer/TEST-PLAN.md", "docs/features/walk-forward-oos-producer/TASKS.md", "docs/features/walk-forward-oos-validation/DESIGN.md", "docs/features/strategy-evidence-envelope/DESIGN.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["claim-sensitive-computation", "declared-denominator", "deterministic-hash", "provenance"], rationale: "Fold outcomes and pass rate must be reproducible without consumer recomputation.", evidence_path: "process/stories/STORY-CR166-S03-deterministic-c2-producer-LLD.md"}
file_ownership: {primary: ["engine/walk_forward_oos_evidence.py", "tests/research/test_walk_forward_oos_producer.py"], shared: [], merge_owner: "CR166-S03"}
lld_gate: {design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR166-S03-deterministic-c2-producer-LLD.md", status: "approved"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true, cp6_result_ref: "process/checks/CP6-CR166-S03.result.json", cp7_result_ref: "process/checks/CP7-CR166-S03.result.json", verification_status: "PASS"}
task_count: 5
created_at: "2026-07-13T12:22:00+08:00"
updated_at: "2026-07-13T15:05:04+08:00"
---

## 目标

对 validated common input 重算 fold/metric decisions、declared-denominator pass rate、reason、provenance 与 canonical identity，并执行 evidence self-validation。

## 量化验收标准

present evidence 重算差异=0；10 次→1 component/envelope hash；missing/invalid fold 不从分母消失；tamper acceptance=0；I/O=0。

## 技术说明

producer 不调用 gate、不决定 overall admission。CP5 已批准，等待 S01/S02 门控满足后实现。
