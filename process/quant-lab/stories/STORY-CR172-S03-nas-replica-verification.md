---
story_id: "CR172-S03-nas-replica-verification"
title: "NAS sealed-artifact replica verification and distribution pointer"
story_slug: "nas-replica-verification"
status: "verified"
priority: "P0"
wave: "CR172-W3-NAS-REPLICA"
depends_on: ["CR172-S01-action-authorization-eligibility-governance", "CR172-S02-trial-return-artifact-native-hook"]
dependency_contracts:
  - {upstream: "CR172-S01-action-authorization-eligibility-governance", type: "contract", lld_gate: "upstream_contract_declared", dev_gate: "upstream_contract_frozen_and_merged"}
  - {upstream: "CR172-S02-trial-return-artifact-native-hook", type: "contract", lld_gate: "verified_seal_contract_declared", dev_gate: "upstream_contract_frozen_and_merged"}
feature_design_refs:
  - "docs/features/research-artifact-replica-materialization/DESIGN.md"
  - "docs/features/research-artifact-replica-materialization/TEST-PLAN.md"
  - "docs/features/research-artifact-replica-materialization/TASKS.md"
  - "docs/features/path-i-authorization-claim-governance/DESIGN.md"
  - "docs/features/path-i-authorization-claim-governance/TEST-PLAN.md"
  - "docs/features/path-i-authorization-claim-governance/TASKS.md"
feature_refs: ["research-artifact.replica-materialization", "path-i.authorization.claim.governance"]
feature_reference_ids: ["FEAT-CR172-I02", "FEAT-CR172-I03"]
lld_policy: "full-lld"
lld_policy_detail:
  required_level: "full-lld"
  trigger_reasons: ["NAS boundary", "immutable replica receipt", "atomic pointer and recovery", "independent sync authorization"]
  rationale: "跨机副本、staging、freshness/hash/release 校验和 pointer rollback 是高风险持久化合同。"
  waiver_reason: ""
  revisit_condition: "receipt、verification tuple、freshness、NAS mapping、pointer 或 authorization semantics 变化。"
  evidence_path: "process/stories/STORY-CR172-S03-nas-replica-verification-LLD.md"
file_ownership:
  primary: ["engine/research_artifact_replica.py", "tests/research/test_cr172_nas_replica_verification.py"]
  shared: []
  read_only: ["engine/path_i_governance.py", "engine/trial_return_artifact.py"]
  merge_owner: "CR172-S03-nas-replica-verification"
  forbidden: ["engine/mature_multifactor_research.py", "engine/path_i_governance.py", "engine/trial_return_artifact.py", "engine/research_artifact_materialization.py", "engine/experiment_family_lineage.py", "engine/effective_trial_evidence.py"]
lld_gate: {required_inputs: ["HLD v1.4", "ADR v1.4", "FEATURE-DESIGN-MATRIX v1.30", "Feature I01 v1.2/I02 v1.2/I03 v1.2", "S01/S02 contracts", "Story"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR172-S03-nas-replica-verification-LLD.md", status: "confirmed"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-CR172-S03-nas-replica-verification-IMPLEMENTATION.md", evidence_type: "implementation-md", implementation_objects: ["contract-code", "replica-verifier", "recovery-tests"], test_plan_refs: ["docs/features/research-artifact-replica-materialization/TEST-PLAN.md"], local_validation_results: ["CP6R1 S03: 27 passed", "CP6R1 S01+S02+S03: 108 passed"], status: "rework-passed"}
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
  remaining_risks: ["R-CR172-SYNC-REPLICA-STALE", "R-CR172-NAS-CANONICALITY-CONFLICT"]
  route_to: "host-orchestrator-unlock-s04"
  status: "passed"
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 4
created_at: "2026-07-18T10:30:00+08:00"
updated_at: "2026-07-18T21:55:49+08:00"
---

## 目标

用 repository-local mapping/fixture 实现原 seal 的 NAS replica verification、receipt 和 atomic distribution pointer 合同；不连接真实 NAS，不重新 seal，不改变 research canonical。

## 开发上下文（dev_context）

输入=`SealedTrialReturnBundleV1` + `ResearchCanonicalSelectionV1` + `ActionDecisionV1`/`ActionScopeContextV1` + expected release；必须先消费 S02 `verify_sealed_trial_return_bundle` 返回的 `VerifiedTrialReturnBundleV1`。输出 receipt 的 `original_seal_sha256` 只能取 verifier 结果；secondary digest/re-seal=`0/0`。

### 依赖与文件所有权

S01 governance + S02 verifier contract dependencies；S03 独占两 primary 文件。S04 消费 selected-replica staging port/receipt，S05 只读 production module。

R3 需向 S04 暴露 selected replica staging port：返回精确 sealed bundle + selection；可提供窄 verifier facade，但不得改变 S02 verifier 真相。

### AI 可执行任务

T01 LLD；T02 verifier/receipt；T03 negative/recovery fixtures；T04 original seal/research pointer/no-real-op guard。

## 技术说明

full-lld 必须冻结 receipt canonicalization、expected release/freshness、staging 状态、atomic pointer 和 mid-operation revoke。真实 NAS path 不是测试前置。

## 量化验收标准（acceptance_criteria）

- [ ] original seal/manifest/content/release/freshness verification=`5/5`。
- [ ] partial/stale/unversioned/mismatch accepted/pointer=`0/0`。
- [ ] S03 seal creation/research pointer mutation=`0/0`。
- [ ] revoke/interruption previous verified replica preservation=`100%`。
- [ ] absolute path in identity/hash=`0`；real NAS operation=`0`。

## 失败路由

需要 NAS canonical 或重新 seal→NEEDS_DESIGN_CLARIFICATION/BLOCKED；implementation defect→NEEDS_REWORK；真实 sync/write→BLOCKED。

## 实现摘要

- 已实现 `engine/research_artifact_replica.py`：preflight authorization、source/staging 双次复用 S02 verifier、5/5 verification vector、immutable replica receipt、fresh decision recheck、CAS distribution selection 与 selection-bound read。
- 已实现 `tests/research/test_cr172_nas_replica_verification.py`：repository-owned in-memory port、21 项正向/负向/恢复/static guard 测试；S01+S02+S03 定向回归共 102 项通过。
- `original_seal_sha256` 只取 `VerifiedTrialReturnBundleV1`；S03 seal canonicalizer/reseal/verifier facade/direct data bypass=`0/0/0/0`。
- research canonical mutation、真实 NAS/network/env/credential/runtime 操作=`0`；S04/S05 启动或修改=`0`。
- 完整证据：`process/stories/STORY-CR172-S03-nas-replica-verification-IMPLEMENTATION.md`、`process/returns/CR172-S03-nas-replica-verification.CP6.return.json`、`process/evidence/CR172-S03-nas-replica-verification.CP6.index.json`。

### CP6R1 最小回修

- `F-CP7-S03-001`：`ReplicaPreflightV1` 现保留首次 `ActionDecisionV1`；publish 在 staging 前、commit 在 persist 前均要求 action/context/origin/target 一致，且 `commit.evaluated_at` 严格晚于 preflight。reused/equal-time/older/expired/revoked 五类路径的 stage/persist/CAS/selection mutation=`0/0/0/0`。
- `F-CP7-S03-002`：新增 deterministic concurrent-current CAS fixture；结果=`POINTER_CONFLICT`、attempt selection=`None`、CAS=`1`，并保留 concurrent writer current。
- CP6R1 验证：S03=`27 passed`；S01+S02+S03=`108 passed`；compile/whitespace/static zero-operation guards PASS；设计差异与真实操作=`0/0`。
- CP6R1 证据：`process/returns/CR172-S03-nas-replica-verification.CP6R1.return.json`、`process/evidence/CR172-S03-nas-replica-verification.CP6R1.index.json`、`process/checks/CP6-CR172-S03-REWORK-1.result.json`。
