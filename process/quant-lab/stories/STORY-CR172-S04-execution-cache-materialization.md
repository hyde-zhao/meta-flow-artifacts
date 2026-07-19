---
story_id: "CR172-S04-execution-cache-materialization"
title: "Execution-local immutable cache verification and atomic materialization"
story_slug: "execution-cache-materialization"
status: "verified"
priority: "P0"
wave: "CR172-W4-EXECUTION-MATERIALIZATION"
depends_on: ["CR172-S01-action-authorization-eligibility-governance", "CR172-S02-trial-return-artifact-native-hook", "CR172-S03-nas-replica-verification"]
dependency_contracts:
  - {upstream: "CR172-S01-action-authorization-eligibility-governance", type: "contract", lld_gate: "upstream_contract_declared", dev_gate: "upstream_contract_frozen_and_merged"}
  - {upstream: "CR172-S02-trial-return-artifact-native-hook", type: "verifier-library", lld_gate: "unique_verifier_contract_declared", dev_gate: "upstream_contract_frozen_and_merged"}
  - {upstream: "CR172-S03-nas-replica-verification", type: "runtime", lld_gate: "replica_receipt_contract_declared", dev_gate: "upstream_implementation_evidence_passed"}
feature_design_refs:
  - "docs/features/trial-return-artifact-pipeline/DESIGN.md"
  - "docs/features/trial-return-artifact-pipeline/TEST-PLAN.md"
  - "docs/features/trial-return-artifact-pipeline/TASKS.md"
  - "docs/features/research-artifact-replica-materialization/DESIGN.md"
  - "docs/features/research-artifact-replica-materialization/TEST-PLAN.md"
  - "docs/features/research-artifact-replica-materialization/TASKS.md"
  - "docs/features/path-i-authorization-claim-governance/DESIGN.md"
  - "docs/features/path-i-authorization-claim-governance/TEST-PLAN.md"
  - "docs/features/path-i-authorization-claim-governance/TASKS.md"
feature_refs: ["trial-return.artifact.pipeline", "research-artifact.replica-materialization", "path-i.authorization.claim.governance"]
feature_reference_ids: ["FEAT-CR172-I01", "FEAT-CR172-I02", "FEAT-CR172-I03"]
lld_policy: "full-lld"
lld_policy_detail:
  required_level: "full-lld"
  trigger_reasons: ["execution-machine boundary", "pull verification", "atomic immutable cache", "direct-NAS deny"]
  rationale: "执行机数据面与 runtime 依赖必须在实现前证明只消费本地已验证 cache。"
  waiver_reason: ""
  revisit_condition: "materialization receipt、4/4 verify、cache selection、execution mapping 或 runtime consumer 变化。"
  evidence_path: "process/stories/STORY-CR172-S04-execution-cache-materialization-LLD.md"
file_ownership:
  primary: ["engine/research_artifact_materialization.py", "tests/research/test_cr172_execution_materialization.py"]
  shared: []
  read_only: ["engine/path_i_governance.py", "engine/research_artifact_replica.py", "engine/trial_return_artifact.py"]
  merge_owner: "CR172-S04-execution-cache-materialization"
  forbidden: ["engine/mature_multifactor_research.py", "engine/path_i_governance.py", "engine/trial_return_artifact.py", "engine/research_artifact_replica.py", "engine/experiment_family_lineage.py", "engine/effective_trial_evidence.py"]
lld_gate: {required_inputs: ["HLD v1.4", "ADR v1.4", "FEATURE-DESIGN-MATRIX v1.30", "Feature I01 v1.2/I02 v1.2/I03 v1.2", "S01/S02/S03 contracts", "Story"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR172-S04-execution-cache-materialization-LLD.md", status: "confirmed"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-CR172-S04-execution-cache-materialization-IMPLEMENTATION.md", evidence_type: "implementation-md", implementation_objects: ["contract-code", "materializer", "recovery-tests"], test_plan_refs: ["docs/features/research-artifact-replica-materialization/TEST-PLAN.md"], local_validation_results: ["S04 19/19 PASS", "S01-S04 127/127 PASS", "py_compile PASS", "whitespace diagnostics 0/2", "CP6R1 Story Return PASS", "CP6R1 Result PASS", "CP7R1 independent verification PASS"], rework_attempt: 1, finding_closure_status: "closed_by_cp7r1", status: "completed"}
verification_gate:
  validation_mode: "mixed"
  validation_target: {sut_type: "code-project", native_test_required: true, workflow_eval_required: false, prompt_bundle_required: false, eval_suite_refs: [], prompt_bundle_refs: [], runtime_authorization_required: []}
  verification_report_path: "docs/quality/VERIFICATION-REPORT.md"
  test_report_path: "docs/quality/TEST-REPORT.md"
  review_path: "docs/quality/REVIEW.md"
  cp7_result: "PASS"
  validation_object_inventory_status: "complete"
  traceability_matrix_status: "complete"
  design_contract_verification_status: "passed"
  layered_validation_plan_status: "complete"
  remaining_risks: ["R-CR172-NAS-CANONICALITY-CONFLICT", "R-CR172-SYNC-REPLICA-STALE"]
  route_to: "host-orchestrator-unlock-s05"
  status: "passed"
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 4
created_at: "2026-07-18T10:30:00+08:00"
updated_at: "2026-07-18T23:49:02+08:00"
---

## 目标

实现 execution local staging→`4/4` verify→atomic immutable cache→receipt 合同，证明 future runtime 只消费本地 selected cache；本 Story 不实现 runtime 或真实 pull。

## 开发上下文（dev_context）

输入必须由 S03 selected-replica staging port 返回 sealed bundle+selection，并包含 expected release、S01 decision；S04 复用 S02 唯一 verifier 做 bytes-level 4/4 复验。不得绕过 S03 selection，也不得信任 receipt 替代 bytes或创建第二 digest。

### 依赖与文件所有权

S01 contract + S02 verifier-library + S03 selected-replica runtime dependencies；S04 独占两 primary 文件；不得修改上游或 runner。

### AI 可执行任务

T01 LLD；T02 materializer/receipt；T03 4/4/mismatch/interruption tests；T04 previous-cache/no-runtime-edge guard。

## 技术说明

R3 full-lld 必须冻结 staging port sealed bundle+selection、S02 verifier-library call、`4/4` bytes compare、atomic cache 与 direct-NAS deny；tampered seal bytes 时 seal=false/pointer=0。

## 量化验收标准（acceptance_criteria）

- [x] expected release/manifest/seal/content verify=`4/4`。
- [x] direct-NAS/direct-research runtime accepted=`0/2`。
- [x] mismatch/interruption/revoke cache pointer advance=`0`。
- [x] previous cache preservation=`100%`；S04 seal/research/NAS pointer mutation=`0/0/0`。
- [x] real pull/execution/runtime operation=`0`。
- [x] S04 test IDs unique=`16/16`，duplicate=`0`。
- [x] bypass-S03-selection/receipt-only-seal-trust/secondary-digest=`0/0/0`；S02 verifier call=`1` per staging candidate。
- [x] selection 引用的 materialization receipt 在 CAS 前独立不可变持久化且 exact-readable=`100%`。
- [x] resolver 对 old/forged/uncommitted selection accepted=`0/3`；current exact accepted=`1`。
- [x] password/passwd/secret/token/credential 在 preflight/commit persist 前 accepted=`0/20`，canonical marker count=`0`。

## CP6 实现结果

- 实现：`engine/research_artifact_materialization.py`，只提供 repository-local pure contracts 与 fixture port；S03 `read_selected_replica` 是唯一供数入口，S02 `verify_sealed_trial_return_bundle` 是唯一 bytes verifier。
- 测试：`tests/research/test_cr172_execution_materialization.py` 覆盖 LLD 16 个唯一 ID；S04=`16/16 PASS`，S01-S04 组合回归=`124/124 PASS`，failed/skipped=`0/0`。
- 原子与不可变：preflight/commit 双 guard、content-addressed immutable cache、每次 commit 最多一次 CAS；interruption/revoke/selection drift/immutable conflict/CAS conflict 均不推进本次 pointer。
- runtime 边界：只可解析 `source_kind=execution_local_immutable_cache` 的 selected local handle；receipt 不替代 bundle bytes，不存在 NAS/research/staging runtime handle。
- 证据：`process/stories/STORY-CR172-S04-execution-cache-materialization-IMPLEMENTATION.md`、`process/returns/CR172-S04-execution-cache-materialization.CP6.return.json`、`process/evidence/CR172-S04-execution-cache-materialization.CP6.index.json`、`process/checks/CP6-CR172-S04-IMPLEMENTATION-DONE.result.json`。
- 设计变化：`design_delta_required=false`；长期 Feature/HLD/ADR 变化=`0`。真实 NAS/lake/runtime/network/credential/signal/trading/deploy/Git remote 操作=`0`。

## CP6R1 最小回修结果

- F-001 `closed_pending_cp7r1`：receipt 以 self-hash 独立持久化；相同 bytes + rotated receipt 仅在 CAS 前 exact-read 成功后推进。receipt persist、immutable 或 CAS 失败均返回 `selection/handle=None` 并保留 previous current。
- F-002 `closed_pending_cp7r1`：public resolver 改为消费 `MaterializationStoragePortV1` current-selection capability；old、forged revision/hash、uncommitted selection 均 fail closed，current exact selection 才返回 local-only handle。
- F-003 `closed_pending_cp7r1`：preflight 与 commit evidence refs 统一拒绝 password/passwd/secret/token/credential，且校验发生在 persist/CAS 前；合法 opaque refs 回归继续通过。
- 验证：原 S04 16 项 + 3 组 CP6R1 负向测试=`19/19 PASS`；S01-S04=`127/127 PASS`；py_compile PASS；两项 whitespace diagnostics=`0/2`。
- 证据：`process/returns/CR172-S04-execution-cache-materialization.CP6R1.return.json`、`process/evidence/CR172-S04-execution-cache-materialization.CP6R1.index.json`、`process/checks/CP6-CR172-S04-REWORK-1.result.json`。
- Design Delta：`design_delta_required=false`；回修只收紧既有 receipt correlation、current-selection capability 与 credential marker 合同，未修改长期设计。

## 失败路由

需要 runtime direct-NAS/shared-drive→BLOCKED/设计澄清；implementation defect→NEEDS_REWORK；真实 pull/materialize→BLOCKED。
