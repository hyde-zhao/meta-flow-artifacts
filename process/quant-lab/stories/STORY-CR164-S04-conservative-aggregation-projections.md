---
story_id: "CR164-S04-conservative-aggregation-projections"
title: "Conservative aggregation and existing-consumer projections"
story_slug: "conservative-aggregation-projections"
status: "verified"
priority: "P0"
wave: "CR164-W3-PROJECTION"
depends_on: ["CR164-S01-statistical-evidence-contract-validator", "CR164-S02-bh-wrc-spa-evidence", "CR164-S03-pbo-cscv-dsr-evidence"]
feature_design_refs: ["docs/features/statistical-evidence-projection/DESIGN.md", "docs/features/statistical-evidence-projection/TEST-PLAN.md", "docs/features/statistical-evidence-projection/TASKS.md", "docs/features/statistical-evidence-contract/DESIGN.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["cross-module-contract", "admission-policy", "three-consumers"], rationale: "Worst-state projection affects admission claims.", evidence_path: "process/stories/STORY-CR164-S04-conservative-aggregation-projections-LLD.md"}
file_ownership: {primary: ["engine/strategy_admission_statistical_gate.py", "engine/cross_strategy_reliability_gates.py", "engine/strategy_admission_package.py", "tests/research/test_statistical_evidence_projection.py"], shared: [], merge_owner: "CR164-S04", forbidden: ["engine/experiment_family_lineage.py"]}
lld_gate: {design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR164-S04-conservative-aggregation-projections-LLD.md", status: "confirmed"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 4
created_at: "2026-07-12T20:22:00+08:00"
updated_at: "2026-07-12T21:42:00+08:00"
---

## 目标

以 severity lattice 聚合四方法，并投影到 CR151、CR154、admission package；UC59/60 compatibility-only。

## 开发上下文

- 决策：BLOCKED > FAIL > TYPED_UNAVAILABLE > PASS；all mandatory PASS 才 PASS。
- 禁止：OR/majority、新 gate、consumer 重算、raw→effective。

## 量化验收标准

- disagreement fixtures=3/3；consumer projection=3/3；orphan=0；overclaim=0。

## 技术说明

设计证据为独立 full LLD；CP5 前不得修改 consumer 源码。
