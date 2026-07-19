---
story_id: "CR172-S01-action-authorization-eligibility-governance"
title: "PATH-I action authorization, execution eligibility, and claim governance"
story_slug: "action-authorization-eligibility-governance"
status: "verified"
priority: "P0"
wave: "CR172-W1-GOVERNANCE-CONTRACT"
depends_on: []
dependency_contracts: []
feature_design_refs:
  - "docs/features/path-i-authorization-claim-governance/DESIGN.md"
  - "docs/features/path-i-authorization-claim-governance/TEST-PLAN.md"
  - "docs/features/path-i-authorization-claim-governance/TASKS.md"
feature_refs: ["path-i.authorization.claim.governance"]
feature_reference_ids: ["FEAT-CR172-I03"]
lld_policy: "full-lld"
lld_policy_detail:
  required_level: "full-lld"
  trigger_reasons: ["six-action security contract", "execution eligibility DAG", "claim and path compatibility", "SignalBatch scope containment"]
  rationale: "六动作分权、直接前置和 claim ceiling 被全部下游 Story 共享，必须先形成独立可评审合同。"
  waiver_reason: ""
  revisit_condition: "action kind、record schema、DAG edge、context equality、empirical disposition、path 或 signal slot 变化。"
  evidence_path: "process/stories/STORY-CR172-S01-action-authorization-eligibility-governance-LLD.md"
file_ownership:
  primary: ["engine/path_i_governance.py", "tests/research/test_cr172_path_i_governance.py"]
  shared: []
  read_only: ["engine/experiment_family_lineage.py", "engine/effective_trial_evidence.py"]
  merge_owner: "CR172-S01-action-authorization-eligibility-governance"
  forbidden: ["engine/mature_multifactor_research.py", "engine/experiment_family_lineage.py", "engine/effective_trial_evidence.py", "engine/effective_trial_estimator.py", "engine/cross_strategy_reliability_gates.py", "engine/strategy_admission_package.py"]
lld_gate:
  required_inputs: ["HLD v1.4", "ADR v1.4", "FEATURE-DESIGN-MATRIX v1.30", "Feature I03 v1.2/v1.1", "Story"]
  design_evidence_type: "full-lld"
  design_evidence_path: "process/stories/STORY-CR172-S01-action-authorization-eligibility-governance-LLD.md"
  status: "confirmed"
implementation_gate:
  evidence_required: true
  evidence_path: "process/stories/STORY-CR172-S01-action-authorization-eligibility-governance-IMPLEMENTATION.md"
  evidence_type: "implementation-md"
  implementation_objects: ["contract-code", "authorization-validator", "claim-guard", "unit-tests"]
  test_plan_refs: ["docs/features/path-i-authorization-claim-governance/TEST-PLAN.md"]
  local_validation_results:
    - {command: "cp6r1-targeted-pytest", result: "PASS", tests_passed: 46, tests_failed: 0, tests_skipped: 0}
    - {command: "cp6r1-py-compile", result: "PASS"}
    - {command: "cp6r1-no-index-check-production", result: "PASS", exit: 1, whitespace_diagnostics: 0}
    - {command: "cp6r1-no-index-check-test", result: "PASS", exit: 1, whitespace_diagnostics: 0}
  status: "rework-1-complete"
verification_gate:
  validation_mode: "mixed"
  validation_target: {sut_type: "code-project", native_test_required: true, workflow_eval_required: false, prompt_bundle_required: false, eval_suite_refs: [], prompt_bundle_refs: [], runtime_authorization_required: []}
  verification_report_path: "docs/quality/VERIFICATION-REPORT.md"
  test_report_path: "docs/quality/TEST-REPORT.md"
  review_path: "docs/quality/REVIEW.md"
  cp7_result: "PASS"
  validation_object_inventory_status: "complete"
  traceability_matrix_status: "complete"
  design_contract_verification_status: "pass"
  layered_validation_plan_status: "complete"
  remaining_risks: ["R-CR172-RUNTIME-AUTHORIZATION-GAP", "R-CR172-REAL-R-DOMAIN-MISMATCH", "R-CR172-SIGNAL-TRANSFER-AMBIGUITY"]
  route_to: "CR172-S02:CP6"
  status: "complete"
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 4
created_at: "2026-07-18T10:30:00+08:00"
updated_at: "2026-07-18T18:44:16+08:00"
---

## 目标

创建纯 repository-local 的六动作 deny-default 合同；current-v1 `approved_ledger` 固定 authorized/eligible 双 false，reason=`APPROVED_LEDGER_ADAPTER_UNAVAILABLE`。同时提供 contract-only `RunPathDecisionV1`，不连接真实授权 backend，不执行 action 或 runtime path switch。

## 开发上下文（dev_context）

- 输入：CR172 HLD/ADR v1.4、I03 v1.2 三件套、REQ-001～015 中的授权/claim/compatibility 条款。
- 输出：`engine/path_i_governance.py`、`tests/research/test_cr172_path_i_governance.py`。
- 设计约束：12-field record 不变；approved-ledger current-v1 accepted/eligible=0/0；`evidence_kind` truth=0；`decision_origin`/`target_kind` binding `2/2`；REQ-013=contract-ready/runtime-enforcement-deferred；record 独立 `6/6`；DAG edges `5/5`。
- 平台：pure Python + fixture；无 IO/network/env/credential。

### 依赖与并行门控

无上游 Story。S02-S04 的 LLD 可引用本 Story 卡片合同，但开发必须等待 S01 LLD 确认并合并。

### 文件所有权与布局

| 类型 | 路径 | 规则 |
|---|---|---|
| primary | `engine/path_i_governance.py` | S01 独占 |
| primary | `tests/research/test_cr172_path_i_governance.py` | S01 独占 |
| read-only | lineage/effective evidence modules | 只核对边界 |
| forbidden | runner/public C1/admission paths | production diff=0 |

### AI 可执行任务清单

| TASK | 动作 | 目标 |
|---|---|---|
| CR172-S01-T01 | 创建 | full LLD |
| CR172-S01-T02 | 创建 | governance contracts/validators |
| CR172-S01-T03 | 创建 | six-action/DAG/revoke fixtures |
| CR172-S01-T04 | 创建 | empirical/path/signal/claim tests |

## 实现执行上下文（implementation_context）

| 项目 | 内容 |
|---|---|
| 实现证据 | CP5 后 `...-IMPLEMENTATION.md` |
| 对象 | contract-code / guardrail-test |
| 最小切片 | record+decision → DAG → disposition/path/signal |
| 未覆盖 | 真实 authorization backend、FU-v2、signal exchange |

## 技术说明

设计证据类型=`full-lld`。缺少前置时返回 `eligible_to_execute=false`；完整性冲突用 BLOCKED，前置不足用 typed_unavailable。任何需要真实 backend 或新增 action kind 的变更返回设计澄清。

## 验证上下文（validation_context）

验证模式 mixed（unit/static/fixture）。必须分别验证 `authorized` 与 `eligible_to_execute`；六类真实动作 authorized/executed=`0/6`,`0/6`。

## 量化验收标准（acceptance_criteria）

- [ ] action kinds/records/enforcement points=`6/6/6`，approval union=`0`。
- [ ] decision origin/target enums=`2/2`；fixture decision + real target accepted=`0`。
- [ ] caller 自报 approved-ledger authorized/eligible=`false/false`，稳定 reason=`APPROVED_LEDGER_ADAPTER_UNAVAILABLE`；`evidence_kind`=`0`。
- [ ] RunPath contract ready=`1`；current runtime path enforcement/default switch=`0/0`。
- [ ] DAG nodes/edges=`6/5`，runtime-without-read launch/workspace/pointer=`0/0/0`。
- [ ] empirical disposition valid states=`4/4`，pre-v2 positive count/C1=`0/0`。
- [ ] Signal slots=`8/8`，extra mandatory/exchange module/Story=`0/0/0`。
- [ ] legacy mutation 与五项高阶 claim true=`0`。
- [ ] real IO/network/env/credential/action=`0`。

## 失败路由

合同歧义或 action/DAG 扩张→NEEDS_DESIGN_CLARIFICATION；实现/测试失败→NEEDS_REWORK；权限并集、真实操作或 overclaim→BLOCKED。
