---
story_id: "CR164-S03-pbo-cscv-dsr-evidence"
title: "PBO CSCV and raw-count DSR evidence"
story_slug: "pbo-cscv-dsr-evidence"
status: "verified"
priority: "P0"
wave: "CR164-W2-CALCULATORS"
depends_on: ["CR164-S01-statistical-evidence-contract-validator"]
feature_design_refs: ["docs/features/overfit-deflation-calculators/DESIGN.md", "docs/features/overfit-deflation-calculators/TEST-PLAN.md", "docs/features/overfit-deflation-calculators/TASKS.md", "docs/features/statistical-evidence-contract/DESIGN.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["statistical-algorithm", "split-leakage", "raw-effective-non-alias"], rationale: "CSCV and DSR boundaries are claim-sensitive.", evidence_path: "process/stories/STORY-CR164-S03-pbo-cscv-dsr-evidence-LLD.md"}
file_ownership: {primary: ["engine/overfit_evidence.py", "tests/research/test_overfit_evidence.py"], shared: [], merge_owner: "CR164-S03", forbidden: ["engine/statistical_evidence.py", "engine/experiment_family_lineage.py"]}
lld_gate: {design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR164-S03-pbo-cscv-dsr-evidence-LLD.md", status: "confirmed"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 4
created_at: "2026-07-12T20:22:00+08:00"
updated_at: "2026-07-12T21:42:00+08:00"
---

## 目标

实现 stable CSCV/PBO 与 `dsr_input_method=raw_trial_count` 的 DSR evidence，并硬性阻止 raw/effective alias。

## 开发上下文

- PBO minima：4 candidates / 4 valid splits / non-empty train-test。
- DSR minima：2 trials / sample≥30 / finite moments / variance>0。

## 量化验收标准

- PBO/DSR domain valid=100%；leaky/duplicate split blocked=100%；effective alias blocked=100%。

## 技术说明

设计证据为独立 full LLD；effective-trial estimator 不在范围。
