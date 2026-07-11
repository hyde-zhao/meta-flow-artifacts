---
story_id: "CR163-S03-two-producer-chain-instrumentation"
title: "Two producer chains and four instrumentation mappings"
story_slug: "two-producer-chain-instrumentation"
status: "verified-with-risk"
priority: "P0"
wave: "CR163-W3-PRODUCERS-CONSUMERS"
depends_on: ["CR163-S01-family-contract-validator", "CR163-S02-recorder-seal-supersession"]
dependency_contracts:
  - {upstream: "CR163-S01-family-contract-validator", type: "contract", lld_gate: "upstream_contract_declared", dev_gate: "upstream_contract_confirmed"}
  - {upstream: "CR163-S02-recorder-seal-supersession", type: "contract", lld_gate: "upstream_contract_declared", dev_gate: "upstream_contract_confirmed"}
feature_design_refs: ["docs/features/trial-lineage-producer-adapters/DESIGN.md", "docs/features/trial-lineage-producer-adapters/TEST-PLAN.md", "docs/features/trial-lineage-producer-adapters/TASKS.md", "docs/features/experiment-family-lineage/DESIGN.md"]
feature_refs: ["FEAT-21", "FEAT-20"]
lld_policy: "full-lld"
lld_policy_detail: {required_level: "full-lld", trigger_reasons: ["cross-module", "two-producer-chains", "identity-ownership", "four-mapping-coverage"], rationale: "Four call sites share one identity contract across two chains and require single-writer coordination.", evidence_path: "process/stories/STORY-CR163-S03-two-producer-chain-instrumentation-LLD.md"}
file_ownership:
  primary: ["engine/mature_multifactor_research.py", "engine/multifactor_strategy_candidates.py", "scripts/research/run_multifactor_strategy_research.py", "scripts/legacy/research/run_multifactor_strategy_candidates.py", "tests/test_cr163_trial_lineage_producer_adapters.py"]
  shared: []
  merge_owner: "CR163-S03-two-producer-chain-instrumentation"
  forbidden: ["engine/strategy_admission_statistical_gate.py", "engine/cross_strategy_reliability_gates.py", "engine/strategy_admission_package.py", "data/**", "credentials", "external runtime"]
lld_gate: {required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Story", "S01/S02 contracts", "FEAT-21 DESIGN/TEST-PLAN/TASKS"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR163-S03-two-producer-chain-instrumentation-LLD.md", status: "approved"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-CR163-S03-two-producer-chain-instrumentation-IMPLEMENTATION.md", evidence_type: "implementation-md", implementation_objects: ["code", "producer-adapter", "guardrail-test"], test_plan_refs: ["docs/features/trial-lineage-producer-adapters/TEST-PLAN.md"], local_validation_results: [], status: "not-started"}
verification_gate: {validation_mode: "static-only", validation_target: {sut_type: "code-project", native_test_required: true, workflow_eval_required: false, prompt_bundle_required: false, eval_suite_refs: [], prompt_bundle_refs: [], runtime_authorization_required: []}, cp7_result: "not-started", remaining_risks: [], route_to: "", status: "not-started"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 4
created_at: "2026-07-11T12:05:00+08:00"
updated_at: "2026-07-11T12:05:00+08:00"
---

## 目标

在单一 Story 内接入 public Stage3 与 legacy CR039 两条去重 producer chains，并覆盖 CPI-CR163-001..004 四个 instrumentation mappings，消除 wrapper/hook 双计数与绕过。

## 开发上下文（dev_context）

- 输出：两个 engine、两个 wrappers 的显式 session/identity/selection mapping。
- 约束：orchestration 是 session owner，wrapper透传；hook只允许一个 selection writer策略；不访问真实数据/runtime。

### 依赖与并行门控

S01/S02 为 contract 依赖。S03 可与 S04 在 CP5批准且上游 contracts confirmed 后并行开发；二者 primary 文件无交集。

### AI 可执行任务清单

| TASK-ID | 动作 | 目标 |
|---|---|---|
| TASK-CR163-S03-01 | 冻结 | CPI-001/003 public chain mapping |
| TASK-CR163-S03-02 | 冻结 | CPI-002/004 legacy chain mapping |
| TASK-CR163-S03-03 | 修改（CP5 后） | 四个 call sites与single session owner |
| TASK-CR163-S03-04 | 创建（CP5 后） | 4/4 coverage/no-double-count fixtures |

## 量化验收标准（acceptance_criteria）

- [ ] CPI-CR163-001..004 mapping coverage=4/4；deduplicated chain coverage=2/2。
- [ ] 每条 chain session owner count=1；wrapper/hook duplicate trial count=0。
- [ ] 两条 chain declaration 均严格早于 first trial，post-hoc fixture 2/2 blocked。
- [ ] same trial 3 attempts raw=1；different seed trials raw=2。
- [ ] failed/cancelled/excluded trial retention=100%，forbidden operation counters=0。

## 阻塞说明

实现被 CP5 与 S01/S02 contracts 阻断；无规划阻塞。
