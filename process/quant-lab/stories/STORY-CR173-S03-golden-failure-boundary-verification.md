---
story_id: "CR173-S03-golden-failure-boundary-verification"
title: "Golden-vector, failure-recovery, and public-boundary verification"
story_slug: "golden-failure-boundary-verification"
status: "ready-for-verification"
priority: "P0"
wave: "CR173-W3-GOLDEN-BOUNDARY"
depends_on:
  - "CR173-S01-contract-evidence-canonicalization"
  - "CR173-S02-exact-spectral-estimator"
dependency_contracts:
  - upstream: "CR173-S01-contract-evidence-canonicalization"
    type: "runtime"
    lld_gate: "upstream_contract_declared"
    dev_gate: "upstream_implementation_evidence_passed"
  - upstream: "CR173-S02-exact-spectral-estimator"
    type: "runtime"
    lld_gate: "upstream_algorithm_contract_declared"
    dev_gate: "upstream_implementation_evidence_passed"
feature_design_refs:
  - "docs/features/effective-trial-offline-estimator/DESIGN.md"
  - "docs/features/effective-trial-offline-estimator/TEST-PLAN.md"
  - "docs/features/effective-trial-offline-estimator/TASKS.md"
feature_refs:
  - "effective.trial.offline.estimator"
feature_reference_ids:
  - "FEAT-173-01"
lld_policy: "full-lld"
lld_policy_detail:
  required_level: "full-lld"
  trigger_reasons:
    - "cross-story verification contract"
    - "authorization and forbidden-path guard"
    - "append-only recovery and deterministic evidence"
    - "public claim ceiling and regression inventory"
  rationale: "6×3 oracle、8类 failure、existing public regressions 与 zero-operation guard 涉及跨模块安全结论，必须在 CP5 前冻结验证对象和失败路由。"
  waiver_reason: ""
  revisit_condition: "fixture schema、scenario set、public inventory、authorization boundary、release claim 或 upstream contract 变化。"
  evidence_path: "process/stories/STORY-CR173-S03-golden-failure-boundary-verification-LLD.md"
file_ownership:
  primary:
    - "tests/fixtures/effective_trial/"
    - "tests/research/test_effective_trial_cr173_qac.py"
    - "tests/research/test_effective_trial_authorization.py"
  shared: []
  read_only:
    - "engine/effective_trial_evidence.py"
    - "engine/effective_trial_estimator.py"
    - "engine/experiment_family_lineage.py"
    - "engine/experiment_family_lineage_store.py"
    - "engine/strategy_admission_statistical_gate.py"
    - "engine/statistical_evidence.py"
    - "engine/multiple_testing_evidence.py"
    - "engine/overfit_evidence.py"
    - "engine/cross_strategy_reliability_gates.py"
    - "engine/strategy_admission_package.py"
  merge_owner: "CR173-S03-golden-failure-boundary-verification"
  forbidden:
    - "engine/effective_trial_evidence.py"
    - "engine/effective_trial_estimator.py"
    - "engine/experiment_family_lineage.py"
    - "engine/experiment_family_lineage_store.py"
    - "engine/strategy_admission_statistical_gate.py"
    - "engine/statistical_evidence.py"
    - "engine/multiple_testing_evidence.py"
    - "engine/overfit_evidence.py"
    - "engine/cross_strategy_reliability_gates.py"
    - "engine/strategy_admission_package.py"
lld_gate:
  required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Feature TEST-PLAN", "S01 contract", "S02 algorithm contract", "Story"]
  design_evidence_type: "full-lld"
  design_evidence_path: "process/stories/STORY-CR173-S03-golden-failure-boundary-verification-LLD.md"
  status: "approved"
implementation_gate:
  evidence_required: true
  evidence_path: "process/stories/STORY-CR173-S03-golden-failure-boundary-verification-IMPLEMENTATION.md"
  evidence_type: "implementation-md"
  implementation_objects: ["fixture", "qac-tests", "authorization-guard", "regression-selection"]
  test_plan_refs:
    - "docs/features/effective-trial-offline-estimator/TEST-PLAN.md#3-golden-vectors-v1"
    - "docs/features/effective-trial-offline-estimator/TEST-PLAN.md#7-public-boundary-与授权守卫"
  local_validation_results:
    - "S01 CP7 PASS: 60/60 + adversarial 10/10"
    - "S02 CP7 PASS: 87/87 + exact PSD oracle 854/854"
    - "S03 targeted 121/121 + public read-only 255/255 = combined 376/376 PASS"
  status: "implemented"
verification_gate:
  validation_mode: "mixed"
  validation_target:
    sut_type: "code-project"
    native_test_required: true
    workflow_eval_required: false
    prompt_bundle_required: false
    eval_suite_refs: []
    prompt_bundle_refs: []
    runtime_authorization_required: []
  workflow_eval:
    run_id: ""
    run_summary_path: ""
    suite_health_path: ""
    prompt_bundle_hash_status: "n/a"
    eval_trace_coverage_status: "n/a"
    eval_suite_health_status: "n/a"
  verification_report_path: "docs/quality/VERIFICATION-REPORT.md"
  test_report_path: "docs/quality/TEST-REPORT.md"
  review_path: "docs/quality/REVIEW.md"
  cp7_result: "not-started"
  validation_object_inventory_status: "pending"
  traceability_matrix_status: "pending"
  design_contract_verification_status: "pending"
  layered_validation_plan_status: "pending"
  remaining_risks: ["R-CR173-CONSUMER-OVERCLAIM", "R-CR173-SCOPE-ESCAPE", "R-CR173-PUBLIC-MIGRATION"]
  route_to: ""
  status: "not-started"
dev_gate:
  design_evidence_confirmed: true
  lld_confirmed: true
  dependencies_satisfied: true
  file_conflict_free: true
task_count: 4
created_at: "2026-07-16T15:20:41+08:00"
updated_at: "2026-07-17T11:52:46+08:00"
---

## 目标

用六类 declared_exact synthetic vectors、八类 fail-closed cases 和静态授权/边界守卫，证明 standalone estimator 稳定、可恢复、策略无关，并且 public C1/Gate1/DSR/admission 没有被修改、调用或改善。

## 开发上下文（dev_context）

- **输入文件**：S01/S02 contracts 与 implementation evidence、Feature TEST-PLAN、Dependency Map 的 8+12 public inventory。
- **输出文件**：`tests/fixtures/effective_trial/golden_vectors_v1.json`、CR173 QAC/auth tests。
- **设计约束**：fixture only；6/6×3/3；每组 stable computation ref/evidence hash=1/1、external attempt audit refs=3/3；8/8 failures；非 public NP-01..09 inventory=9/9且各0；public 四个 new-code zero + read-only 12/12 + expected edits=0 独立计数；跨组重复计数=0；真实/strategy/runtime=0。
- **LLD 策略**：full-lld；LLD v1.4 已完成三轮独立复核与 pointer-only refresh，当前为 `approved/confirmed=true`，规范合同变化数为 `0`；等待 S01/S02 通过依赖门控。
- **平台目标**：repository-local `uv run`，无外部网络/服务。

### 依赖与并行门控

| 上游 Story | 类型 | LLD 门控 | 开发门控 | 说明 |
|---|---|---|---|---|
| S01 | runtime | contract declared | implementation evidence passed | 只读 evidence contract/serializer |
| S02 | runtime | algorithm contract declared | implementation evidence passed | 只读 estimator |

### 文件所有权

| 类型 | 文件 | Owner / 合并规则 |
|---|---|---|
| primary | `tests/fixtures/effective_trial/` | S03 独占 |
| primary | `tests/research/test_effective_trial_cr173_qac.py` | S03 独占 |
| primary | `tests/research/test_effective_trial_authorization.py` | S03 独占 |
| read-only/forbidden | S01/S02 source 与 8 个 public production paths | 可读取、不可由 S03 修改；diff=0 |

### 文件系统布局

```text
tests/fixtures/effective_trial/golden_vectors_v1.json
tests/research/test_effective_trial_cr173_qac.py
tests/research/test_effective_trial_authorization.py
```

### 验证对象清单

| 对象 | 数量 | 通过标准 |
|---|---:|---|
| golden class | 6 | 每类 3/3 repeat；stable computation/evidence=1/1，external audits=3/3 |
| fail-closed reason | 8 | present/available=0；F03/F04 不重叠；每次 attempt 有 canonical basis/audit |
| evidence schema | 7 fields | present 7/7；failed count null |
| public production inventory | 8 | CR173 新代码依赖/调用/production diff/write=0/0/0/0 |
| public regression/authorization inventory | 12 | CP7 read-only 调用 12/12；existing expected edits=0；current unavailable/worst-state不恶化 |
| non-public deny-default operation class | 9 | NP-01..09 inventory=9/9；每类 counter=0；duplicate=0 |

#### Non-public operation-class inventory

| Class ID | Operation class | 归类边界 | Counter target |
|---|---|---|---:|
| NP-01 | credential | credential/env/account read；不复用 provider counter | 0 |
| NP-02 | real data | real data read；不含 lake/NAS/provider | 0 |
| NP-03 | lake/NAS | lake/NAS read or write | 0 |
| NP-04 | provider/network | provider/network fetch；不含 QMT/trading/Git remote | 0 |
| NP-05 | catalog/store/pointer | catalog/store/current-pointer write；不代替 public diff/write | 0 |
| NP-06 | strategy runtime | strategy runtime/external framework/simulation；不含 QMT/trading | 0 |
| NP-07 | QMT/trading | QMT/broker/trading | 0 |
| NP-08 | publish/deploy | publish/deploy | 0 |
| NP-09 | Git remote | Git remote write；不复用 provider counter | 0 |

每个 guard/probe 只能声明一个 NP-01..09 `operation_class_id`；九类不包含 public 边界。public 必须独立报告 `cr173_new_code_public_dependency_edges=0`、`cr173_new_code_public_calls=0`、`public_production_diff=0`、`public_writes=0`、`cp7_read_only_public_regression_inventory=12/12`、`existing_expected_edits=0`，跨 inventory 重复计数为 `0`。

### AI 可执行任务清单

| TASK-ID | 动作 | 目标文件 | 描述 |
|---|---|---|---|
| CR173-F01-T09 | 创建 | golden fixture | 6 类 declared_exact synthetic vectors + canonical F01-F08 attempt-basis oracle |
| CR173-F01-T10 | 创建 | QAC tests | 6×3 stable computation/evidence + 3 audits、8 failures、两类 append-only recovery |
| CR173-F01-T11 | 创建 | authorization tests | NP-01..09 inventory=9/9、各counter=0、duplicate=0；public new-code dependency/call/production diff/write四计数=0且不复用counter |
| CR173-F01-T12 | 创建 | QAC tests | CP7 read-only public regressions 12/12、existing expected edits=0、existing public calls不进入NP-01..09；standalone/claim ceiling/CR172 assertions |

## 实现执行上下文（implementation_context）

| 项目 | 内容 |
|---|---|
| 实现证据类型 | implementation-md |
| 实现证据路径 | `process/stories/STORY-CR173-S03-golden-failure-boundary-verification-IMPLEMENTATION.md` |
| 实现对象清单 | fixture / qac-tests / authorization-guard / regression-selection |
| 设计契约映射状态 | LLD v1.4 已批准；HLD/Domain/ADR v1.2 与 Feature v0.3 权威指针已刷新，normative contract delta=`0` |
| 单元测试 / Fixture 计划 | Feature TEST-PLAN 全文，重点 §3/4/7 |
| 最小实现切片 | T09→T10→T11→T12 |
| 局部验证结果 | CP5 已批准；S01/S02 CP7 均 PASS，S03 依赖门已解锁并可进入 CP6 |
| 平台差异检查 | N/A |
| 未覆盖项 | public projection、真实 activation、independent external verifier |
| QA / Review / Doc 关注点 | static guard不偷渡adapter；current unavailable不误称computable |

## 技术说明

| 项目 | 内容 |
|---|---|
| 设计证据类型 | full-lld |
| 设计依据 | HLD §7-12；ADR-004/005；Feature TEST-PLAN |
| 文件影响 | 仅新建 CR173 fixtures/tests；source/public production diff=0 |
| 接口 / 数据 / 权限变化 | 无接口/权限变化；只测试 standalone internal surface |
| 异常、失败与回退 | 任一 CR173 新代码 public zero counter 非0→BLOCKED；read-only regression 不是新代码 public call；upstream bug→NEEDS_REWORK；public incompatibility→future CR |
| 测试入口 | 6×3 QAC、authorization static、12/12 selected regressions |
| 风险与重访条件 | inventory/claim/runtime授权变化需重设计 |
| 偏离记录 | 任何 public production/expected 修改为禁止偏离 |

## 验证上下文（validation_context）

| 项目 | 内容 |
|---|---|
| validation_mode | mixed（fixture runtime + static authorization + read-only regression） |
| validation_target.sut_type | code-project |
| 验证追踪 | REQ-003..008；SC-F01/N01/B01/D01/C01/A01 |
| 验证对象清单状态 | pending（目标6+8+7+8+12+9；另有public六项指标） |
| CP7 结论 | not-started |
| 剩余风险 | second-order model bias、future projection migration（均不阻断 estimator-only） |
| 路由 | upstream失败→meta-dev；public语义冲突→meta-se/future CR；越权→BLOCKED |

## 量化验收标准（acceptance_criteria）

- [ ] golden classes `6/6`，每类 repeats `3/3`；合法每组 stable computation ref/evidence hash `1/1`、external attempt audit refs `3/3`。
- [ ] fail-closed classes `8/8` 的 present/available/PASS 均为 `0`；NaN/Inf 唯一命中 F03，F04 只覆盖有限 exact-rational matrix domain；两类 append-only recovery 均通过。
- [ ] strategy identity required/inferred、real input、raw fallback 均为 `0`。
- [ ] standalone evidence `1/1`；public projection/competing gate/overclaim `0/0/0`。
- [ ] `cr173_new_code_public_dependency_edges/cr173_new_code_public_calls/public_production_diff/public_writes=0/0/0/0`。
- [ ] `cp7_read_only_public_regression_inventory=12/12` 且 `existing_expected_edits=0`；既有 public 调用只属 read-only verification lane。
- [ ] NP-01..09（credential、real data、lake/NAS、provider/network、catalog/store/pointer、strategy runtime、QMT/trading、publish/deploy、Git remote）inventory=`9/9`、各 counter=`0`、duplicate=`0`。
- [ ] NP-01..09 与 public 双 lane 六项指标不共享 class/counter；existing public calls 只属 read-only lane，跨组重复计数=`0`。
- [ ] `public_effective_trial_count_populatable=false`、`c1_computable=false`、CR172 auto-resume/close=`0/0`。

## 失败路由与阻塞说明

当前无需要用户新增决策或范围扩张的 blocker；S01/S02 是正常 DAG 前置。S03 LLD v1.4 已由用户在 CP5 批准，`design_evidence_confirmed/lld_confirmed=true/true`。任一 NP-01..09 counter、class duplicate、CR173 新代码 public zero counter 或 production diff 非 0 时结论为 `BLOCKED`；golden/failure contract 偏差路由 `NEEDS_REWORK`；若只有修改 public contract 才能满足验收，再路由 `NEEDS_DESIGN_CLARIFICATION` 并创建独立 future projection CR，不得扩张 CR173。
