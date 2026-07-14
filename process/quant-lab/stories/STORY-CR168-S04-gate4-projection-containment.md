---
story_id: "CR168-S04-gate4-projection-containment"
title: "Gate4 Projection 局部安全封闭"
story_slug: "gate4-projection-containment"
status: "verified-with-risk"
priority: "P0"
wave: "CR168-W4-GATE4-CONTAINMENT"
depends_on: ["CR168-S01-c3-contract-identity-validation", "CR168-S02-deterministic-cost-producer", "CR168-S03-envelope-multi-strategy-compatibility"]
feature_design_refs: ["docs/features/economic-cost-gate4-projection/DESIGN.md", "docs/features/economic-cost-gate4-projection/TEST-PLAN.md", "docs/features/economic-cost-gate4-projection/TASKS.md", "docs/features/economic-cost-evidence/DESIGN.md"]
feature_refs: ["economic.cost.gate4.projection", "economic.cost.evidence"]
feature_reference_ids: ["FEAT-168-02", "FEAT-168-01"]
lld_policy: "full-lld"
lld_policy_detail: {required_level: "full-lld", trigger_reasons: ["security", "cross_module", "public_interface", "claim_sensitive"], rationale: "Gate4 C3+C4 joint semantic、false PASS escape 与 canonical-source prohibition 是最高风险合同。", waiver_reason: "", revisit_condition: "direct caller/C4 present/canonical public contract/aggregate 变化", evidence_path: "process/stories/STORY-CR168-S04-gate4-projection-containment-LLD.md"}
file_ownership: {primary: ["engine/economic_cost_gate4_projection.py", "tests/research/test_economic_cost_gate4_projection.py"], shared: [], merge_owner: "CR168-S04-gate4-projection-containment", forbidden: ["engine/cross_strategy_reliability_gates.py", "engine/strategy_admission_package.py"]}
lld_gate: {required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Story", "S01-S03 outputs"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR168-S04-gate4-projection-containment-LLD.md", status: "confirmed"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-CR168-S04-gate4-projection-containment-IMPLEMENTATION.md", evidence_type: "implementation-md", implementation_objects: ["adapter", "pre/post guards", "double/integration tests"], test_plan_refs: ["docs/features/economic-cost-gate4-projection/TEST-PLAN.md"], local_validation_results: ["55/55 targeted tests passed"], status: "passed"}
verification_gate: {validation_mode: "static-only", validation_target: {sut_type: "code-project", native_test_required: true, workflow_eval_required: false, prompt_bundle_required: false, eval_suite_refs: [], prompt_bundle_refs: [], runtime_authorization_required: []}, verification_report_path: "docs/quality/VERIFICATION-REPORT-CR168.md", test_report_path: "docs/quality/TEST-REPORT-CR168.md", review_path: "docs/quality/REVIEW-CR168.md", cp7_result: "PASS_WITH_RISK", validation_object_inventory_status: "story-complete", traceability_matrix_status: "passed", design_contract_verification_status: "passed", layered_validation_plan_status: "passed", remaining_risks: ["R-CR168-GATE4-C3-C4-SEMANTIC", "R-CR168-VERIFIER-INDEPENDENCE"], route_to: "meta-dev", status: "passed-with-risk"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 4
created_at: "2026-07-14T11:15:00+08:00"
updated_at: "2026-07-14T14:11:00+08:00"
---

## 目标

提供 CR168 唯一 C3-to-Gate4 adapter，封闭 C4 absent+N/A reason 的虚假 PASS 逃逸，且 canonical/aggregate 修改为 0。

## 开发上下文

| 项目 | 内容 |
|---|---|
| 输入 | present typed C3、C4 unavailable marker、zero operation counts |
| 输出 | ProjectionOutcome、canonical summary（不外传 PASS） |
| allowlist | 4 keys：impact family/ref、cost status、no-real-TCA |
| denylist | 8/8 reason key presence，空值也拒绝 |
| postcondition | `candidate-release` 下必须 BLOCKED + `adv_participation_missing` / `capacity_dollars_missing` / `liquidity_sizing_missing`；PASS/claims violation 分 reason |

### AI 可执行任务清单

| TASK-ID | 动作 | 目标文件 | 描述 |
|---|---|---|---|
| CR168-S04-T01 | 创建 | `engine/economic_cost_gate4_projection.py` | typed payload/precheck/zero operation guard。 |
| CR168-S04-T02 | 创建 | 同上 | fixed candidate-release public canonical call + postcondition。 |
| CR168-S04-T03 | 创建 | `tests/research/test_economic_cost_gate4_projection.py` | B01、8-key B02、`gate4_validator=` DI public double 的 PASS/claims violation。 |
| CR168-S04-T04 | 静态检查 | canonical/aggregate source | diff/import/call-site counts remain 0/0/1。 |

## 技术说明

不运行时 import `_has_na_reason`。test double 使用 keyword-only `gate4_validator=` DI，production 使用默认 public validator；禁止 monkeypatch canonical module，fake 只造 public result、不复制 private helper。C4 present 或任何 aggregate 需求路由 FU-005/FU-007。CP5 未批准前不得实现。

## 量化验收标准

- [ ] allowlist=4、denylist=8/8、escape calls=0。
- [ ] B01/B02=2/2；capacity/aggregate PASS=0。
- [ ] canonical PASS reason 与 claim violation reason 各 1 个；private helper dependency=0。
- [ ] canonical/aggregate source modifications=0/0。
