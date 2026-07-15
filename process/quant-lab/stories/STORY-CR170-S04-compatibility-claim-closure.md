---
story_id: "CR170-S04-compatibility-claim-closure"
title: "Canonical compatibility、adapter 与 claim ceiling 回归"
story_slug: "compatibility-claim-closure"
status: "verified-with-risk"
priority: "P0"
wave: "CR170-W4-COMPATIBILITY"
depends_on: ["CR170-S02-gate1-5-na-consumer-hardening", "CR170-S03-admission-worst-state-tier-hardening"]
dependency_contracts: [{story_id: "CR170-S02-gate1-5-na-consumer-hardening", type: "runtime", required_state: "implemented", contract: "Gate1-5 hardening"}, {story_id: "CR170-S03-admission-worst-state-tier-hardening", type: "runtime", required_state: "implemented", contract: "admission tier hardening"}]
feature_design_refs: ["docs/features/cross-strategy-reliability-gates/DESIGN.md", "docs/features/cross-strategy-reliability-gates/TEST-PLAN.md", "docs/features/cross-strategy-reliability-gates/TASKS.md"]
feature_refs: ["cross.strategy.reliability.compatibility.claim.guard"]
feature_reference_ids: ["FEAT-15"]
lld_policy: "full-lld"
lld_policy_detail: {required_level: "full-lld", trigger_reasons: ["cross_component_regression", "claim_ceiling", "security_boundary"], rationale: "必须证明 canonical hardening 不破坏 CR168/169 defense-in-depth，且不产生 Stage3/aggregate/CR155 overclaim。", waiver_reason: "", revisit_condition: "adapter/public schema/claim ceiling/runner integration 变化", evidence_path: "process/stories/STORY-CR170-S04-compatibility-claim-closure-LLD.md"}
file_ownership: {primary: ["tests/research/test_canonical_reliability_regression.py"], shared: [], merge_owner: "CR170-S04-compatibility-claim-closure", forbidden: ["engine/cross_strategy_reliability_gates.py", "engine/economic_cost_gate4_projection.py", "engine/capacity_liquidity_gate4_projection.py", "engine/strategy_admission_package.py", "engine/mature_multifactor_research.py", "engine/mature_multifactor_framework.py"]}
lld_gate: {required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Story", "S02 LLD", "S03 LLD"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR170-S04-compatibility-claim-closure-LLD.md", status: "approved"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-CR170-S04-compatibility-claim-closure-IMPLEMENTATION.md", evidence_type: "implementation-md", implementation_objects: ["guardrail-test", "docs-handoff", "repository-governance-integration"], test_plan_refs: ["docs/features/cross-strategy-reliability-gates/TEST-PLAN.md"], local_validation_results: ["91 related tests passed", "27 governance tests passed", "py_compile PASS", "diff-check PASS", "forbidden production diff=0"], status: "implemented"}
verification_gate: {validation_mode: "static-only", validation_target: {sut_type: "code-project", native_test_required: true, workflow_eval_required: false, prompt_bundle_required: false, eval_suite_refs: [], prompt_bundle_refs: [], runtime_authorization_required: []}, cp7_result: "PASS_WITH_RISK", remaining_risks: ["R-CR170-VERIFIER-INDEPENDENCE", "R-CR170-RUNNER-GAP"], status: "verified-with-risk"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 4
created_at: "2026-07-15T14:40:00+08:00"
updated_at: "2026-07-15T16:08:00+08:00"
---

## 目标

用只读回归证明 canonical public contract 与 CR-168/169 adapters 保持兼容，并把 CR155、Stage3、aggregate、真实数据/运行授权的 claim ceiling 固化为零提升。

## 开发上下文与边界

- adapters 是 defense-in-depth，本 Story 修改 adapter 生产文件数=`0`、guard 删除数=`0`。
- current Stage3 runner integration=`0`；aggregate orchestration=`0`；CR155 promotion=`0`。
- 独立验证者是 FU-006 future consumer；CR-170 由 Gate maintainer 自验证，CP8 必须披露风险。
- 只允许 repository-local fixture/static tests，不读取真实数据、credentials 或外部服务。

## 依赖与并行门控

等待 S02/S03 实现与局部测试完成，再执行兼容与 claim 回归。

## AI 可执行任务清单

| TASK-ID | 动作 | 目标文件 | 完成准则 |
|---|---|---|---|
| CR170-S04-T01 | 创建 | `tests/research/test_canonical_reliability_regression.py` | public callable/Gate IDs 与 adapter 2/2 回归。 |
| CR170-S04-T02 | 创建 | 同上 | CR155 blocked、paper_candidate=false、Stage3/aggregate/real-op counters=0。 |
| CR170-S04-T03 | 创建 | 同上 | 添加 public-callable evidence→Gate NR→merge NR→T1 BLOCKED 端到端 fixture。 |
| CR170-S04-T04 | 验证 | 既有相关 tests | canonical/adapters/claim regression 无新增失败。 |

## 量化验收标准

- [x] public callable/Gate ID compatibility=`100%`；public break=`0`。
- [x] CR168/CR169 adapter regression=`2/2`；adapter production modifications/guard deletion=`0/0`。
- [x] CR155 promotion=`0`、paper_candidate=`false`、runner/aggregate integration=`0/0`。
- [x] real data/credential/runtime/trading/publish/remote write=`0`。
- [x] public-callable end-to-end fixture=`1/1`；Gate/merge/T1=`NEEDS_REVIEW/NEEDS_REVIEW/BLOCKED`，PASS=`0`。

## 阻塞说明

无 blocker；CP5 已批准，CP6 PASS、CP7 `PASS_WITH_RISK`，剩余 verifier independence 与 runner gap 在 CP8 披露。
