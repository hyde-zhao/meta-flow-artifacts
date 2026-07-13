---
story_id: "CR166-S01-evidence-envelope-contracts"
title: "Strategy evidence envelope and C2 contracts"
story_slug: "evidence-envelope-contracts"
status: "done"
priority: "P0"
wave: "CR166-W1-CONTRACTS"
depends_on: []
dependency_types: []
feature_design_refs: ["docs/features/strategy-evidence-envelope/DESIGN.md", "docs/features/strategy-evidence-envelope/TEST-PLAN.md", "docs/features/strategy-evidence-envelope/TASKS.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["public-schema", "cross-module-canonical-contract", "C1-hash-compatibility"], rationale: "All CR166 Stories and future component slots depend on the same neutral envelope.", evidence_path: "process/stories/STORY-CR166-S01-evidence-envelope-contracts-LLD.md"}
file_ownership: {primary: ["engine/strategy_evidence.py", "engine/walk_forward_oos_evidence.py", "engine/statistical_evidence.py", "tests/research/test_walk_forward_oos_contracts.py", "tests/research/test_statistical_evidence_contracts.py"], shared: [], merge_owner: "CR166-S01", forbidden: ["engine/experiment_family_lineage_store.py"]}
lld_gate: {design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR166-S01-evidence-envelope-contracts-LLD.md", status: "approved"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true, cp6_result_ref: "process/checks/CP6-CR166-S01.result.json", cp7_result_ref: "process/checks/CP7-CR166-S01.result.json", verification_status: "PASS"}
task_count: 5
created_at: "2026-07-13T12:20:00+08:00"
updated_at: "2026-07-13T15:05:04+08:00"
---

## 目标

建立 method-neutral canonical/envelope、静态 component catalog 与 C2 value contracts，并以兼容 wrapper 保持 C1 public API/default domain/bytes/hash 100% 不变。

## 开发上下文

- 输入：CR166 HLD/ADR、FEAT-166-01 三件套、当前 `engine/statistical_evidence.py`。
- 输出：neutral contract、C2 contract shape、C1 compatibility regression。
- 约束：C3/C4 calculator=0；dynamic registry/store=0；外部 I/O=0。

## 量化验收标准

contract 字段族=7/7；C1 compatibility difference=0；同一 payload 10 次 hash distinct=1；mandatory unknown false PASS=0；forbidden operations=0。

## 技术说明

设计证据为 full LLD。CP5 已批准 repository-local 源代码、测试与 synthetic fixture 实现。
