---
story_id: "CR169-S03-neutral-envelope-c4-activation"
title: "Neutral Envelope C4 激活与多策略兼容"
story_slug: "neutral-envelope-c4-activation"
status: "verified-with-risk"
priority: "P0"
wave: "CR169-W3-ENVELOPE-ACTIVATION"
depends_on: ["CR169-S01-capacity-liquidity-contract-validation", "CR169-S02-deterministic-static-c4-producer"]
feature_design_refs: ["docs/features/capacity-liquidity-evidence/DESIGN.md", "docs/features/capacity-liquidity-evidence/TEST-PLAN.md", "docs/features/strategy-evidence-envelope/DESIGN.md", "docs/features/strategy-evidence-envelope/TEST-PLAN.md"]
feature_refs: ["capacity.liquidity.evidence", "strategy.evidence.envelope"]
feature_reference_ids: ["FEAT-169-01", "FEAT-166-01"]
lld_policy: "full-lld"
lld_policy_detail: {required_level: "full-lld", trigger_reasons: ["catalog_activation", "hash_identity", "backward_compatibility"], rationale: "激活 C4 catalog slot 同时必须证明 C1/C2/C3 descriptor 和 envelope hash 兼容。", waiver_reason: "", revisit_condition: "catalog/envelope/hash 变化", evidence_path: "process/stories/STORY-CR169-S03-neutral-envelope-c4-activation-LLD.md"}
file_ownership: {primary: ["engine/strategy_evidence.py", "tests/research/test_capacity_liquidity_envelope_compatibility.py"], shared: [], merge_owner: "CR169-S03-neutral-envelope-c4-activation", forbidden: ["engine/cross_strategy_reliability_gates.py", "engine/economic_cost_gate4_projection.py", "engine/strategy_admission_package.py"]}
lld_gate: {design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR169-S03-neutral-envelope-c4-activation-LLD.md", status: "approved"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 3
created_at: "2026-07-14T19:05:00+08:00"
updated_at: "2026-07-15T09:57:00+08:00"
---

## 目标

将 neutral catalog 的 C4 slot 激活为唯一 `capacity_liquidity@v1` descriptor，并证明 daily/ML attachment 的 component semantics 与 envelope identity 正确分域。

## 边界与验收

只允许 `engine/strategy_evidence.py` 的 C4 descriptor/catalog 增量；不得修改 C1/C2/C3 schema/hash。active C4 descriptor=1、parallel envelope/registry=0；同 computational body 跨 package component hash distinct=1、envelope hash distinct=2；C1/C2/C3 compatibility failures=0。
