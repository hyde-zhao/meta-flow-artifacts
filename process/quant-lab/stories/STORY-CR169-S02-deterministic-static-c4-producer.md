---
story_id: "CR169-S02-deterministic-static-c4-producer"
title: "确定性 Static C4 Producer"
story_slug: "deterministic-static-c4-producer"
status: "verified-with-risk"
priority: "P0"
wave: "CR169-W2-STATIC-PRODUCER"
depends_on: ["CR169-S01-capacity-liquidity-contract-validation"]
feature_design_refs: ["docs/features/capacity-liquidity-evidence/DESIGN.md", "docs/features/capacity-liquidity-evidence/TEST-PLAN.md", "docs/features/capacity-liquidity-evidence/TASKS.md"]
feature_refs: ["capacity.liquidity.evidence"]
feature_reference_ids: ["FEAT-169-01"]
lld_policy: "full-lld"
lld_policy_detail: {required_level: "full-lld", trigger_reasons: ["numeric_contract", "public_producer", "determinism"], rationale: "Decimal static proxy、cap、quantization 与 3 refs 是公共可复算合同。", waiver_reason: "", revisit_condition: "formula/rounding/method 变化", evidence_path: "process/stories/STORY-CR169-S02-deterministic-static-c4-producer-LLD.md"}
file_ownership: {primary: ["engine/capacity_liquidity_calculator.py", "engine/capacity_liquidity_evidence.py", "tests/research/test_capacity_liquidity_producer.py"], shared: ["engine/capacity_liquidity_evidence.py"], merge_owner: "CR169-S02-deterministic-static-c4-producer", forbidden: ["engine/cross_strategy_reliability_gates.py", "engine/economic_cost_gate4_projection.py"]}
lld_gate: {design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR169-S02-deterministic-static-c4-producer-LLD.md", status: "approved"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 3
created_at: "2026-07-14T19:05:00+08:00"
updated_at: "2026-07-15T09:46:00+08:00"
---

## 目标

实现 `static_adv_cap_v1` pure Decimal calculator 与唯一 C4 producer，生成 3/3 typed refs、limitations、no-real claims 和 deterministic component。

## 编排合同

`build_capacity_liquidity_evidence` 固定 normalize→validate→issues short-circuit→calculate→produce；调用方不能传 validation flag。invalid path calculator calls=0。

## 量化验收标准

- [ ] 3 refs present=3/3；valid calculator call=1，invalid=0。
- [ ] precision=28；minor-unit HALF_EVEN；ratio>cap false PASS=0。
- [ ] real ADV/liquidity/capacity-ready 与 alpha calculator全 false/0。
