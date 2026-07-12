---
story_id: "CR164-S02-bh-wrc-spa-evidence"
title: "BH and WRC SPA computable evidence"
story_slug: "bh-wrc-spa-evidence"
status: "verified"
priority: "P0"
wave: "CR164-W2-CALCULATORS"
depends_on: ["CR164-S01-statistical-evidence-contract-validator"]
feature_design_refs: ["docs/features/multiple-testing-calculators/DESIGN.md", "docs/features/multiple-testing-calculators/TEST-PLAN.md", "docs/features/multiple-testing-calculators/TASKS.md", "docs/features/statistical-evidence-contract/DESIGN.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["statistical-algorithm", "bootstrap-provenance", "legacy-contract-risk"], rationale: "Formula and deterministic bootstrap require exact design.", evidence_path: "process/stories/STORY-CR164-S02-bh-wrc-spa-evidence-LLD.md"}
file_ownership: {primary: ["engine/multiple_testing_evidence.py", "tests/research/test_multiple_testing_evidence.py"], shared: ["engine/anomaly_multiple_testing.py"], merge_owner: "CR164-S02", forbidden: ["engine/statistical_evidence.py"]}
lld_gate: {design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR164-S02-bh-wrc-spa-evidence-LLD.md", status: "confirmed"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 4
created_at: "2026-07-12T20:22:00+08:00"
updated_at: "2026-07-12T21:42:00+08:00"
---

## 目标

实现 provenance-complete BH 与 fixed-window stationary-bootstrap WRC/SPA pure calculators。

## 开发上下文

- 依赖：S01 frozen contract。
- 约束：candidate≥2；explicit window/seed/replications/benchmark/null；禁止 legacy OR-pass。

## 量化验收标准

- known BH golden vector 100%；bootstrap reruns deterministic；invalid config false PASS=0。

## 技术说明

设计证据为独立 full LLD；automatic block selector 不在范围。
