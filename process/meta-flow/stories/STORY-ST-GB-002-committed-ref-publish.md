---
story_id: "ST-GB-002"
title: "只发布调用前已有提交并生成matching evidence"
story_slug: "committed-ref-publish"
status: "verified-with-risk"
priority: "P0"
wave: "W2"
depends_on: ["ST-GB-001"]
dependency_contracts: [{story_id: "ST-GB-001", type: "contract", lld_gate: "declared-contract", dev_gate: "upstream-merged"}]
feature_design_refs: ["process/docs/features/cr050-publish/DESIGN.md", "process/docs/features/cr050-publish/TEST-PLAN.md", "process/docs/features/cr050-publish/TASKS.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["remote-write", "evidence", "security"], rationale: "published tip是default merge的高风险输入。", evidence_path: "process/stories/STORY-ST-GB-002-committed-ref-publish-LLD.md"}
file_ownership: {primary: ["meta_flow/workflow/git_branch_lifecycle.py", "tests/test_git_branch_lifecycle.py"], shared: ["meta_flow/workspace/git_sync.py", "meta_flow/cli.py"], merge_owner: "ST-GB-002", forbidden: ["git add/commit/amend", "force", "credentials", "real remote operations during CP6"]}
lld_gate: {required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Story"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-ST-GB-002-committed-ref-publish-LLD.md", status: "approved"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-ST-GB-002-committed-ref-publish-IMPLEMENTATION.md", evidence_type: "implementation-md", implementation_objects: ["code", "guardrail-test"], test_plan_refs: ["process/docs/features/cr050-publish/TEST-PLAN.md"], local_validation_results: ["2 publish fixtures passed", "36 legacy regression passed", "CLI/Ruff/diff PASS"], status: "completed"}
verification_gate: {validation_mode: "mixed", validation_target: {sut_type: "meta-flow-core-code", native_test_required: true, workflow_eval_required: false}, remaining_risks: ["real-remote-publish-unverified", "no-independent-qa"], cp7_result: "process/checks/CP7-CR050-ST-GB-002-VERIFICATION-DONE-R2.result.json", status: "pass-with-risk"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 3
created_at: "2026-07-16T15:20:00Z"
updated_at: "2026-07-16T16:34:00Z"
---

## 目标

将调用开始时已经存在的2/2 local HEAD发布到remote CR branches，生成可被merge消费的matching evidence，且stage/commit/amend为0。

## 开发上下文（dev_context）

消费ST-GB-001 shared types和FEAT-GB-02 pack；修改lifecycle/git adapter/CLI并扩展同一bare fixture。依赖类型为contract+file conflict，开发必须等待ST-GB-001合并。

| TASK-ID | 动作 | 描述 |
|---|---|---|
| TASK-GB-002-01 | 创建 | publish planner和evidence schema |
| TASK-GB-002-02 | 修改 | paired executor/CLI exact branch push |
| TASK-GB-002-03 | 创建 | no-commit/non-FF/partial/resume tests |

## 技术说明

正式证据为full LLD。publish不更新default、不隐式调用merge；HEAD或remote drift均fail closed。

## 实现执行上下文（implementation_context）

CP5前not-started；Implementation必须列出command spy与evidence schema映射。

## 验证上下文（validation_context）

计划bare integration+argv spy；真实remote policy和独立QA保持风险。

## 量化验收标准（acceptance_criteria）

- success时2/2 remote CR OID=entry local HEAD。
- dirty/wrong/non-FF preflight失败时writes=0。
- add/commit/amend/force调用=0；dry-run mutation=0。

## 阻塞说明

无；如需工具代为commit必须另立CR。
