---
story_id: "CR172-S05-path-i-integration-claim-zero-operation-verification"
title: "PATH-I integrated failure, claim-ceiling, and zero-operation verification"
story_slug: "path-i-integration-claim-zero-operation-verification"
status: "verified"
priority: "P0"
wave: "CR172-W5-INTEGRATED-VERIFICATION"
depends_on: ["CR172-S01-action-authorization-eligibility-governance", "CR172-S02-trial-return-artifact-native-hook", "CR172-S03-nas-replica-verification", "CR172-S04-execution-cache-materialization"]
dependency_contracts:
  - {upstream: "CR172-S01-action-authorization-eligibility-governance", type: "runtime", lld_gate: "contract_declared", dev_gate: "implementation_evidence_passed"}
  - {upstream: "CR172-S02-trial-return-artifact-native-hook", type: "runtime", lld_gate: "artifact_contract_declared", dev_gate: "implementation_evidence_passed"}
  - {upstream: "CR172-S03-nas-replica-verification", type: "runtime", lld_gate: "replica_contract_declared", dev_gate: "implementation_evidence_passed"}
  - {upstream: "CR172-S04-execution-cache-materialization", type: "runtime", lld_gate: "materialization_contract_declared", dev_gate: "implementation_evidence_passed"}
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
  trigger_reasons: ["cross-feature verification", "27 P0 scenarios", "authorization/claim/security guard", "failure recovery and compatibility"]
  rationale: "最终证据必须独立覆盖所有 Story 和 forbidden boundaries，不能由实现 Story 自证替代。"
  waiver_reason: ""
  revisit_condition: "scenario/requirement、public boundary、authorization action、claim、legacy path 或 deferred signal/FU scope 变化。"
  evidence_path: "process/stories/STORY-CR172-S05-path-i-integration-claim-zero-operation-verification-LLD.md"
file_ownership:
  primary: ["tests/fixtures/cr172_path_i/", "tests/research/test_cr172_path_i_integration.py", "tests/research/test_cr172_path_i_authorization.py", "tests/research/test_cr172_path_i_claim_regression.py"]
  shared: []
  read_only: ["engine/path_i_governance.py", "engine/trial_return_artifact.py", "engine/research_artifact_replica.py", "engine/research_artifact_materialization.py", "engine/mature_multifactor_research.py", "engine/experiment_family_lineage.py", "engine/effective_trial_evidence.py"]
  merge_owner: "CR172-S05-path-i-integration-claim-zero-operation-verification"
  forbidden: ["engine/path_i_governance.py", "engine/trial_return_artifact.py", "engine/research_artifact_replica.py", "engine/research_artifact_materialization.py", "engine/mature_multifactor_research.py", "engine/experiment_family_lineage.py", "engine/effective_trial_evidence.py", "engine/effective_trial_estimator.py", "engine/cross_strategy_reliability_gates.py", "engine/strategy_admission_package.py"]
lld_gate: {required_inputs: ["HLD v1.4", "ADR v1.4", "FEATURE-DESIGN-MATRIX v1.30", "I01/I02/I03 TEST-PLAN v1.2", "S01-S04 contracts", "Story"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR172-S05-path-i-integration-claim-zero-operation-verification-LLD.md", status: "confirmed"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-CR172-S05-path-i-integration-claim-zero-operation-verification-IMPLEMENTATION.md", evidence_type: "implementation-md", implementation_objects: ["fixture", "integration-tests", "authorization-guard", "claim-regression"], test_plan_refs: ["docs/features/trial-return-artifact-pipeline/TEST-PLAN.md", "docs/features/research-artifact-replica-materialization/TEST-PLAN.md", "docs/features/path-i-authorization-claim-governance/TEST-PLAN.md"], local_validation_results: ["S05 CP6R1 scoped: 27 passed/0 failed/0 skipped", "S01-S05 CP6R1 combined: 154 passed/0 failed/0 skipped", "py_compile: PASS", "whitespace diagnostics: 0", "semantic mismatch: 0"], rework_attempt: 1, finding_closure_status: "closed_f001", status: "passed"}
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
  remaining_risks: ["R-CR172-TRIAL-RETURN-SOURCE-ABSENT", "R-CR172-RUNTIME-AUTHORIZATION-GAP", "R-CR172-SYNC-REPLICA-STALE", "R-CR172-REAL-R-DOMAIN-MISMATCH"]
  route_to: "host-orchestrator-cp8-preparation"
  status: "passed"
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 4
created_at: "2026-07-18T10:30:00+08:00"
updated_at: "2026-07-19T01:31:46+08:00"
---

## 目标

以独立 test-only Story 证明 CR172 PATH-I 的 15/15 requirements、27/27 P0 scenarios、三段 artifact 链、六动作/DAG、legacy/signal/FU/claim 边界全部闭环，同时六类真实动作保持 authorized/executed=`0/6`,`0/6`。

## 开发上下文（dev_context）

- 输入：S01-S04 contracts/implementation evidence、3 套 TEST-PLAN、SC/TEST-MATRIX。
- 输出：deterministic fixtures + integration/authorization/claim QAC tests。
- 约束：production files read-only；发现缺陷回原 Story；Signal detailed/FU-v2/real runtime success paths=`0`。

### 依赖与文件所有权

4 个 runtime dependencies；S05 独占 4 个 test/fixture paths，禁止所有 production source write。

### AI 可执行任务

T01 LLD；T02 fixture；T03 three QAC suites；T04 scoped coverage/DAG/source-boundary/zero-op evidence。

## 技术说明

full-lld 必须列出 27 scenario→test case 映射、failure oracle、source caller inventory、six action counters、five claim flags、legacy/signal deferred static inventory。测试不能以真实 NAS/lake/runtime 为前置。

## 量化验收标准（acceptance_criteria）

- [x] requirements/scenarios/outcomes coverage=`15/15`,`27/27`,`11/11`，uncovered=`0/0/0`。
- [x] current runner/lineage diff=`0/0`；fixture binding/seal public contract=`2/2`,`1/1`；failure/test-merge-rollback=`5/5`,`3/3`。
- [x] forward-label accepted、partial-lineage canonical advance、S03 secondary digest=`0/0/0`。
- [x] approved-ledger authorized/eligible=`0/0`；`evidence_kind` field/helper/assertion=`0`，唯一 provenance 为 decision origin + target kind + fixture URI/port。
- [x] REQ-013 contract ready=`1`；runtime path enforcement/default switch/runtime-delivered claim=`0/0/0`。
- [x] DAG cycle/invalid ref/file conflict=`0/0/0`。
- [x] six actions authorized/executed=`0/6`,`0/6`；runtime-without-read negative=`1/1`。
- [x] stage3/C1/real-data/multi-trial/signal-transport flags true=`0/5`。
- [x] Signal detailed module/Story/implementation、FU-v2 Story、real runtime Story=`0/0/0`,`0`,`0`。
- [x] legacy write/move/rename/rewrite=`0/0/0/0`；public C1 production diff/call=`0/0`。

## 失败路由

上游实现缺陷→NEEDS_REWORK 到 S01-S04；合同本身矛盾→NEEDS_DESIGN_CLARIFICATION；任一 forbidden counter/overclaim/real action→BLOCKED。

## CP6 实现摘要

- 实现证据：`process/stories/STORY-CR172-S05-path-i-integration-claim-zero-operation-verification-IMPLEMENTATION.md`。
- fixture inventory：`README.md`、`scenario_catalog.json`、`sealed_chain_v1.json`、`failure_mutations_v1.json`、`zero_operation_oracle_v1.json` 与 test-only `path_i_fixture.py` adapter。
- S05 scoped：`27 passed / 0 failed / 0 skipped`；S01-S05 combined：`154 passed / 0 failed / 0 skipped`。
- requirement/scenario/outcome：`15/15`、`27/27`、`11/11`；unknown/duplicate/uncovered=`0/0/0`。
- 六类真实动作 authorized/executed=`0/6`,`0/6`；五项高阶 claim true=`0/5`；真实外部操作=`0`。
- production source 与 S01-S04 上游测试 SHA-256 均与各自 CP7 证据一致，diff=`0`；上游缺陷=`0`，下一路由=`CP7 verification`。

## CP6R1 最小回修摘要

- `F-001=CLOSED`：`SC-CR172-021 / test_req013_is_contract_ready_only` 只绑定 `REQ-CR172-013`，不再误绑 `REQ-CR172-006`。
- 追踪测试新增显式语义绑定断言；`semantic_mismatch=0`，且 `SC-CR172-006 / test_partial_lineage_never_advances_selection` 继续权威覆盖 `REQ-CR172-006`。
- requirement/scenario/outcome 仍为 `15/15`、`27/27`、`11/11`，unknown/duplicate/uncovered=`0/0/0`。
- S05 scoped=`27/27`、S01-S05 combined=`154/154`；failed/skipped=`0/0`，py_compile 与 whitespace 均 PASS。
- production/upstream hashes=`8/8` 不变；六类真实动作 authorized/executed=`0/6`,`0/6`，五项高阶 claim true=`0/5`，真实外部操作=`0`。
- 无设计契约变化、无 design delta；Story 已重新进入 `ready-for-verification`，下一路由为 CP7R1 独立复验。
