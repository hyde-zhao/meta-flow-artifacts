---
story_id: "CR166-S02-fold-validator-adapters"
title: "Fold, leakage and sufficiency validator with daily/ML adapters"
story_slug: "fold-validator-adapters"
status: "done"
priority: "P0"
wave: "CR166-W2-VALIDATION"
depends_on: ["CR166-S01-evidence-envelope-contracts"]
dependency_types: ["contract"]
feature_design_refs: ["docs/features/walk-forward-oos-validation/DESIGN.md", "docs/features/walk-forward-oos-validation/TEST-PLAN.md", "docs/features/walk-forward-oos-validation/TASKS.md", "docs/features/strategy-evidence-envelope/DESIGN.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["temporal-integrity", "leakage-risk", "cross-strategy-adapters", "authorization-boundary"], rationale: "Incorrect fold or purge/embargo validation can create false OOS claims.", evidence_path: "process/stories/STORY-CR166-S02-fold-validator-adapters-LLD.md"}
file_ownership: {primary: ["engine/walk_forward_oos_evidence.py", "tests/research/test_walk_forward_oos_validation.py"], shared: [], merge_owner: "CR166-S02", read_only: ["engine/daily_multifactor_baseline_artifact.py", "engine/research_production_contracts.py"]}
lld_gate: {design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR166-S02-fold-validator-adapters-LLD.md", status: "approved"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true, cp6_result_ref: "process/checks/CP6-CR166-S02.result.json", cp7_result_ref: "process/checks/CP7-CR166-S02.result.json", verification_status: "PASS"}
task_count: 5
created_at: "2026-07-13T12:21:00+08:00"
updated_at: "2026-07-13T15:05:04+08:00"
---

## 目标

建立唯一 common validation boundary，覆盖 7 字段族、半开时间区间、purge/embargo、metrics、lineage、authorization，并提供 daily/ML 显式 adapter 与 event N/A。

## 量化验收标准

daily/ML=2/2；8 类 P0 fail-closed 有 typed reason；temporal/leakage=3/3 blocked；event N/A=1/1；dereference/forbidden operations=0。

## 技术说明

只映射传入值，不读取 source refs、不推断 calendar/session/OOS bounds。CP5 已批准，等待 S01 契约门控满足后实现。
