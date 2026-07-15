---
story_id: "CR169-S05-fixture-claim-stage2-exit-verification"
title: "Fixture、Claim 与 Stage2 Exit 验证"
story_slug: "fixture-claim-stage2-exit-verification"
status: "verified-with-risk"
priority: "P0"
wave: "CR169-W5-FIXTURE-CLAIM-EXIT"
depends_on: ["CR169-S01-capacity-liquidity-contract-validation", "CR169-S02-deterministic-static-c4-producer", "CR169-S03-neutral-envelope-c4-activation", "CR169-S04-strict-c3-c4-gate4-joint-adapter"]
feature_design_refs: ["docs/features/capacity-liquidity-claim-exit-guard/DESIGN.md", "docs/features/capacity-liquidity-claim-exit-guard/TEST-PLAN.md", "docs/features/capacity-liquidity-claim-exit-guard/TASKS.md", "docs/features/capacity-liquidity-evidence/TEST-PLAN.md", "docs/features/c3-c4-correlation-boundary/TEST-PLAN.md", "docs/features/capacity-liquidity-gate4-joint-adapter/TEST-PLAN.md"]
feature_refs: ["capacity.liquidity.claim.exit.guard", "capacity.liquidity.evidence", "c3.c4.correlation.boundary", "capacity.liquidity.gate4.joint.adapter"]
feature_reference_ids: ["FEAT-169-04", "FEAT-169-01", "FEAT-169-02", "FEAT-169-03"]
lld_policy: "full-lld"
lld_policy_detail: {required_level: "full-lld", trigger_reasons: ["cross_feature_verification", "claim_ceiling", "authorization", "stage_transition"], rationale: "17 scenarios、15 QAC、CR155/CR168 回归与 Stage2 7/7 exit 证据属于高风险 claim 验证。", waiver_reason: "", revisit_condition: "claim/Stage2 contract/verification lane 变化", evidence_path: "process/stories/STORY-CR169-S05-fixture-claim-stage2-exit-verification-LLD.md"}
file_ownership: {primary: ["tests/fixtures/capacity_liquidity/", "tests/research/test_capacity_liquidity_cr169_qac.py", "tests/research/test_capacity_liquidity_claim_regression.py", "scripts/check_stage2_exit_contracts.py"], shared: [], merge_owner: "CR169-S05-fixture-claim-stage2-exit-verification", forbidden: ["engine/cross_strategy_reliability_gates.py", "engine/economic_cost_gate4_projection.py", "engine/strategy_admission_package.py"]}
lld_gate: {design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR169-S05-fixture-claim-stage2-exit-verification-LLD.md", status: "approved"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 4
created_at: "2026-07-14T19:05:00+08:00"
updated_at: "2026-07-15T10:09:00+08:00"
---

## 目标与失败分流

用 2/2 fixture 与完整负向矩阵证明 C4 foundation 的确定性、安全、消费兼容和 claim ceiling；定义 CP8 Stage2 7/7 result shape。历史前 6 项合同失败时路由 CR-157 owner 或新治理 CR，不由 CR-169 修改。

- [x] REQ=9/9、scenarios=17/17、QAC=15/15、P0=12/12、fixtures=2/2、10→1。
- [x] CR168 absent regression=1；strict joint fixture pass=1；canonical/aggregate modifications=0。
- [x] CR155 BLOCKED、paper_candidate=false；Stage2 result items=7；Stage3/real/alpha/external operations=0。
