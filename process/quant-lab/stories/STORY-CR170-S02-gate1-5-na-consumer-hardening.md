---
story_id: "CR170-S02-gate1-5-na-consumer-hardening"
title: "Gate 1-5 mandatory N/A consumer 硬化"
story_slug: "gate1-5-na-consumer-hardening"
status: "verified-with-risk"
priority: "P0"
wave: "CR170-W2-GATE-CONSUMERS"
depends_on: ["CR170-S01-na-policy-inventory-five-state-contract"]
dependency_contracts: [{story_id: "CR170-S01-na-policy-inventory-five-state-contract", type: "contract", required_state: "implemented", contract: "21-unit inventory + five-state decision"}]
feature_design_refs: ["docs/features/cross-strategy-reliability-gates/DESIGN.md", "docs/features/cross-strategy-reliability-gates/TEST-PLAN.md", "docs/features/cross-strategy-reliability-gates/TASKS.md"]
feature_refs: ["cross.strategy.reliability.gate.consumers"]
feature_reference_ids: ["FEAT-15"]
lld_policy: "full-lld"
lld_policy_detail: {required_level: "full-lld", trigger_reasons: ["canonical_shared_module", "security_fail_closed", "bidirectional_blast_radius"], rationale: "五个 canonical Gate 同时消费 shared policy，必须分向验证 stricter/controlled-widening/preserve。", waiver_reason: "", revisit_condition: "Gate validator/public result/policy inventory 变化", evidence_path: "process/stories/STORY-CR170-S02-gate1-5-na-consumer-hardening-LLD.md"}
file_ownership: {primary: ["engine/cross_strategy_reliability_gates.py", "tests/research/test_cross_strategy_reliability_gates.py"], shared: ["engine/cross_strategy_reliability_gates.py"], merge_owner: "CR170-S03-admission-worst-state-tier-hardening-after-S02", forbidden: ["engine/economic_cost_gate4_projection.py", "engine/capacity_liquidity_gate4_projection.py", "engine/strategy_admission_package.py"]}
lld_gate: {required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Story", "S01 LLD"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR170-S02-gate1-5-na-consumer-hardening-LLD.md", status: "approved"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-CR170-S02-gate1-5-na-consumer-hardening-IMPLEMENTATION.md", evidence_type: "implementation-md", implementation_objects: ["code", "guardrail-test"], test_plan_refs: ["docs/features/cross-strategy-reliability-gates/TEST-PLAN.md"], local_validation_results: ["61 passed", "py_compile PASS", "diff-check PASS"], status: "implemented"}
verification_gate: {validation_mode: "static-only", validation_target: {sut_type: "code-project", native_test_required: true, workflow_eval_required: false, prompt_bundle_required: false, eval_suite_refs: [], prompt_bundle_refs: [], runtime_authorization_required: []}, cp7_result: "PASS_WITH_RISK", remaining_risks: ["R-CR170-VERIFIER-INDEPENDENCE"], status: "verified-with-risk"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 3
created_at: "2026-07-15T14:40:00+08:00"
updated_at: "2026-07-15T15:34:00+08:00"
---

## 目标

让 Gate 1-5 局部消费 S01 五态 decision，堵住 generic/incomplete reason escape，并只对 5 个已批准 unit 引入 complete-N/A 到 `NEEDS_REVIEW` 的受控放宽。

## 开发上下文与边界

- 继续保留 `_has_na_reason` 的通用布尔语义，但它不得作为 21 个 mandatory unit 的充分条件。
- Gate1 masked escape 必须分别验证 classifier decision、mandatory claim、final worst-state `3/3`。
- Gate 2-5 保持 public callable/Gate ID/schema；complete N/A 不得生成 Gate `PASS`。
- mandatory 只在 unit + applicability 层成立；conditional not-applicable complete N/A 产生 audit-only NR ref，但 mandatory claim=0、status floor=None、Gate status elevation=0。
- 不修改 Gate6 resolver、adapters、aggregate 或 runner。

## 依赖与并行门控

S01 contract 必须先 implemented。S03 与本 Story 写同一 canonical 文件，须按 S02→S03 串行。

## AI 可执行任务清单

| TASK-ID | 动作 | 目标文件 | 完成准则 |
|---|---|---|---|
| CR170-S02-T01 | 修改 | `engine/cross_strategy_reliability_gates.py` | Gate1-5 局部消费 five-state，生成稳定 claim/status floor。 |
| CR170-S02-T02 | 修改 | `tests/research/test_cross_strategy_reliability_gates.py` | 按 15/5/1 三组参数化回归，Gate1 三层断言。 |
| CR170-S02-T03 | 验证 | 同上 | public signatures/Gate IDs 不变，generic/incomplete escape PASS=0。 |

## 量化验收标准

- [x] Gate 覆盖 `5/5`；policy 覆盖 `21/21`。
- [x] stricter `15/15`、controlled-widening `5/5`、preserve `1/1`。
- [x] Gate1 masked escape `3/3`；generic/incomplete mandatory PASS `0`。
- [x] public break `0`；global helper semantics change `0`；adapter modifications `0`。
- [x] conditional not-applicable audit-only contract `1/1`，不被 resolver 误读为 mandatory Gate NR。

## 阻塞说明

无；实现仍受 CP5 人工批准约束。
