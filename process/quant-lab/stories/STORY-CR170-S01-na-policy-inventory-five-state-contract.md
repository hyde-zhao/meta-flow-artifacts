---
story_id: "CR170-S01-na-policy-inventory-five-state-contract"
title: "21-unit N/A policy inventory 与五态合同"
story_slug: "na-policy-inventory-five-state-contract"
status: "verified-with-risk"
priority: "P0"
wave: "CR170-W1-POLICY"
depends_on: []
dependency_contracts: []
feature_design_refs: ["docs/features/cross-strategy-reliability-gates/DESIGN.md", "docs/features/cross-strategy-reliability-gates/TEST-PLAN.md", "docs/features/cross-strategy-reliability-gates/TASKS.md"]
feature_refs: ["cross.strategy.reliability.na.policy"]
feature_reference_ids: ["FEAT-15"]
lld_policy: "full-lld"
lld_policy_detail: {required_level: "full-lld", trigger_reasons: ["security_semantics", "shared_contract", "cross_story_dependency"], rationale: "21-unit inventory、五态与 boundary caller contract 是全部下游 Gate/admission 的安全单一真相源。", waiver_reason: "", revisit_condition: "policy unit/five-state/direction/disposition/boundary contract 变化", evidence_path: "process/stories/STORY-CR170-S01-na-policy-inventory-five-state-contract-LLD.md"}
file_ownership: {primary: ["engine/reliability_na_policy.py", "tests/research/test_reliability_na_policy.py"], shared: [], merge_owner: "CR170-S01-na-policy-inventory-five-state-contract", forbidden: ["engine/cross_strategy_reliability_gates.py", "engine/economic_cost_gate4_projection.py", "engine/capacity_liquidity_gate4_projection.py", "engine/strategy_admission_package.py"]}
lld_gate: {required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Story"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR170-S01-na-policy-inventory-five-state-contract-LLD.md", status: "approved"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-CR170-S01-na-policy-inventory-five-state-contract-IMPLEMENTATION.md", evidence_type: "implementation-md", implementation_objects: ["code", "guardrail-test"], test_plan_refs: ["docs/features/cross-strategy-reliability-gates/TEST-PLAN.md"], local_validation_results: ["16 passed", "py_compile PASS", "diff-check PASS"], status: "implemented"}
verification_gate: {validation_mode: "static-only", validation_target: {sut_type: "code-project", native_test_required: true, workflow_eval_required: false, prompt_bundle_required: false, eval_suite_refs: [], prompt_bundle_refs: [], runtime_authorization_required: []}, cp7_result: "PASS_WITH_RISK", remaining_risks: ["R-CR170-VERIFIER-INDEPENDENCE"], status: "verified-with-risk"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 3
created_at: "2026-07-15T14:40:00+08:00"
updated_at: "2026-07-15T15:23:00+08:00"
---

## 目标

创建 private、无 IO 的 `NaPolicySpec` / `NaEvidenceDecision` 五态合同及 21/21 policy inventory，精确记录现有路径类型、15/5/1 硬化方向、complete-N/A disposition 与 caller boundary。

## 开发上下文与边界

- 输入：CR-170 HLD/ADR、FEAT-15 v0.2、canonical Gate 1-5 现有字段事实。
- 输出：`engine/reliability_na_policy.py` 与 unit tests。
- `n_a_boundaries` 当前 writer 仅 fixture/test；evaluator 合成 boundary/authorization ref 数量必须为 `0`。
- `authorization_ref` 是 opaque audit pointer，不是 credential、token 或 runtime authorization。
- 不修改 canonical Gate、adapters、aggregate 或 public exports。

## 依赖与并行门控

无上游 Story。S02/S03 消费本 Story 的 frozen internal contract，因此 S01 必须先完成且不得与 S02 并行实现。

## AI 可执行任务清单

| TASK-ID | 动作 | 目标文件 | 完成准则 |
|---|---|---|---|
| CR170-S01-T01 | 创建 | `engine/reliability_na_policy.py` | 定义 private typed five-state、policy/boundary/decision 对象与 deterministic reason ID。 |
| CR170-S01-T02 | 创建 | 同上 | 定义 exact 21-unit inventory、6/6/1/5/3 Gate 分布、15/5/1 与 classifier；不做 IO/合成。 |
| CR170-S01-T03 | 创建 | `tests/research/test_reliability_na_policy.py` | 验证 21/21、5/5、4/4 boundary、caller、方向/disposition 与 deterministic output。 |

## 量化验收标准

- [x] policy inventory `21/21`，Gate 分布 `6/6/1/5/3`，ID 重复 `0`。
- [x] five-state `5/5`；方向 `15 stricter / 5 controlled-widening / 1 preserve`。
- [x] 每行 `baseline_path_type/hardening_direction/complete_na_disposition` 填写率 `100%`。
- [x] complete boundary 字段 `4/4`；evaluator synthesis `0`；public API/schema break `0`。

## 阻塞说明

无；实现仍受 CP5 人工批准约束。
