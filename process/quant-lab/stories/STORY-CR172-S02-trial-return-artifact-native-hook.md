---
story_id: "CR172-S02-trial-return-artifact-native-hook"
title: "Trial-return artifact contract, verified seal, and fixture producer port"
story_slug: "trial-return-artifact-native-hook"
status: "verified"
priority: "P0"
wave: "CR172-W2-TRIAL-RETURN-HOOK"
depends_on: ["CR172-S01-action-authorization-eligibility-governance"]
dependency_contracts:
  - {upstream: "CR172-S01-action-authorization-eligibility-governance", type: "contract", lld_gate: "upstream_contract_declared", dev_gate: "upstream_contract_frozen_and_merged"}
feature_design_refs:
  - "docs/features/trial-return-artifact-pipeline/DESIGN.md"
  - "docs/features/trial-return-artifact-pipeline/TEST-PLAN.md"
  - "docs/features/trial-return-artifact-pipeline/TASKS.md"
  - "docs/features/path-i-authorization-claim-governance/DESIGN.md"
  - "docs/features/path-i-authorization-claim-governance/TEST-PLAN.md"
  - "docs/features/path-i-authorization-claim-governance/TASKS.md"
feature_refs: ["trial-return.artifact.pipeline", "path-i.authorization.claim.governance"]
feature_reference_ids: ["FEAT-CR172-I01", "FEAT-CR172-I03"]
lld_policy: "full-lld"
lld_policy_detail:
  required_level: "full-lld"
  trigger_reasons: ["new immutable data/schema", "fixture/real authorization binding", "verified seal handoff", "partial-lineage blocked audit"]
  rationale: "R1 review 证明现有 runner 不具备真实 period-return 和原子 lineage 前置；本 Story 安全收窄为 pure contract 与 fixture producer port。"
  waiver_reason: ""
  revisit_condition: "object kind/schema、fixture/real binding、seal digest、lineage audit、canonicalization 或 future producer scope 变化。"
  evidence_path: "process/stories/STORY-CR172-S02-trial-return-artifact-native-hook-LLD.md"
file_ownership:
  primary: ["engine/trial_return_artifact.py", "tests/research/test_cr172_trial_return_artifact.py"]
  shared: []
  read_only: ["engine/path_i_governance.py", "engine/effective_trial_evidence.py"]
  merge_owner: "CR172-S02-trial-return-artifact-native-hook"
  forbidden: ["engine/mature_multifactor_research.py", "tests/research/test_trial_lineage_producer_adapters.py", "engine/path_i_governance.py", "engine/experiment_family_lineage.py", "engine/experiment_family_lineage_store.py", "engine/effective_trial_evidence.py", "engine/effective_trial_estimator.py", "engine/cross_strategy_reliability_gates.py", "engine/strategy_admission_package.py"]
lld_gate:
  required_inputs: ["HLD v1.4", "ADR v1.4", "FEATURE-DESIGN-MATRIX v1.30", "Feature I01 v1.2/I03 v1.2", "S01 contract", "Story"]
  design_evidence_type: "full-lld"
  design_evidence_path: "process/stories/STORY-CR172-S02-trial-return-artifact-native-hook-LLD.md"
  status: "confirmed"
implementation_gate:
  evidence_required: true
  evidence_path: "process/stories/STORY-CR172-S02-trial-return-artifact-native-hook-IMPLEMENTATION.md"
  evidence_type: "implementation-md"
  implementation_objects: ["contract-code", "fixture-producer-port", "manifest-seal-verifier", "unit-contract-tests"]
  test_plan_refs: ["docs/features/trial-return-artifact-pipeline/TEST-PLAN.md"]
  local_validation_results:
    - "S02 CP6R2 targeted pytest: 35 passed, 0 failed, 0 skipped"
    - "S01+S02 CP6R2 regression: 81 passed, 0 failed, 0 skipped"
    - "py_compile: PASS; no-index whitespace diagnostics: 0/2 files"
    - "authorization refs exact ordered equality 5/5 negative variants and shared publisher guard: PASS"
  status: "reworked-2"
verification_gate:
  validation_mode: "mixed"
  validation_target: {sut_type: "code-project", native_test_required: true, workflow_eval_required: false, prompt_bundle_required: false, eval_suite_refs: [], prompt_bundle_refs: [], runtime_authorization_required: []}
  verification_report_path: "docs/quality/VERIFICATION-REPORT.md"
  test_report_path: "docs/quality/TEST-REPORT.md"
  review_path: "docs/quality/REVIEW.md"
  cp7_result: "PASS"
  validation_object_inventory_status: "complete"
  traceability_matrix_status: "complete"
  design_contract_verification_status: "complete"
  layered_validation_plan_status: "complete"
  remaining_risks: ["R-CR172-TRIAL-RETURN-SOURCE-ABSENT", "R-CR172-RETURN-DEFINITION-DRIFT", "R-CR172-PATH-IDENTITY-DRIFT"]
  route_to: "meta-qa-critical"
  status: "verified"
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 4
created_at: "2026-07-18T10:30:00+08:00"
updated_at: "2026-07-18T20:30:33+08:00"
---

## 目标

实现 `trial_portfolio_return_series@v1` 的 repository-local pure contract、fixture producer port、manifest/seal/verifier；现有 mature runner、lineage store 和真实 canonical 修改均为 `0`。历史 Story ID 只保留审计稳定性。

## 开发上下文（dev_context）

- 输入：fixture family/run/trial identity、显式 period observations、basis、release、fixture decision/context 与 lineage refs。
- 输出：exact 2-column payload、manifest、`SealedTrialReturnBundleV1`、`VerifiedTrialReturnBundleV1`、fixture selection。
- 不允许：现有 runner hook、`forward_label_proxy@v1` alias、lineage store 写入、真实 canonical/lake/runtime/artifact 运行。

### 依赖与并行门控

S01 contract dependency。S01 LLD 可先声明接口，S02 开发必须等待其合并。S03 需要 S02 sealed bundle runtime evidence。

### 文件所有权

primary `2/2` 均由 S02 独占；S03-S05 禁止修改 artifact module；runner/lineage store 全部 forbidden。

### AI 可执行任务清单

T01 R2 full LLD；T02 artifact/seal/verifier/fixture port；T03 schema/hash/binding/failure/runner-zero-diff tests；T04 public handoff evidence。详细 ID 见 Feature TASKS。

## 实现执行上下文（implementation_context）

| 项目 | 内容 |
|---|---|
| 最小切片 | governance binding → pure contract/fixture port → seal verifier → failure/zero-diff guard |
| 设计合同 | current runner hook/diff `0/0`；fixture binding `2/2`；seal truth `1/1/1`；failure `5/5` |
| 未覆盖 | multi-trial sweep、真实运行、NAS、R、public C1 |

## 技术说明

R2 LLD 必须冻结 fixture decision/target、timestamp UTC encoding、canonical JSON/Parquet、唯一 seal digest/verifier、partial-lineage audit 与 S02→S03 exact types。若需要现有 runner、lineage schema/store 或真实 canonical 写入，必须转独立 CR。

## 验证上下文（validation_context）

unit/static fixtures；existing runner/lineage diff=`0/0`，真实 action=0/6。

## 量化验收标准（acceptance_criteria）

- [ ] current runner hook/diff、lineage store diff=`0/0`,`0`；source/file-owner inventory=`100%/100%`。
- [ ] payload exact columns=`2/2`；`forward_label_proxy@v1` accepted=`0`。
- [ ] decision origin/target binding=`2/2`；fixture+real accepted=`0`。
- [ ] seal bytes/digest/verifier truth=`1/1/1`；S03 secondary digest=`0`。
- [ ] five failures fixture selection=`0/5`；partial lineage state=`partial_lineage_blocked_audit`，canonical advance=`0`。
- [ ] real operations=`0`；future native producer/atomic lineage are independent prerequisites。
- [ ] future native-producer prerequisite 包含 launch/workspace first-side-effect 前的 `RunPathDecisionV1` enforcement；current default switch=0。

## 失败路由

第二 hook/新 owner/schema ambiguity→NEEDS_DESIGN_CLARIFICATION；实现 defect→NEEDS_REWORK；真实路径/未授权 write/public C1→BLOCKED。

## CP6 实现摘要

- 结果：`implemented`；设计差异=`0`，开放实现问题=`0`，waiver=`0`。
- 实现：exact 2-column Parquet payload、15-field manifest、8-field seal、唯一 canonical seal bytes/digest/verifier、repository fixture producer port 与 `partial_lineage_blocked_audit` pure value。
- 安全边界：`forward_label_proxy@v1` 永久 hard deny；verified result 前 port commit=`0`；runner/lineage/真实湖/NAS/runtime 改动与操作=`0`。
- 验证：S02 `12 passed`；S01+S02 `58 passed`；py_compile PASS；whitespace diagnostics=`0/2`。
- 证据：`process/stories/STORY-CR172-S02-trial-return-artifact-native-hook-IMPLEMENTATION.md`、`process/evidence/CR172-S02-trial-return-artifact-native-hook.CP6.index.json`。

## CP6R1 回修摘要

- Finding：`F-CR172-S02-CP7-001` 已在实现层关闭，待独立 CP7R1 复验。
- Port：删除 Protocol/`hasattr` 路径；只接受 exact nominal `RepositoryFixtureTrialReturnPortV1`，结构兼容或 subclass 冒充 accepted/called=`0/0`。
- Binding：immutable capability 同时绑定 `decision_origin`、`target_kind`、scope revision/hash、release/run/family、logical URI 与 authorization/approval/evidence refs；13 个单字段 drift 全部在 port call 前拒绝。
- Atomicity：模块自有 commit 的受控失败发生在 selection mutation 前；`None` 与既有 selection 两种初始状态均保持原值，commit advance=`0`。
- 验证：S02 `29 passed`；S01+S02 `75 passed`；py_compile PASS；whitespace diagnostics=`0/2`；原 public `3/3+3/3` 与 schema/hash/seal/verifier 合同不变。
- CP6R1 证据：`process/evidence/CR172-S02-trial-return-artifact-native-hook.CP6R1.index.json`。

## CP6R2 回修摘要

- Finding：`F-CR172-S02-CP7-001` 的剩余 authorization-evidence binding 缺口已在实现层完整关闭，待独立 CP7R2 复验。
- Guard：既有 `_require_verified_matches_port_binding` 现在要求 `seal.authorization_evidence_refs == (binding.approval_ref, binding.evidence_ref)`；比较同时约束值、顺序与长度，新增 validator=`0`。
- Failure：approval/evidence 单字段不匹配、顺序交换、额外引用、缺失引用共 `5/5` 在 nominal port `call_count` 增加和 selection mutation 前拒绝，commit/selection advance=`0/0`。
- Publisher：发布器继续只通过 nominal `commit_verified` 进入同一 guard；verifier 后注入授权引用漂移时 call/commit/selection=`0/0/0`。
- 验证：S02 `35 passed`；S01+S02 `81 passed`；py_compile PASS；whitespace diagnostics=`0/2`；原 nominal binding、原子失败、public `3/3+3/3`、schema/hash/seal/verifier 合同不变。
- CP6R2 证据：`process/evidence/CR172-S02-trial-return-artifact-native-hook.CP6R2.index.json`。
