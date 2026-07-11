---
story_id: "CR163-S05-integrity-recovery-permission-regression"
title: "Integrity, recovery, permission and CR155 regression"
story_slug: "integrity-recovery-permission-regression"
status: "verified-with-risk"
priority: "P0"
wave: "CR163-W4-INTEGRITY-REGRESSION"
depends_on: ["CR163-S01-family-contract-validator", "CR163-S02-recorder-seal-supersession", "CR163-S03-two-producer-chain-instrumentation", "CR163-S04-existing-admission-projection"]
dependency_contracts:
  - {upstream: "CR163-S01-family-contract-validator", type: "runtime", lld_gate: "upstream_contract_declared", dev_gate: "upstream_ready_for_verification"}
  - {upstream: "CR163-S02-recorder-seal-supersession", type: "runtime", lld_gate: "upstream_contract_declared", dev_gate: "upstream_ready_for_verification"}
  - {upstream: "CR163-S03-two-producer-chain-instrumentation", type: "runtime", lld_gate: "upstream_story_declared", dev_gate: "upstream_ready_for_verification"}
  - {upstream: "CR163-S04-existing-admission-projection", type: "runtime", lld_gate: "upstream_story_declared", dev_gate: "upstream_ready_for_verification"}
feature_design_refs: ["docs/features/experiment-family-lineage/DESIGN.md", "docs/features/experiment-family-lineage/TEST-PLAN.md", "docs/features/experiment-family-lineage/TASKS.md", "docs/features/trial-lineage-producer-adapters/DESIGN.md", "docs/features/trial-lineage-producer-adapters/TEST-PLAN.md", "docs/features/trial-lineage-producer-adapters/TASKS.md", "docs/features/strategy-admission-lineage-projection/DESIGN.md", "docs/features/strategy-admission-lineage-projection/TEST-PLAN.md", "docs/features/strategy-admission-lineage-projection/TASKS.md"]
feature_refs: ["FEAT-20", "FEAT-21", "FEAT-22", "FEAT-23"]
lld_policy: "full-lld"
lld_policy_detail: {required_level: "full-lld", trigger_reasons: ["cross-module-fixtures", "tamper-security", "recovery", "permission-boundary", "negative-regression"], rationale: "S05 composes all three Feature contracts and validates security/tamper/CR155 boundaries; technical-note is insufficient.", evidence_path: "process/stories/STORY-CR163-S05-integrity-recovery-permission-regression-LLD.md"}
file_ownership:
  primary: ["tests/test_cr163_trial_lineage_integrity.py", "tests/test_cr163_trial_lineage_authorization.py", "tests/test_cr163_trial_lineage_cr155_regression.py"]
  shared: []
  merge_owner: "CR163-S05-integrity-recovery-permission-regression"
  forbidden: ["engine/**", "scripts/**", "data/**", "credentials", "runtime", "historical lineage backfill", "external systems"]
lld_gate: {required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Story", "S01-S04 Story contracts", "all three Feature TEST-PLANs"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR163-S05-integrity-recovery-permission-regression-LLD.md", status: "approved"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-CR163-S05-integrity-recovery-permission-regression-IMPLEMENTATION.md", evidence_type: "implementation-md", implementation_objects: ["guardrail-test", "fixture", "docs-handoff"], test_plan_refs: ["docs/features/experiment-family-lineage/TEST-PLAN.md", "docs/features/trial-lineage-producer-adapters/TEST-PLAN.md", "docs/features/strategy-admission-lineage-projection/TEST-PLAN.md"], local_validation_results: ["33 focused tests passed", "262 full packet tests passed", "return/evidence checks OK"], status: "implemented-rework-round-2"}
verification_gate: {validation_mode: "static-only", validation_target: {sut_type: "code-project", native_test_required: true, workflow_eval_required: false, prompt_bundle_required: false, eval_suite_refs: [], prompt_bundle_refs: [], runtime_authorization_required: []}, cp7_result: "PASS_WITH_RISK", remaining_risks: ["Future producer retry-loop changes require a same-session/same-trial ordinal integration fixture", "Effective trials and C1 computation remain out of scope"], route_to: "CP8", status: "verified-with-risk"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 3
created_at: "2026-07-11T12:05:00+08:00"
updated_at: "2026-07-11T17:56:00+08:00"
---

## 目标

以 fixture/static 方式闭合 8/8 requirements、12/12 P0 scenarios、determinism/tamper/supersession、permission counters 与 CR155 blocked regression，不修改任何生产模块或历史数据。

## 开发上下文（dev_context）

- 输入：S01-S04 contracts/implementation evidence、三套 Feature TEST-PLAN。
- 输出：三个独占测试文件与 traceability evidence。
- 约束：不得访问真实 lake/NAS/provider/credential/runtime/trading/external systems；不得 backfill CR155。

### 依赖与并行门控

S01-S04 全部为 runtime dependencies；LLD 可在 CP5 批次提前起草，S05 实现必须等待上游达到其 LLD 指定的 ready-for-verification/verified gate。

### AI 可执行任务清单

| TASK-ID | 动作 | 目标 |
|---|---|---|
| TASK-CR163-S05-01 | 冻结 | 12-scenario fixture/trace matrix |
| TASK-CR163-S05-02 | 创建（CP5 后） | integrity/recovery/permission/CR155 tests |
| TASK-CR163-S05-03 | 验证（CP5 后） | quantitative exits and zero forbidden counters |

## 量化验收标准（acceptance_criteria）

- [ ] REQ coverage=8/8；SC-CR163 P0 planned/executed coverage=12/12。
- [ ] CPI mappings=4/4、chains=2/2、unexplained orphan/count mismatch=0。
- [ ] identical seal 10次 distinct hash=1；negative fixture classes 5/5 blocked；valid supersession=100%。
- [ ] forbidden operation counters全部=0；任何非0 fixture结论=blocked。
- [ ] CR155 blocked=1/1、paper_candidate=false、historical family/trial reconstruction=0。
- [ ] effective available claims=0、C1 computed evidence claims=0、runtime-ready claims=0。

## 阻塞说明

实现被 CP5 与 S01-S04 runtime gates 阻断；无规划阻塞。
