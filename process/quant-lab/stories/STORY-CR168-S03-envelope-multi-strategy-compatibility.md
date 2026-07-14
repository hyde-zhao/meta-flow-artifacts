---
story_id: "CR168-S03-envelope-multi-strategy-compatibility"
title: "Neutral Envelope 激活与多策略兼容"
story_slug: "envelope-multi-strategy-compatibility"
status: "verified-with-risk"
priority: "P0"
wave: "CR168-W3-ENVELOPE-COMPATIBILITY"
depends_on: ["CR168-S01-c3-contract-identity-validation", "CR168-S02-deterministic-cost-producer"]
feature_design_refs: ["docs/features/strategy-evidence-envelope/DESIGN.md", "docs/features/strategy-evidence-envelope/TEST-PLAN.md", "docs/features/strategy-evidence-envelope/TASKS.md", "docs/features/economic-cost-evidence/DESIGN.md", "docs/features/economic-cost-evidence/TEST-PLAN.md"]
feature_refs: ["strategy.evidence.envelope", "economic.cost.evidence"]
feature_reference_ids: ["FEAT-166-01", "FEAT-168-01"]
lld_policy: "full-lld"
lld_policy_detail: {required_level: "full-lld", trigger_reasons: ["shared_contract", "backward_compatibility", "hash_identity", "cross_module"], rationale: "catalog activation 影响 CR166 neutral public contract 与 C1/C2 compatibility。", waiver_reason: "", revisit_condition: "catalog/schema/domain/C1-C2 golden 变化", evidence_path: "process/stories/STORY-CR168-S03-envelope-multi-strategy-compatibility-LLD.md"}
file_ownership: {primary: ["engine/strategy_evidence.py", "tests/research/test_economic_cost_envelope_compatibility.py"], shared: [], merge_owner: "CR168-S03-envelope-multi-strategy-compatibility", forbidden: ["engine/statistical_evidence.py", "engine/walk_forward_oos_evidence.py", "engine/cross_strategy_reliability_gates.py"]}
lld_gate: {required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Story", "S01/S02 outputs"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR168-S03-envelope-multi-strategy-compatibility-LLD.md", status: "confirmed"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-CR168-S03-envelope-multi-strategy-compatibility-IMPLEMENTATION.md", evidence_type: "implementation-md", implementation_objects: ["catalog descriptor", "envelope attach", "compatibility tests"], test_plan_refs: ["docs/features/strategy-evidence-envelope/TEST-PLAN.md"], local_validation_results: ["40/40 targeted tests passed"], status: "passed"}
verification_gate: {validation_mode: "static-only", validation_target: {sut_type: "code-project", native_test_required: true, workflow_eval_required: false, prompt_bundle_required: false, eval_suite_refs: [], prompt_bundle_refs: [], runtime_authorization_required: []}, verification_report_path: "docs/quality/VERIFICATION-REPORT-CR168.md", test_report_path: "docs/quality/TEST-REPORT-CR168.md", review_path: "docs/quality/REVIEW-CR168.md", cp7_result: "PASS_WITH_RISK", validation_object_inventory_status: "story-complete", traceability_matrix_status: "passed", design_contract_verification_status: "passed", layered_validation_plan_status: "passed", remaining_risks: ["R-CR168-VERIFIER-INDEPENDENCE"], route_to: "meta-dev", status: "passed-with-risk"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 3
created_at: "2026-07-14T11:15:00+08:00"
updated_at: "2026-07-14T13:48:00+08:00"
---

## 目标

激活一个 `economic_cost@v1` descriptor，并将 present C3 component attach 到 CR166 neutral envelope；验证 daily/ML 的成本语义等价不被 subject identity 污染。

## 开发上下文

| 项目 | 内容 |
|---|---|
| 上游 | S01 subject-neutral component、S02 present producer |
| 输出 | static catalog activation、attachment integration、daily/ML fixture tests |
| invariant | component hash distinct=1；different subject envelope hash distinct=2；C1/C2 bytes/hash diffs=0 |
| 禁止 | parallel envelope/registry、reverse import C3、C4 activation、Gate4 changes |

### AI 可执行任务清单

| TASK-ID | 动作 | 目标文件 | 描述 |
|---|---|---|---|
| CR168-S03-T01 | 修改 | `engine/strategy_evidence.py` | static catalog 把 reserved slot 激活为 v1，不改变 C1/C2。 |
| CR168-S03-T02 | 创建 | `tests/research/test_economic_cost_envelope_compatibility.py` | daily/ML hash split、identity tamper、catalog compatibility。 |
| CR168-S03-T03 | 修改 | `docs/features/strategy-evidence-envelope/*` | 仅回填 revision/traceability；禁止改既有设计语义，若需要语义变更则回 CP3/新 CR。 |

## 技术说明

component hash 绑定 families 2-9 成本语义；envelope hash 绑定 manifest/run/strategy/package 与 auth/provenance。S03 对既有 Feature 文档只可追加 revision/traceability，不得在实现阶段改设计语义；若 C1/C2 golden 改变，停止 activation、恢复 reserved，不更新 golden 掩盖。CP5 未批准前不得实现。

## 量化验收标准

- [ ] active C3 descriptor=1、C4 descriptor active=0。
- [ ] daily/ML component hash=1、envelope hash=2。
- [ ] C1/C2 regression=0、identity tamper false PASS=0。
