---
story_id: "CR163-S04-existing-admission-projection"
title: "Existing admission consumer lineage projection"
story_slug: "existing-admission-projection"
status: "verified-with-risk"
priority: "P0"
wave: "CR163-W3-PRODUCERS-CONSUMERS"
depends_on: ["CR163-S01-family-contract-validator", "CR163-S02-recorder-seal-supersession"]
dependency_contracts:
  - {upstream: "CR163-S01-family-contract-validator", type: "contract", lld_gate: "upstream_contract_declared", dev_gate: "upstream_contract_confirmed"}
  - {upstream: "CR163-S02-recorder-seal-supersession", type: "contract", lld_gate: "upstream_contract_declared", dev_gate: "upstream_contract_confirmed"}
feature_design_refs: ["docs/features/strategy-admission-lineage-projection/DESIGN.md", "docs/features/strategy-admission-lineage-projection/TEST-PLAN.md", "docs/features/strategy-admission-lineage-projection/TASKS.md", "docs/features/experiment-family-lineage/DESIGN.md"]
feature_refs: ["FEAT-22", "FEAT-20"]
lld_policy: "full-lld"
lld_policy_detail: {required_level: "full-lld", trigger_reasons: ["three-consumer-surfaces", "compatibility", "fail-closed", "claim-ceiling"], rationale: "Three existing consumers share availability/raw count/manual reconciliation semantics and status precedence.", evidence_path: "process/stories/STORY-CR163-S04-existing-admission-projection-LLD.md"}
file_ownership:
  primary: ["engine/strategy_admission_statistical_gate.py", "engine/cross_strategy_reliability_gates.py", "engine/strategy_admission_package.py", "tests/test_cr163_trial_lineage_admission_projection.py"]
  shared: []
  merge_owner: "CR163-S04-existing-admission-projection"
  forbidden: ["engine/mature_multifactor_research.py", "engine/multifactor_strategy_candidates.py", "engine/experiment_family_lineage_store.py", "data/**", "credentials", "new gate family"]
lld_gate: {required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Story", "S01/S02 contracts", "FEAT-22 DESIGN/TEST-PLAN/TASKS"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR163-S04-existing-admission-projection-LLD.md", status: "approved"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-CR163-S04-existing-admission-projection-IMPLEMENTATION.md", evidence_type: "implementation-md", implementation_objects: ["code", "consumer-adapter", "guardrail-test"], test_plan_refs: ["docs/features/strategy-admission-lineage-projection/TEST-PLAN.md"], local_validation_results: [], status: "not-started"}
verification_gate: {validation_mode: "static-only", validation_target: {sut_type: "code-project", native_test_required: true, workflow_eval_required: false, prompt_bundle_required: false, eval_suite_refs: [], prompt_bundle_refs: [], runtime_authorization_required: []}, cp7_result: "not-started", remaining_risks: [], route_to: "", status: "not-started"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 3
created_at: "2026-07-11T12:05:00+08:00"
updated_at: "2026-07-11T12:05:00+08:00"
---

## 目标

把 sealed+validated family lineage projection 接入现有 CR151、CR154 与 admission package consumers，冻结 availability/ref/raw count/manual reconciliation/status-worsening，且不建立新 gate。

## 开发上下文（dev_context）

- 输入：S01 projection contract、S02 manifest validation、FEAT-22 design。
- 输出：三个 existing consumer modules 和 projection fixture tests。
- 约束：effective unavailable/ref/method empty；C1 non-computable；CR155 blocked；runtime flags不变。

### 依赖与并行门控

S01/S02 contract dependencies。与 S03 可并行开发，因为 primary files不重叠。

### AI 可执行任务清单

| TASK-ID | 动作 | 目标 |
|---|---|---|
| TASK-CR163-S04-01 | 冻结 | DTO/status/manual reconciliation table |
| TASK-CR163-S04-02 | 修改（CP5 后） | three existing consumer surfaces |
| TASK-CR163-S04-03 | 创建（CP5 后） | present/unavailable/blocked/CR155 fixtures |

## 量化验收标准（acceptance_criteria）

- [ ] 3/3 consumers消费相同 validation-bound lineage ref/hash/raw count。
- [ ] present只在 seal/completeness/ref/count/tamper五类 checks全 PASS 时出现。
- [ ] uninstrumented fixtures 100% typed_unavailable；invalid/tampered/manual mismatch fixtures 100% blocked。
- [ ] effective available=0、nonempty ref/method=0、C1 computed=0、新 gate count=0。
- [ ] CR155 blocked 1/1、paper_candidate=false、historical backfill=0、runtime auth flag changes=0。

## 阻塞说明

实现被 CP5 与 S01/S02 contracts 阻断；无规划阻塞。
