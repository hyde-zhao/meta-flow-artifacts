---
story_id: "CR168-S01-c3-contract-identity-validation"
title: "C3 合同、身份分域与输入校验"
story_slug: "c3-contract-identity-validation"
status: "verified-with-risk"
priority: "P0"
wave: "CR168-W1-CONTRACT-IDENTITY"
depends_on: []
feature_design_refs: ["docs/features/economic-cost-evidence/DESIGN.md", "docs/features/economic-cost-evidence/TEST-PLAN.md", "docs/features/economic-cost-evidence/TASKS.md", "docs/features/strategy-evidence-envelope/DESIGN.md"]
feature_refs: ["economic.cost.evidence", "strategy.evidence.envelope"]
feature_reference_ids: ["FEAT-168-01", "FEAT-166-01"]
lld_policy: "full-lld"
lld_policy_detail:
  required_level: "full-lld"
  trigger_reasons: ["public_contract", "hash_identity", "security", "cross_module"]
  rationale: "九字段族、availability、component/envelope identity 分域和 fail-closed reason code 为后续四 Story 的公共合同。"
  waiver_reason: ""
  revisit_condition: "schema/hash domain 或 attachment identity 变化"
  evidence_path: "process/stories/STORY-CR168-S01-c3-contract-identity-validation-LLD.md"
file_ownership:
  primary: ["engine/economic_cost_evidence.py", "tests/research/test_economic_cost_contracts.py"]
  shared: []
  merge_owner: "CR168-S01-c3-contract-identity-validation"
  forbidden: ["engine/strategy_evidence.py", "engine/cross_strategy_reliability_gates.py", "engine/strategy_admission_package.py"]
lld_gate:
  required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Story"]
  design_evidence_type: "full-lld"
  design_evidence_path: "process/stories/STORY-CR168-S01-c3-contract-identity-validation-LLD.md"
  status: "confirmed"
implementation_gate:
  evidence_required: true
  evidence_path: "process/stories/STORY-CR168-S01-c3-contract-identity-validation-IMPLEMENTATION.md"
  evidence_type: "implementation-md"
  implementation_objects: ["C3 immutable contracts", "validator", "semantic hash"]
  test_plan_refs: ["docs/features/economic-cost-evidence/TEST-PLAN.md"]
  local_validation_results: ["18 S01 contract tests passed", "7 CR166 C2 compatibility tests passed", "25 targeted tests passed after N03 static-assumption repair"]
  status: "completed"
verification_gate:
  validation_mode: "static-only"
  validation_target: {sut_type: "code-project", native_test_required: true, workflow_eval_required: false, prompt_bundle_required: false, eval_suite_refs: [], prompt_bundle_refs: [], runtime_authorization_required: []}
  verification_report_path: "docs/quality/VERIFICATION-REPORT-CR168.md"
  test_report_path: "docs/quality/TEST-REPORT-CR168.md"
  review_path: "docs/quality/REVIEW-CR168.md"
  cp7_result: "process/checks/CP7-CR168-S01-C3-CONTRACT-VERIFICATION.result.json"
  validation_object_inventory_status: "complete"
  traceability_matrix_status: "complete"
  design_contract_verification_status: "complete"
  layered_validation_plan_status: "complete"
  remaining_risks: ["R-CR168-COST-UNDERSTATEMENT"]
  route_to: "CR168-S02-deterministic-cost-producer"
  status: "verified-with-risk"
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 3
created_at: "2026-07-14T11:15:00+08:00"
updated_at: "2026-07-14T12:52:44+08:00"
---

## 目标

创建 C3 `economic_cost@v1` 输入、availability、reason 与 subject-neutral semantic hash 合同；保证九字段族全校验，identity 仅由 envelope 绑定。

## 开发上下文

| 项目 | 内容 |
|---|---|
| 输入 | CP3 HLD/ADR、FEAT-168-01、FEAT-166-01 public canonical contract |
| 输出 | C3 values/validator/hash；contract tests |
| 关键约束 | family 1 identity 不进 component hash；families 2-9 进；不做 I/O/registry/C4/Gate4 |
| 测试入口 | EC-T01、EC-T05..T10 |

### 依赖与文件所有权

无上游依赖。S02 才可扩展 `engine/economic_cost_evidence.py`；其他 Story 只读本 Story 的 public contract。

### AI 可执行任务清单

| TASK-ID | 动作 | 目标文件 | 描述 |
|---|---|---|---|
| CR168-S01-T01 | 创建 | `engine/economic_cost_evidence.py` | immutable input/evidence/availability/domain constants。 |
| CR168-S01-T02 | 创建 | 同上 | 9-family normalization、N01..N10 精确 validation/稳定排序、component semantic hash/self-validation。 |
| CR168-S01-T03 | 创建 | `tests/research/test_economic_cost_contracts.py` | identity separation、minor unit、nonfinite、tamper tests。 |

## 技术说明

设计证据为 full LLD。S01 result 是 `normalized_input + attachment_context + ordered issues` 的 typed 三元；S02 只可消费它，issues 非空时禁止 calculator。失败输入产生 typed_unavailable 或 blocked，不原地提升；C1/C2/envelope canonical 源文件禁止修改。CP5 未批准前不得执行上述实现任务。

## 量化验收标准

- [ ] 9/9 字段族有 required/optional/N/A/auth 规则。
- [ ] 10 类输入 fail-closed 断言=10/10。
- [ ] 同 families 2-9 输入运行 10 次，component hash distinct=1。
- [ ] family 1 identity 不在 component hash，identity tamper false PASS=0。
