---
story_id: "CR164-S05-independent-verification"
title: "Independent QAC permission and CR155 verification"
story_slug: "independent-verification"
status: "verified-with-risk"
priority: "P0"
wave: "CR164-W4-VERIFICATION"
depends_on: ["CR164-S01-statistical-evidence-contract-validator", "CR164-S02-bh-wrc-spa-evidence", "CR164-S03-pbo-cscv-dsr-evidence", "CR164-S04-conservative-aggregation-projections"]
feature_design_refs: ["docs/features/statistical-evidence-contract/TEST-PLAN.md", "docs/features/multiple-testing-calculators/TEST-PLAN.md", "docs/features/overfit-deflation-calculators/TEST-PLAN.md", "docs/features/statistical-evidence-projection/TEST-PLAN.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["cross-story-verification", "authorization", "negative-regression"], rationale: "Independent evidence must cover all contracts and claim ceilings.", evidence_path: "process/stories/STORY-CR164-S05-independent-verification-LLD.md"}
file_ownership: {primary: ["tests/research/test_statistical_evidence_qac.py", "tests/research/test_statistical_evidence_authorization.py", "tests/research/test_statistical_evidence_cr155_regression.py"], shared: [], merge_owner: "CR164-S05", forbidden: ["engine/**"]}
lld_gate: {design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR164-S05-independent-verification-LLD.md", status: "confirmed"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 3
created_at: "2026-07-12T20:22:00+08:00"
updated_at: "2026-07-12T21:42:00+08:00"
---

## 目标

独立证明 10 QAC、13 scenarios、determinism、permission zero 与 CR155 1/1 blocked。

## 开发上下文

仅 local synthetic fixtures；测试不得修复生产代码、读取真实数据或执行外部/runtime 操作。

## 量化验收标准

- QAC=10/10；scenarios=13/13；10 runs→1 hash；forbidden counters each 0；CR155=1/1 blocked。

## 技术说明

设计证据为独立 full LLD；CP7 必须由 fresh independent verification 执行，但当前 no-subagent 限制仍有效，届时需继续 inline 或重新获得用户指示。
