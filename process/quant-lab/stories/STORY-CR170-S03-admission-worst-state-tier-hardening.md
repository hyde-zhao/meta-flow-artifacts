---
story_id: "CR170-S03-admission-worst-state-tier-hardening"
title: "Gate 6 protected merge 与 admission tier 硬化"
story_slug: "admission-worst-state-tier-hardening"
status: "verified-with-risk"
priority: "P0"
wave: "CR170-W3-ADMISSION"
depends_on: ["CR170-S01-na-policy-inventory-five-state-contract", "CR170-S02-gate1-5-na-consumer-hardening"]
dependency_contracts: [{story_id: "CR170-S01-na-policy-inventory-five-state-contract", type: "contract", required_state: "implemented", contract: "five-state policy"}, {story_id: "CR170-S02-gate1-5-na-consumer-hardening", type: "runtime", required_state: "implemented", contract: "Gate1-5 statuses/claims"}, {story_id: "CR170-S02-gate1-5-na-consumer-hardening", type: "file-conflict", required_state: "merged", contract: "canonical file write ownership"}]
feature_design_refs: ["docs/features/cross-strategy-reliability-gates/DESIGN.md", "docs/features/cross-strategy-reliability-gates/TEST-PLAN.md", "docs/features/cross-strategy-reliability-gates/TASKS.md"]
feature_refs: ["cross.strategy.reliability.admission.policy"]
feature_reference_ids: ["FEAT-15"]
lld_policy: "full-lld"
lld_policy_detail: {required_level: "full-lld", trigger_reasons: ["admission_policy", "shared_file", "release_safety", "public_compatibility"], rationale: "T0-T2 的 mandatory NEEDS_REVIEW 当前会升级 PASS；同时 merge/T3 是受保护兼容路径。", waiver_reason: "", revisit_condition: "tier/mode/summary/resolver 变化", evidence_path: "process/stories/STORY-CR170-S03-admission-worst-state-tier-hardening-LLD.md"}
file_ownership: {primary: ["engine/cross_strategy_reliability_gates.py", "tests/research/test_reliability_admission_policy.py"], shared: ["engine/cross_strategy_reliability_gates.py"], merge_owner: "CR170-S03-admission-worst-state-tier-hardening", forbidden: ["engine/economic_cost_gate4_projection.py", "engine/capacity_liquidity_gate4_projection.py", "engine/strategy_admission_package.py"]}
lld_gate: {required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Story", "S01 LLD", "S02 LLD"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR170-S03-admission-worst-state-tier-hardening-LLD.md", status: "approved"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-CR170-S03-admission-worst-state-tier-hardening-IMPLEMENTATION.md", evidence_type: "implementation-md", implementation_objects: ["code", "guardrail-test"], test_plan_refs: ["docs/features/cross-strategy-reliability-gates/TEST-PLAN.md"], local_validation_results: ["69 passed", "py_compile PASS", "diff-check PASS"], status: "implemented"}
verification_gate: {validation_mode: "static-only", validation_target: {sut_type: "code-project", native_test_required: true, workflow_eval_required: false, prompt_bundle_required: false, eval_suite_refs: [], prompt_bundle_refs: [], runtime_authorization_required: []}, cp7_result: "PASS_WITH_RISK", remaining_risks: ["R-CR170-VERIFIER-INDEPENDENCE"], status: "verified-with-risk"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 3
created_at: "2026-07-15T14:40:00+08:00"
updated_at: "2026-07-15T15:45:00+08:00"
---

## 目标

先证明并保护既有 bottom-up worst-state merge，再最小硬化 `resolve_admission_policy`，使 mandatory `NEEDS_REVIEW` 在 T0/T1/T2 分别得到 `NEEDS_REVIEW/BLOCKED/BLOCKED`，T3 保持现状。

## 开发上下文与边界

- `build_shared_gate_summary` / `evaluate_shared_contract` 回归通过时 production diff=`0`。
- T3 现有 early-return 必须保持 `BLOCKED + NOT_AUTHORIZED`，production diff=`0`。
- resolver 只消费 S02 已形成的 Gate summary `NEEDS_REVIEW`；该 floor 仅来自 applicable mandatory unit，不把 Gate ID 本身解释为全局 mandatory，也不从 artifact reason 重算。
- 不接 runner、不做 aggregate、不提升 CR155。

## 依赖与并行门控

必须等待 S02 合并以取得 canonical 文件 owner；S03 是该文件的最终 merge owner。

## AI 可执行任务清单

| TASK-ID | 动作 | 目标文件 | 完成准则 |
|---|---|---|---|
| CR170-S03-T01 | 创建 | `tests/research/test_reliability_admission_policy.py` | 先锁定 merge 1/1 与 T0-T3 基线/期望。 |
| CR170-S03-T02 | 修改 | `engine/cross_strategy_reliability_gates.py` | 仅最小硬化 resolver T0/T1/T2；merge/T3 不改。 |
| CR170-S03-T03 | 验证 | 两文件 | source rule/wording 稳定、unknown profile fail-closed、public break=0。 |

## 量化验收标准

- [x] protected merge `1/1`，通过时 production diff=`0`。
- [x] T0/T1/T2/T3 `4/4` = `NEEDS_REVIEW/BLOCKED/BLOCKED/NOT_AUTHORIZED`。
- [x] T3 existing early-return `1/1`，production diff=`0`。
- [ ] mandatory NEEDS_REVIEW admission PASS=`0`；new public status/signature=`0`。

## 阻塞说明

S02 未合并前为 file-conflict blocked；实现仍受 CP5 人工批准约束。
