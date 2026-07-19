---
story_id: "CR173-S02-exact-spectral-estimator"
title: "Exact-rational spectral participation-ratio estimator"
story_slug: "exact-spectral-estimator"
status: "verified"
priority: "P0"
wave: "CR173-W2-EXACT-ESTIMATOR"
depends_on:
  - "CR173-S01-contract-evidence-canonicalization"
dependency_contracts:
  - upstream: "CR173-S01-contract-evidence-canonicalization"
    type: "contract"
    lld_gate: "upstream_contract_declared"
    dev_gate: "upstream_contract_frozen_and_merged"
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
    - "exact-rational numeric algorithm"
    - "deterministic singular PSD validation"
    - "public method semantics and output invariant"
    - "security-sensitive fail-closed classification"
  rationale: "pivot 唯一性、fraction-free LDLT、一次舍入与双范围 invariant 必须在实现前形成可评审算法证据。"
  waiver_reason: ""
  revisit_condition: "formula、pivot comparator、PSD domain、precision、rounding、source mode 或 method hash 变化。"
  evidence_path: "process/stories/STORY-CR173-S02-exact-spectral-estimator-LLD.md"
file_ownership:
  primary:
    - "engine/effective_trial_estimator.py"
    - "tests/research/test_effective_trial_estimator.py"
  shared: []
  read_only:
    - "engine/effective_trial_evidence.py"
  merge_owner: "CR173-S02-exact-spectral-estimator"
  forbidden:
    - "engine/experiment_family_lineage.py"
    - "engine/experiment_family_lineage_store.py"
    - "engine/strategy_admission_statistical_gate.py"
    - "engine/statistical_evidence.py"
    - "engine/multiple_testing_evidence.py"
    - "engine/overfit_evidence.py"
    - "engine/cross_strategy_reliability_gates.py"
    - "engine/strategy_admission_package.py"
lld_gate:
  required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Feature DESIGN", "S01 contract", "Story"]
  design_evidence_type: "full-lld"
  design_evidence_path: "process/stories/STORY-CR173-S02-exact-spectral-estimator-LLD.md"
  status: "approved"
implementation_gate:
  evidence_required: true
  evidence_path: "process/stories/STORY-CR173-S02-exact-spectral-estimator-IMPLEMENTATION.md"
  evidence_type: "implementation-md"
  implementation_objects: ["code", "numeric-validator", "estimator", "unit-tests"]
  test_plan_refs:
    - "docs/features/effective-trial-offline-estimator/TEST-PLAN.md#5-数值测试矩阵"
  local_validation_results:
    - "S01+S02 targeted pytest: 87/87 PASS"
    - "S02 py_compile: 3/3 PASS"
    - "S02 git diff --check: PASS"
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
  cp7_result: "PASS"
  validation_object_inventory_status: "complete"
  traceability_matrix_status: "complete"
  design_contract_verification_status: "PASS"
  layered_validation_plan_status: "complete"
  remaining_risks: ["R-CR173-METHOD-NONDETERMINISM", "R-CR173-SECOND-ORDER-MODEL-BIAS"]
  route_to: "CR173-S03-CP6-IMPLEMENTATION"
  status: "verified"
dev_gate:
  design_evidence_confirmed: true
  lld_confirmed: true
  dependencies_satisfied: true
  file_conflict_free: true
task_count: 4
created_at: "2026-07-16T15:20:41+08:00"
updated_at: "2026-07-17T11:29:58+08:00"
---

## 目标

实现只消费 S01 validated contracts 的 exact-rational matrix validator 与 `spectral_participation_ratio` estimator，保证算法确定、输出 non-alias 且严格落在 `[1,n]`。

## 开发上下文（dev_context）

- **输入文件**：S01 contract、CR173 HLD/ADR、Feature DESIGN/TEST-PLAN。
- **输出文件**：`engine/effective_trial_estimator.py`、`tests/research/test_effective_trial_estimator.py`。
- **设计约束**：不估计 matrix；不接收 empirical/real；无 tolerance、binary float、eigen solver 隐式分支；non-canonical token（含 NaN/Inf）在 parser 边界唯一落 F03，F04 只消费已解析为有限 exact rational 的 matrix；只在 evidence 边界舍入一次。
- **LLD 策略**：full-lld；LLD v1.3 已完成三轮独立复核与 pointer-only refresh，当前为 `approved/confirmed=true`，规范合同变化数为 `0`；等待 S01 通过依赖门控。
- **平台目标**：repository-local pure Python，无 I/O。

### 依赖与并行门控

| 上游 Story | 类型 | LLD 门控 | 开发门控 | 说明 |
|---|---|---|---|---|
| S01 | contract | upstream contract declared | S01 contract frozen and merged | estimator 只读 S01 types/serializer，不修改 S01 primary files |

### 文件所有权

| 类型 | 文件 | Owner / 合并规则 |
|---|---|---|
| primary | `engine/effective_trial_estimator.py` | S02 独占 |
| primary | `tests/research/test_effective_trial_estimator.py` | S02 独占 |
| read-only | `engine/effective_trial_evidence.py` | S01 owner；接口变更必须回 S01/设计，不由 S02 写 |
| forbidden | frontmatter 8 public paths | diff=0 |

### 文件系统布局

```text
engine/effective_trial_estimator.py
tests/research/test_effective_trial_estimator.py
```

### 算法前置表

| 项 | 冻结值 | 失败行为 |
|---|---|---|
| representation | sealed-trial correlation matrix / declared_exact | other→typed_unavailable |
| numeric/parser | ≤12 decimals exact base-10 rational；non-canonical token、NaN/Inf | parser 失败唯一→F03 typed_unavailable |
| matrix domain | 已成功解析为有限 exact rational 后检查 shape/symmetry/diag/range/PSD | 任一失败唯一→F04 typed_unavailable |
| PSD | deterministic lexicographic symmetric-pivot fraction-free LDLT | negative/residual coupling→F04 typed_unavailable |
| formula | `n²/ΣRij²` | method mismatch→blocked |
| output | exact check→half-even≤12→second check | violation→blocked，不 clamp |

### AI 可执行任务清单

| TASK-ID | 动作 | 目标文件 | 描述 |
|---|---|---|---|
| CR173-F01-T05 | 创建 | estimator module | exact parser（F03）+ finite exact-rational matrix domain（F04）+ identity/method validator |
| CR173-F01-T06 | 创建 | estimator module | deterministic fraction-free exact LDLT |
| CR173-F01-T07 | 创建 | estimator module | exact formula + double invariant + once-only rounding |
| CR173-F01-T08 | 创建 | estimator tests | analytic/PSD/F03-F04 non-overlap/invalid/rounding/stable computation hash unit tests |

## 实现执行上下文（implementation_context）

| 项目 | 内容 |
|---|---|
| 实现证据类型 | implementation-md |
| 实现证据路径 | `process/stories/STORY-CR173-S02-exact-spectral-estimator-IMPLEMENTATION.md` |
| 实现对象清单 | code / numeric-validator / estimator / unit-tests |
| 设计契约映射状态 | LLD v1.3 已批准；HLD/Domain/ADR v1.2 与 Feature v0.3 权威指针已刷新，normative contract delta=`0` |
| 单元测试 / Fixture 计划 | Feature TEST-PLAN §3/§5 |
| 最小实现切片 | T05→T06→T07→T08 |
| 局部验证结果 | S01 CP7 已通过；S01+S02 targeted pytest `87/87 PASS`，py_compile `3/3 PASS`，diff-check PASS |
| 平台差异检查 | N/A |
| 未覆盖项 | evidence public projection、真实矩阵估计 |
| QA / Review / Doc 关注点 | pivot唯一性、zero pivot、no tolerance/no float、双 invariant |

## 技术说明

| 项目 | 内容 |
|---|---|
| 设计证据类型 | full-lld |
| 设计依据 | HLD §3-6；ADR-001..003；Feature DESIGN §6 |
| 文件影响 | 创建两个 S02 primary；S01/public路径只读 |
| 接口 / 数据 / 权限变化 | 新 internal pure estimator；权限不变 |
| 异常、失败与回退 | invalid domain unavailable；mismatch/invariant blocked；回退为 unavailable |
| 测试入口 | Feature TEST-PLAN numeric unit |
| 风险与重访条件 | algorithm唯一性或二阶claim变化需回设计/Spike |
| 偏离记录 | tolerance、float、clamp、默认method均为禁止偏离 |

## 验证上下文（validation_context）

| 项目 | 内容 |
|---|---|
| validation_mode | mixed（numeric unit + static no-I/O） |
| validation_target.sut_type | code-project |
| 验证追踪 | REQ-001/002/003/005/006；SC-P01/Q01/F01/B01/D01 |
| CP7 结论 | PASS：87/87 + exact PSD oracle 854/854，findings/waivers=0/0 |
| 剩余风险 | second-order model bias（显式接受，不由算法测试消除） |
| 路由 | algorithm bug→meta-dev；estimand/pivot contract歧义→meta-se |

## 量化验收标准（acceptance_criteria）

- [x] identity、positive-correlation、fully-correlated、singleton 四类合法 analytic oracle `4/4` 精确匹配。
- [x] positive definite、singular PSD、negative pivot、zero-pivot residual coupling `4/4` 分支确定；另有 exact oracle `854/854`、mismatch=`0`。
- [x] 输入/计算 binary float、tolerance、clamp、Decimal→float 路径均为 `0`。
- [x] non-canonical token（含 NaN/Inf）F03 命中 `1/1` 且 F04 命中 `0`；成功解析为有限 exact rational 后的 shape/symmetry/diag/range/PSD 失败只命中 F04。
- [x] 未舍入/舍入后 `[1,n]` invariant `2/2`，half-even 只执行 `1` 次。
- [x] 相同 normalized input/method 的 stable computation ref/evidence hash 数为 `1/1`；`cr173_new_code_public_dependency_edges/cr173_new_code_public_calls/public_production_diff/public_writes=0/0/0/0`。

## 失败路由与阻塞说明

当前无需要用户新增决策或范围扩张的 blocker；S01 依赖已通过 CP7，S02 CP6 已实现并进入独立验证。唯一 pivot comparator、exact singular-PSD、F03/F04 边界、双 invariant 与稳定 hash 已在定向测试中通过；仍不得用浮点库替代。
