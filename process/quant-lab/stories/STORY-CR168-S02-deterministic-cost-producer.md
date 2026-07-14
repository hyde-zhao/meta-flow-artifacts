---
story_id: "CR168-S02-deterministic-cost-producer"
title: "确定性经济成本 Producer"
story_slug: "deterministic-cost-producer"
status: "verified-with-risk"
priority: "P0"
wave: "CR168-W2-CALCULATOR-PRODUCER"
depends_on: ["CR168-S01-c3-contract-identity-validation"]
feature_design_refs: ["docs/features/economic-cost-evidence/DESIGN.md", "docs/features/economic-cost-evidence/TEST-PLAN.md", "docs/features/economic-cost-evidence/TASKS.md"]
feature_refs: ["economic.cost.evidence"]
feature_reference_ids: ["FEAT-168-01"]
lld_policy: "full-lld"
lld_policy_detail: {required_level: "full-lld", trigger_reasons: ["numeric_contract", "public_contract", "fail_closed"], rationale: "Decimal、rounding、square-root、reconciliation 的可重算性和 claim boundary 必须在实现前冻结。", waiver_reason: "", revisit_condition: "公式/舍入/active family 变化", evidence_path: "process/stories/STORY-CR168-S02-deterministic-cost-producer-LLD.md"}
file_ownership: {primary: ["engine/economic_cost_calculator.py", "engine/economic_cost_evidence.py", "tests/research/test_economic_cost_producer.py"], shared: ["engine/economic_cost_evidence.py"], merge_owner: "CR168-S02-deterministic-cost-producer", forbidden: ["engine/strategy_evidence.py", "engine/cross_strategy_reliability_gates.py", "engine/strategy_admission_package.py"]}
lld_gate: {required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Story", "S01 contract"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR168-S02-deterministic-cost-producer-LLD.md", status: "confirmed"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-CR168-S02-deterministic-cost-producer-IMPLEMENTATION.md", evidence_type: "implementation-md", implementation_objects: ["Decimal calculator", "producer", "reconciliation"], test_plan_refs: ["docs/features/economic-cost-evidence/TEST-PLAN.md"], local_validation_results: ["12 S02 producer tests passed", "37 S01/S02/CR166 C2 targeted tests passed"], status: "completed"}
verification_gate: {validation_mode: "static-only", validation_target: {sut_type: "code-project", native_test_required: true, workflow_eval_required: false, prompt_bundle_required: false, eval_suite_refs: [], prompt_bundle_refs: [], runtime_authorization_required: []}, verification_report_path: "docs/quality/VERIFICATION-REPORT-CR168.md", test_report_path: "docs/quality/TEST-REPORT-CR168.md", review_path: "docs/quality/REVIEW-CR168.md", cp7_result: "process/checks/CP7-CR168-S02-DETERMINISTIC-COST-PRODUCER-VERIFICATION.result.json", validation_object_inventory_status: "complete", traceability_matrix_status: "complete", design_contract_verification_status: "complete", layered_validation_plan_status: "complete", remaining_risks: ["R-CR168-COST-UNDERSTATEMENT", "R-CR168-UNIT-CURRENCY-BASIS", "R-CR168-VERIFIER-INDEPENDENCE"], route_to: "CR168-S03-envelope-multi-strategy-compatibility", status: "verified-with-risk"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 3
created_at: "2026-07-14T11:15:00+08:00"
updated_at: "2026-07-14T13:08:00+08:00"
---

## 目标

在 S01 合同上计算 fee/tax/spread/slippage/static square-root impact、total 和 gross-to-net，生成 deterministic typed C3 producer outcome。

## 开发上下文

| 项目 | 内容 |
|---|---|
| 上游 | S01 public contract/hash |
| 输出 | pure calculator、producer/reconciliation、producer tests |
| 数值约束 | Decimal precision=28；proxy finite `>=0` 可 `>1`；raw components sum 后才量化 total/net；minor unit 缺失 blocked |
| 禁止 | real TCA、calibration、rebate、C4/ADV/capacity、I/O |

### AI 可执行任务清单

| TASK-ID | 动作 | 目标文件 | 描述 |
|---|---|---|---|
| CR168-S02-T01 | 创建 | `engine/economic_cost_calculator.py` | 五分项 exact basis/impact/reconciliation pure Decimal routines。 |
| CR168-S02-T02 | 修改 | `engine/economic_cost_evidence.py` | producer 固定 normalize→validate→issue short-circuit→calculator→produce。 |
| CR168-S02-T03 | 创建 | `tests/research/test_economic_cost_producer.py` | formula、proxy、rounding、arithmetic failure tests。 |

## 技术说明

v1 只允许 `square_root` present。issue 非空时 calculator calls=0；fee/tax/spread/slippage/impact basis 与 `net_return=quantized_net_pnl/performance_notional` 以 S02 LLD 为准。unsupported/N/A 不投影；任一公式/舍入变更必须新 schema + method CR。CP5 未批准前不得实现。

## 量化验收标准

- [ ] five components/total/net 可重算，误差≤1 final quantization unit。
- [ ] proxy 0/1/>1 3/3 可算；negative/nonfinite 0 accepted。
- [ ] missing/nonpositive minor unit 3/3 blocked。
- [ ] real TCA/calibration/capacity claim=0。
