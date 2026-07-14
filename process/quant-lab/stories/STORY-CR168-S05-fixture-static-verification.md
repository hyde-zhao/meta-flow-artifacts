---
story_id: "CR168-S05-fixture-static-verification"
title: "Fixture 静态验证、授权与 Claim Regression"
story_slug: "fixture-static-verification"
status: "verified-with-risk"
priority: "P0"
wave: "CR168-W5-STATIC-VERIFICATION"
depends_on: ["CR168-S01-c3-contract-identity-validation", "CR168-S02-deterministic-cost-producer", "CR168-S03-envelope-multi-strategy-compatibility", "CR168-S04-gate4-projection-containment"]
feature_design_refs: ["docs/features/economic-cost-verification/DESIGN.md", "docs/features/economic-cost-verification/TEST-PLAN.md", "docs/features/economic-cost-verification/TASKS.md", "docs/features/economic-cost-evidence/TEST-PLAN.md", "docs/features/strategy-evidence-envelope/TEST-PLAN.md", "docs/features/economic-cost-gate4-projection/TEST-PLAN.md"]
feature_refs: ["economic.cost.verification", "economic.cost.evidence", "strategy.evidence.envelope", "economic.cost.gate4.projection"]
feature_reference_ids: ["FEAT-168-03", "FEAT-168-01", "FEAT-166-01", "FEAT-168-02"]
lld_policy: "full-lld"
lld_policy_detail: {required_level: "full-lld", trigger_reasons: ["cross_feature", "security", "claim_sensitive", "regression"], rationale: "15 QAC、17 scenarios、CR155 blocked、forbidden ops 与 full-suite attribution 必须作为独立可审查验证合同。", waiver_reason: "", revisit_condition: "新 runtime/data/real TCA lane 或 independent verifier service", evidence_path: "process/stories/STORY-CR168-S05-fixture-static-verification-LLD.md"}
file_ownership: {primary: ["tests/fixtures/economic_cost/", "tests/research/test_economic_cost_cr168_qac.py", "tests/research/test_economic_cost_authorization.py", "tests/research/test_economic_cost_cr155_regression.py"], shared: [], merge_owner: "CR168-S05-fixture-static-verification", forbidden: ["engine/economic_cost_evidence.py", "engine/economic_cost_calculator.py", "engine/strategy_evidence.py", "engine/economic_cost_gate4_projection.py", "engine/cross_strategy_reliability_gates.py", "engine/strategy_admission_package.py"]}
lld_gate: {required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Story", "S01-S04 outputs"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR168-S05-fixture-static-verification-LLD.md", status: "confirmed"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-CR168-S05-fixture-static-verification-IMPLEMENTATION.md", evidence_type: "implementation-md", implementation_objects: ["fixtures", "QAC", "authorization regression", "full-suite attribution"], test_plan_refs: ["docs/features/economic-cost-verification/TEST-PLAN.md"], local_validation_results: ["98 targeted tests passed", "5 governance tests passed", "2077 repository tests passed"], status: "completed"}
verification_gate: {validation_mode: "static-only", validation_target: {sut_type: "code-project", native_test_required: true, workflow_eval_required: false, prompt_bundle_required: false, eval_suite_refs: [], prompt_bundle_refs: [], runtime_authorization_required: []}, verification_report_path: "docs/quality/VERIFICATION-REPORT-CR168.md", test_report_path: "docs/quality/TEST-REPORT-CR168.md", review_path: "docs/quality/REVIEW-CR168.md", cp7_result: "process/checks/CP7-CR168-S05-FIXTURE-STATIC-VERIFICATION-VERIFICATION.result.json", validation_object_inventory_status: "complete", traceability_matrix_status: "complete", design_contract_verification_status: "complete", layered_validation_plan_status: "complete", remaining_risks: ["R-CR168-VERIFIER-INDEPENDENCE", "R-CR168-GATE4-C3-C4-SEMANTIC", "R-CR168-TRUE-TCA-OVERCLAIM"], route_to: "CP8", status: "verified-with-risk"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 4
created_at: "2026-07-14T11:15:00+08:00"
updated_at: "2026-07-14T15:00:00+08:00"
---

## 目标

在不执行真实数据/runtime 的条件下，建立两 fixture、17 scenario/15 QAC/10 P0 failure、Gate4、authorization、CR155 与 claim ceiling 的可追溯验证面。

## 开发上下文

| 项目 | 内容 |
|---|---|
| 上游 | S01-S04 public contracts/fixtures |
| 输出 | fixtures、QAC/auth/CR155 regression tests、CP7 suite attribution evidence |
| 验证模式 | static-only；本 Story 生产 test fixture，不生产 engine code |
| 关键边界 | real data/TCA/C4/event/runtime/aggregate=0；CR155 remains BLOCKED + paper_candidate=false |

### AI 可执行任务清单

| TASK-ID | 动作 | 目标文件 | 描述 |
|---|---|---|---|
| CR168-S05-T01 | 创建 | `tests/fixtures/economic_cost/` | daily multifactor + multi-strategy synthetic fixture 2/2。 |
| CR168-S05-T02 | 创建 | `tests/research/test_economic_cost_cr168_qac.py` | 17/15/10/10/10→1 assertions。 |
| CR168-S05-T03 | 创建 | `tests/research/test_economic_cost_authorization.py` | forbidden operation/source/doc guard；capability registry missing 的 N/A-with-reason / existing Feature-module refs 断言。 |
| CR168-S05-T04 | 创建 | `tests/research/test_economic_cost_cr155_regression.py` | CR155 blocked/paper_candidate/claim ceiling regression。 |

## 技术说明

全仓 suite 仅在 CP7 运行；失败逐项 attribution，unknown 不得 PASS。capability registry missing 由 authorization suite 验证 N/A-with-reason + existing Feature/module refs，persistent/parallel registry 必须为 0。若仍 inline verification，CP8 必须披露风险但不得伪造 independent lane。

## 量化验收标准

- [ ] requirements/scenarios/QAC=9/9、17/17、15/15。
- [ ] fixture families=2/2、negative=10/10、10 runs→1 hash。
- [ ] C4/capacity/aggregate PASS=0；CR155 promotion=0。
- [ ] forbidden operations、wrong quality ref、parallel registry/gate/envelope=0。
