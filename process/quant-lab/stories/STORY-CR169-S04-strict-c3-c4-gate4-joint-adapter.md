---
story_id: "CR169-S04-strict-c3-c4-gate4-joint-adapter"
title: "Strict C3+C4 Gate4 Joint Fixture Adapter"
story_slug: "strict-c3-c4-gate4-joint-adapter"
status: "verified-with-risk"
priority: "P0"
wave: "CR169-W4-STRICT-JOINT-ADAPTER"
depends_on: ["CR169-S01-capacity-liquidity-contract-validation", "CR169-S02-deterministic-static-c4-producer", "CR169-S03-neutral-envelope-c4-activation"]
feature_design_refs: ["docs/features/c3-c4-correlation-boundary/DESIGN.md", "docs/features/c3-c4-correlation-boundary/TEST-PLAN.md", "docs/features/capacity-liquidity-gate4-joint-adapter/DESIGN.md", "docs/features/capacity-liquidity-gate4-joint-adapter/TEST-PLAN.md", "docs/features/capacity-liquidity-gate4-joint-adapter/TASKS.md"]
feature_refs: ["c3.c4.correlation.boundary", "capacity.liquidity.gate4.joint.adapter"]
feature_reference_ids: ["FEAT-169-02", "FEAT-169-03"]
lld_policy: "full-lld"
lld_policy_detail: {required_level: "full-lld", trigger_reasons: ["security", "cross_module", "public_callable", "postcondition"], rationale: "13-field precheck、7-key payload、reason escape 与 non-upgrading postcondition 是 Gate4 false-PASS containment。", waiver_reason: "", revisit_condition: "Gate4 public contract/payload/claim 变化", evidence_path: "process/stories/STORY-CR169-S04-strict-c3-c4-gate4-joint-adapter-LLD.md"}
file_ownership: {primary: ["engine/capacity_liquidity_gate4_projection.py", "tests/research/test_capacity_liquidity_gate4_projection.py"], shared: [], merge_owner: "CR169-S04-strict-c3-c4-gate4-joint-adapter", forbidden: ["engine/cross_strategy_reliability_gates.py", "engine/economic_cost_gate4_projection.py", "engine/strategy_admission_package.py"]}
lld_gate: {design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR169-S04-strict-c3-c4-gate4-joint-adapter-LLD.md", status: "approved"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 4
created_at: "2026-07-14T19:05:00+08:00"
updated_at: "2026-07-15T10:16:00+08:00"
---

## 目标与验收

创建 CR169 local strict adapter：verified C3+C4 → exact 13-field correlation → exact 7-key payload → public canonical call → fixture-only postcondition。固定 `validate_gate4_capacity_impact(..., release_profile="candidate-release")`；测试以 Protocol DI callable double。

- [ ] payload=7 keys；C3=4、C4=3；extra/reason escape acceptance=0。
- [ ] precheck failure canonical calls=0；valid call=1。
- [ ] canonical、CR168 adapter、aggregate source modifications=0；aggregate/real-ready/Stage3 PASS=0。
