---
story_id: "CR173-S01-contract-evidence-canonicalization"
title: "Effective-trial input, evidence, and canonicalization contracts"
story_slug: "contract-evidence-canonicalization"
status: "verified"
priority: "P0"
wave: "CR173-W1-CONTRACT-EVIDENCE"
depends_on: []
dependency_contracts: []
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
    - "new typed input/evidence contract"
    - "versioned canonical serialization and method hash"
    - "fail-closed security and append-only recovery"
    - "downstream estimator contract shared with S02/S03"
  rationale: "七字段 schema、numeric token、status/reason、hash domain 和恢复语义是跨 Story 公共实现合同，局部技术说明不足以安全冻结。"
  waiver_reason: ""
  revisit_condition: "字段、reason enum、numeric grammar、hash domain、source mode 或 append-only 规则变化。"
  evidence_path: "process/stories/STORY-CR173-S01-contract-evidence-canonicalization-LLD.md"
file_ownership:
  primary:
    - "engine/effective_trial_evidence.py"
    - "tests/research/test_effective_trial_evidence_contracts.py"
  shared: []
  read_only:
    - "engine/experiment_family_lineage.py"
  merge_owner: "CR173-S01-contract-evidence-canonicalization"
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
  required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Feature DESIGN", "Feature TEST-PLAN", "Story"]
  design_evidence_type: "full-lld"
  design_evidence_path: "process/stories/STORY-CR173-S01-contract-evidence-canonicalization-LLD.md"
  status: "approved"
implementation_gate:
  evidence_required: true
  evidence_path: "process/stories/STORY-CR173-S01-contract-evidence-canonicalization-IMPLEMENTATION.md"
  evidence_type: "implementation-md"
  implementation_objects: ["code", "schema-contract", "canonical-serializer", "unit-tests"]
  test_plan_refs:
    - "docs/features/effective-trial-offline-estimator/TEST-PLAN.md#2-分层测试计划"
  local_validation_results:
    - "pytest: original 53/53 + CP6R2 adversarial 7/7 = 60/60 PASS"
    - "py_compile: PASS"
    - "git diff --check: PASS"
    - "F-CP7-S01-001: closed in CP6 rework 1"
    - "F-CP7-S01-002/003: CP7 reverify 2 PASS; CLOSED"
  status: "completed"
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
  remaining_risks: ["R-CR173-RAW-EFFECTIVE-ALIAS", "R-CR173-METHOD-NONDETERMINISM"]
  route_to: "CR173-S02-IMPLEMENTATION"
  status: "verified"
dev_gate:
  design_evidence_confirmed: true
  lld_confirmed: true
  dependencies_satisfied: true
  file_conflict_free: true
task_count: 4
created_at: "2026-07-16T15:20:41+08:00"
updated_at: "2026-07-17T11:03:04+08:00"
---

## 目标

创建 estimator-only 的 immutable 输入/方法/七字段 evidence 合同，以及不经过 binary float 的 versioned canonical serialization，为 S02 estimator 和 S03 verifier 提供唯一合同。

## 开发上下文（dev_context）

- **输入文件**：CR173 HLD/ADR/Domain Map、Feature DESIGN/TEST-PLAN。
- **输出文件**：`engine/effective_trial_evidence.py`、`tests/research/test_effective_trial_evidence_contracts.py`。
- **设计约束**：七个顶层键始终存在；非 present count=null；第七字段 `effective_trial_computation_ref` 是 stable content-addressed 结果身份；`ComputationAttemptAudit` 外置且不进入七字段/evidence hash；raw fallback=0；strategy identity=0；CR173 新代码 public 依赖/调用/生产 diff/write=0。
- **Feature 设计引用**：三件套见 frontmatter。
- **LLD 策略**：full-lld；LLD v1.3 已完成三轮独立复核与 pointer-only refresh，当前为 `approved/confirmed=true`，规范合同变化数为 `0`；CP5 已批准进入实现。
- **平台目标**：repository-local Python，通过 `uv run` 验证；无外部服务。

### 依赖与并行门控

无上游 Story。CP5 前置是 ADR-001..005 已批准、Feature reason enum/numeric grammar 与 HLD 一致。S01 的 LLD/实现合同冻结后，S02 才可进入开发。

### 文件所有权

| 类型 | 文件 | Owner / 合并规则 |
|---|---|---|
| primary | `engine/effective_trial_evidence.py` | 当前 Story 独占写入 |
| primary | `tests/research/test_effective_trial_evidence_contracts.py` | 当前 Story 独占写入 |
| read-only | `engine/experiment_family_lineage.py` | 只核对 sealed identity/public incompatibility，不修改 |
| forbidden | frontmatter 列出的 8 个 public C1 production paths | diff 必须为 0 |

### 文件系统布局

```text
engine/effective_trial_evidence.py
tests/research/test_effective_trial_evidence_contracts.py
```

### 关键合同字段

| 对象 | 字段/规则 | 必填 |
|---|---|---:|
| SealedTrialIdentity | family ref/hash、raw count、ordered trial IDs | 4/4 |
| DependencyMatrixEnvelope | schema/version、labels、matrix、input hash、lineage ref、source mode | 6/6 |
| EffectiveTrialMethodSpec | ID/version/hash + canonical spec descriptor | 4/4 |
| EffectiveTrialEvidence | 七字段；第七字段是 stable content-addressed `effective_trial_computation_ref` | 7/7 |
| ComputationAttemptAudit | 外置 append-only 审计；链接 computation ref/evidence hash，不进入七字段/hash | 1/1 |

### AI 可执行任务清单

| TASK-ID | 动作 | 目标文件 | 描述 |
|---|---|---|---|
| CR173-F01-T01 | 创建 | evidence module | typed contracts + stable computation identity + external attempt audit；无 strategy/IO 字段 |
| CR173-F01-T02 | 创建 | evidence module | state + 8 failure reason + F01-F08 canonical attempt-basis + null/ref 规则 |
| CR173-F01-T03 | 创建 | evidence module | canonical numeric token/bytes/hash；stable computation/evidence/audit 三类链接；float bridge=0 |
| CR173-F01-T04 | 创建 | contract tests | 7/7、F01-F08 basis、3 repeats identity/audit、mutation/orphan/forged/recovery/non-alias |

## 实现执行上下文（implementation_context）

| 项目 | 内容 |
|---|---|
| 实现证据类型 | implementation-md |
| 实现证据路径 | `process/stories/STORY-CR173-S01-contract-evidence-canonicalization-IMPLEMENTATION.md` |
| 实现对象清单 | code / schema-contract / canonical-serializer / unit-tests |
| 设计契约映射状态 | LLD v1.3 已批准；HLD/Domain/ADR v1.2 与 Feature v0.3 权威指针已刷新，normative contract delta=`0` |
| 单元测试 / Fixture 计划 | Feature TEST-PLAN §2/§4/§5 |
| 最小实现切片 | T01→T02→T03→T04 |
| 局部验证结果 | CP6R2：原 53/53 + 新增对抗 7/7=`60/60 PASS`；py_compile PASS；git diff --check PASS；F-002/F-003 已实现关闭，待独立复验 |
| 平台差异检查 | N/A |
| 未覆盖项 | estimator algorithm、golden fixtures、public projection |
| QA / Review / Doc 关注点 | 七字段严格性、reason 分类、canonical bytes、无 public import/write |

## 技术说明

| 项目 | 内容 |
|---|---|
| 设计证据类型 | full-lld |
| 设计依据 | HLD §4-5/9；ADR-002/003/005；Feature DESIGN §4-6 |
| 文件影响 | 仅创建两个 primary 文件；既有 production 文件修改 0 |
| 接口 / 数据 / 权限变化 | 新 standalone internal contract；无 public C1 变化、无新权限 |
| 异常、失败与回退 | 8类 reason；失败不 present；同一 attempt 的恢复补充不覆盖旧 audit，修复后重算形成新 computation/evidence 并由新 audit 链接前序 |
| 测试入口 | Feature TEST-PLAN contract unit |
| 风险与重访条件 | schema/numeric/hash/reason变化必须回设计 |
| 偏离记录 | `0`；无 float/public projection 偏离，无 design delta |

## 验证上下文（validation_context）

| 项目 | 内容 |
|---|---|
| validation_mode | mixed（unit + static） |
| validation_target.sut_type | code-project |
| 验证追踪 | REQ-001/002/003/004/005/007；SC-Q01/F01/N01/B01/C01 |
| CP7 结论 | not-started |
| 剩余风险 | raw alias、canonical drift、public contract误触达 |
| 路由 | 失败→meta-dev；合同歧义→meta-se |

## 量化验收标准（acceptance_criteria）

- [ ] 七字段 present schema `7/7`，每个缺字段 fixture 的 present 数为 `0`。
- [ ] failure reason `8/8` 有稳定 state；raw fallback、default method、orphan ref 接受数均为 `0`。
- [ ] non-canonical token（含 NaN/Inf）唯一映射 F03；F04 只覆盖成功解析为有限 exact rational 后的 shape/symmetry/diag/range/PSD 失败，交叉命中为 `0`。
- [ ] canonical numeric bytes/hash 对等价输入只有 `1` 种表示；Decimal→float 路径为 `0`。
- [ ] 同一 normalized input/method 的 stable computation ref/evidence hash 为 `1/1`，3 次执行的 external attempt audit ref 为 `3/3`，audit 不进入七字段/hash。
- [ ] strategy identity 字段/推断为 `0`；`cr173_new_code_public_dependency_edges/cr173_new_code_public_calls/public_production_diff/public_writes=0/0/0/0`。
- [ ] append-only recovery 同时覆盖同一 attempt 审计补充链与修复后新 computation/evidence 链，旧 audit/hash 均不覆盖。

## 失败路由与阻塞说明

当前无需要用户新增决策或范围扩张的 blocker；S01 LLD v1.3 已由用户在 CP5 批准，`design_evidence_confirmed/lld_confirmed=true/true`。仅 S01 可先进入 CP6；S02 继续受 S01 实现与验证依赖门控。
