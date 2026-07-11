---
story_id: "CR163-S02-recorder-seal-supersession"
title: "Append-only recorder, deterministic seal and supersession"
story_slug: "recorder-seal-supersession"
status: "verified-with-risk"
priority: "P0"
wave: "CR163-W2-RECORDER-SEAL"
depends_on: ["CR163-S01-family-contract-validator"]
dependency_contracts:
  - {upstream: "CR163-S01-family-contract-validator", type: "contract", lld_gate: "upstream_contract_declared", dev_gate: "upstream_contract_confirmed"}
feature_design_refs: ["docs/features/experiment-family-lineage/DESIGN.md", "docs/features/experiment-family-lineage/TEST-PLAN.md", "docs/features/experiment-family-lineage/TASKS.md"]
feature_refs: ["FEAT-20"]
lld_policy: "full-lld"
lld_policy_detail:
  required_level: "full-lld"
  trigger_reasons: ["append-only-storage", "idempotency", "canonical-hash", "tamper", "recovery"]
  rationale: "Storage, canonical bytes and immutable supersession are integrity-critical and shared downstream."
  evidence_path: "process/stories/STORY-CR163-S02-recorder-seal-supersession-LLD.md"
file_ownership:
  primary: ["engine/experiment_family_lineage_store.py", "tests/test_experiment_family_lineage_store.py"]
  shared: []
  merge_owner: "CR163-S02-recorder-seal-supersession"
  forbidden: ["engine/experiment_family_lineage.py", "engine/mature_multifactor_research.py", "engine/multifactor_strategy_candidates.py", "engine/strategy_admission_statistical_gate.py", "engine/cross_strategy_reliability_gates.py", "engine/strategy_admission_package.py", "data/**"]
lld_gate: {required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Story", "S01 contract", "FEAT-20 DESIGN/TEST-PLAN/TASKS"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR163-S02-recorder-seal-supersession-LLD.md", status: "approved"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-CR163-S02-recorder-seal-supersession-IMPLEMENTATION.md", evidence_type: "implementation-md", implementation_objects: ["code", "storage-contract", "guardrail-test"], test_plan_refs: ["docs/features/experiment-family-lineage/TEST-PLAN.md"], local_validation_results: [], status: "not-started"}
verification_gate: {validation_mode: "static-only", validation_target: {sut_type: "code-project", native_test_required: true, workflow_eval_required: false, prompt_bundle_required: false, eval_suite_refs: [], prompt_bundle_refs: [], runtime_authorization_required: []}, cp7_result: "PASS_WITH_RISK", remaining_risks: ["no-in-place-resume", "cooperative-writer-lock", "10k-characterization-not-capacity"], route_to: "CP8-risk-summary", status: "verified-with-risk"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 3
created_at: "2026-07-11T12:05:00+08:00"
updated_at: "2026-07-11T12:05:00+08:00"
---

## 目标

实现规划 append-only recorder、restricted canonical JSON/JSONL、deterministic SHA-256 seal、immutable manifest versions 与 full supersession resolver。

## 开发上下文（dev_context）

- 输入：S01 frozen contract、FEAT-20 design。
- 输出：store/sealer module 与 golden/integrity tests。
- 设计约束：single writer；create-exclusive；旧 sealed version 永不覆盖；latest pointer非真相。

### 依赖与并行门控

| 上游 | 类型 | LLD 门控 | 开发门控 |
|---|---|---|---|
| S01 | contract | 可基于 declared contract 起草 | S01 contract confirmed |

### AI 可执行任务清单

| TASK-ID | 动作 | 目标 | 描述 |
|---|---|---|---|
| TASK-CR163-S02-01 | 冻结 | S02 LLD | exact layout、canonical bytes/hash domain、I/O atomicity |
| TASK-CR163-S02-02 | 创建（CP5 后） | store module | recorder/sealer/resolver |
| TASK-CR163-S02-03 | 创建（CP5 后） | tests | deterministic/tamper/broken/cyclic/supersession fixtures |

## 量化验收标准（acceptance_criteria）

- [ ] identical fixture 连续 seal 10 次产生 1 个 distinct hash。
- [ ] sealed in-place mutation count=0；valid supersession chain coverage=100%。
- [ ] duplicate-conflict、count mismatch、tamper、broken chain、cyclic chain 5/5 blocked。
- [ ] v1 ref/hash在 v2 supersession后仍可复算；prior ref/hash/reason完整率=100%。
- [ ] path/mtime/current clock/hash itself 进入 hash domain 的字段数=0。

## 阻塞说明

实现被 CP5 human gate 与 S01 contract confirmation 阻断；无规划阻塞。
