---
story_id: "CR163-S01-family-contract-validator"
title: "Experiment-family contract and validator"
story_slug: "family-contract-validator"
status: "verified"
priority: "P0"
wave: "CR163-W1-CONTRACT-VALIDATOR"
depends_on: []
dependency_contracts: []
feature_design_refs:
  - "docs/features/experiment-family-lineage/DESIGN.md"
  - "docs/features/experiment-family-lineage/TEST-PLAN.md"
  - "docs/features/experiment-family-lineage/TASKS.md"
feature_refs: ["FEAT-20"]
lld_policy: "full-lld"
lld_policy_detail:
  required_level: "full-lld"
  trigger_reasons: ["public-contract", "data-state", "security", "cross-story-contract"]
  rationale: "Six shared objects, lifecycle validation and availability projection are contracts for S02-S05."
  waiver_reason: ""
  revisit_condition: ""
  evidence_path: "process/stories/STORY-CR163-S01-family-contract-validator-LLD.md"
file_ownership:
  primary: ["engine/experiment_family_lineage.py", "tests/test_experiment_family_lineage_contracts.py"]
  shared: []
  merge_owner: "CR163-S01-family-contract-validator"
  forbidden: ["engine/mature_multifactor_research.py", "engine/multifactor_strategy_candidates.py", "engine/strategy_admission_statistical_gate.py", "engine/cross_strategy_reliability_gates.py", "engine/strategy_admission_package.py", "data/**", "runtime/**"]
lld_gate:
  required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Story", "FEAT-20 DESIGN/TEST-PLAN/TASKS"]
  design_evidence_type: "full-lld"
  design_evidence_path: "process/stories/STORY-CR163-S01-family-contract-validator-LLD.md"
  status: "approved"
implementation_gate:
  evidence_required: true
  evidence_path: "process/stories/STORY-CR163-S01-family-contract-validator-IMPLEMENTATION.md"
  evidence_type: "implementation-md"
  implementation_objects: ["code", "contract", "guardrail-test"]
  test_plan_refs: ["docs/features/experiment-family-lineage/TEST-PLAN.md"]
  local_validation_results: []
  status: "not-started"
verification_gate:
  validation_mode: "static-only"
  validation_target: {sut_type: "code-project", native_test_required: true, workflow_eval_required: false, prompt_bundle_required: false, eval_suite_refs: [], prompt_bundle_refs: [], runtime_authorization_required: []}
  cp7_result: "PASS"
  remaining_risks: []
  route_to: ""
  status: "verified"
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 3
created_at: "2026-07-11T12:05:00+08:00"
updated_at: "2026-07-11T12:05:00+08:00"
---

## 目标

建立六个持久化 experiment-family lineage 对象、`FamilyLineageSession` façade、typed commands、state/count/error/validation/projection 公共 contract，作为 S02-S05 的唯一契约基础。

## 开发上下文（dev_context）

- 输入：approved HLD/ADR、FEAT-20 三件套、REQ-001..003/006..008。
- 输出：`engine/experiment_family_lineage.py` 与 contract fixture tests。
- 约束：`ExperimentManifest` 保持单次 run；core 不 import producer/consumer；effective count不计算。
- 平台目标：本地 Python/uv，fixture/static only。

### 依赖与并行门控

无上游 Story。S01 contract 未经 CP5 确认前，S02-S04 不可进入开发。

### 文件所有权

| 类型 | 文件 | Owner / 规则 |
|---|---|---|
| primary | `engine/experiment_family_lineage.py`、`tests/test_experiment_family_lineage_contracts.py` | S01 独占 |
| forbidden | producer/consumer modules、data/runtime paths | 不修改 |

### AI 可执行任务清单

| TASK-ID | 动作 | 目标 | 描述 |
|---|---|---|---|
| TASK-CR163-S01-01 | 冻结 | S01 LLD | 对象字段、enums、commands、blocked codes、state transitions |
| TASK-CR163-S01-02 | 创建（CP5 后） | core module | immutable DTO、session interface、pure validator/projection |
| TASK-CR163-S01-03 | 创建（CP5 后） | contract tests | legal/illegal state、identity/count/availability fixtures |

## 技术说明

设计证据类型为 full-lld；本卡不替代 LLD。LLD 必须冻结六对象 6/6、target ref/hash validation、typed unavailable/blocked mapping、no-effective/no-runtime guard。

## 量化验收标准（acceptance_criteria）

- [ ] 六个持久化对象覆盖 6/6，façade 恰为 1 个非持久化应用对象。
- [ ] 合法与非法 family/trial/attempt transitions 均有确定 machine result，未映射转换数为 0。
- [ ] raw count 仅由 distinct stable trial ids 得出；attempt/selection/wrapper call 不增加 count。
- [ ] effective available 声明=0，effective ref/method 非空=0，forbidden operation counters=0。
- [ ] S02-S05 所需 public interfaces 均有字段、错误与降级契约。

## 阻塞说明

实现被 CP5 human gate 阻断；无规划阻塞。
