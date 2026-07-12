---
story_id: "CR164-S01-statistical-evidence-contract-validator"
title: "Statistical evidence contract and validator"
story_slug: "statistical-evidence-contract-validator"
status: "verified"
priority: "P0"
wave: "CR164-W1-CONTRACT"
depends_on: []
feature_design_refs: ["docs/features/statistical-evidence-contract/DESIGN.md", "docs/features/statistical-evidence-contract/TEST-PLAN.md", "docs/features/statistical-evidence-contract/TASKS.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["public-schema", "cross-story-contract", "lineage-integrity"], rationale: "All calculators and consumers share this contract.", evidence_path: "process/stories/STORY-CR164-S01-statistical-evidence-contract-validator-LLD.md"}
file_ownership: {primary: ["engine/statistical_evidence.py", "tests/research/test_statistical_evidence_contracts.py"], shared: [], merge_owner: "CR164-S01", forbidden: ["engine/experiment_family_lineage.py"]}
lld_gate: {design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR164-S01-statistical-evidence-contract-validator-LLD.md", status: "confirmed"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 4
created_at: "2026-07-12T20:22:00+08:00"
updated_at: "2026-07-12T21:42:00+08:00"
---

## 目标

建立 sealed-lineage-bound input、typed method evidence、summary schema、minima validation 与 canonical hash，作为 S02-S05 唯一合同。

## 开发上下文

- 输入：CR164 HLD/ADR、FEAT-24 三件套、CR163 lineage projection。
- 输出：`engine/statistical_evidence.py` 与 `tests/research/test_statistical_evidence_contracts.py`。
- 约束：缺失→typed_unavailable；冲突/非法→blocked；raw/effective non-alias。

## 量化验收标准

- binding coverage=100%；count difference=0；negative false PASS=0；10 runs→1 hash；orphan refs=0。

## 技术说明

设计证据为独立 full LLD；CP5 批准前不得实现。
