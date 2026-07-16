---
story_id: "ST-GB-004"
title: "以独立授权快进两仓默认分支并守住2/2投影门"
story_slug: "paired-default-fast-forward"
status: "verified-with-risk"
priority: "P0"
wave: "W3"
depends_on: ["ST-GB-002"]
dependency_contracts: [{story_id: "ST-GB-002", type: "contract", lld_gate: "declared-contract", dev_gate: "upstream-merged"}]
feature_design_refs: ["process/docs/features/cr050-merge/DESIGN.md", "process/docs/features/cr050-merge/TEST-PLAN.md", "process/docs/features/cr050-merge/TASKS.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["default-write", "authorization", "cross-repo-partial", "workflow-projection"], rationale: "默认分支写和PARTIAL投影属于发布级风险。", evidence_path: "process/stories/STORY-ST-GB-004-paired-default-fast-forward-LLD.md"}
file_ownership: {primary: ["meta_flow/workflow/git_branch_lifecycle.py", "tests/test_git_branch_lifecycle.py"], shared: ["meta_flow/workspace/git_sync.py", "meta_flow/workflow/cr_lifecycle.py", "meta_flow/state/current.py", "meta_flow/cli.py"], merge_owner: "ST-GB-004", forbidden: ["local git merge", "force/force-with-lease", "auto rollback", "single-repo PASS projection", "real default write during CP6"]}
lld_gate: {required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Story"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-ST-GB-004-paired-default-fast-forward-LLD.md", status: "approved"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-ST-GB-004-paired-default-fast-forward-IMPLEMENTATION.md", evidence_type: "implementation-md", implementation_objects: ["code", "guardrail-test", "template-schema"], test_plan_refs: ["process/docs/features/cr050-merge/TEST-PLAN.md"], local_validation_results: ["3 merge/projection fixtures passed", "36 legacy regression passed", "CLI/Ruff/diff PASS"], status: "completed"}
verification_gate: {validation_mode: "mixed", validation_target: {sut_type: "meta-flow-core-code", native_test_required: true, workflow_eval_required: false}, remaining_risks: ["real-protected-default-write-unverified", "forge-receipt-unavailable", "no-independent-qa"], cp7_result: "process/checks/CP7-CR050-ST-GB-004-VERIFICATION-DONE-R2.result.json", status: "pass-with-risk"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 4
created_at: "2026-07-16T15:20:00Z"
updated_at: "2026-07-16T16:46:00Z"
---

## 目标

显式、独立授权地按artifact→project把2/2 exact published tip普通快进到remote default；单仓成功只能PARTIAL，paired projection/finish/close必须保持false。

## 开发上下文（dev_context）

消费CP3 R3 2/2 projection invariant与FEAT-GB-04 pack；共享lifecycle/git/CR/current/CLI文件由本Story作为merge owner串行修改。禁止使用本地merge、任何force或真实default写。

| TASK-ID | 动作 | 描述 |
|---|---|---|
| TASK-GB-004-01 | 创建 | authz/eligibility/outcome/projection types |
| TASK-GB-004-02 | 修改 | exact OID ordinary ref push与post-check |
| TASK-GB-004-03 | 创建 | artifact-first executor/resume |
| TASK-GB-004-04 | 修改/创建 | 2/2 projection gate与PARTIAL不推进fixtures |

## 技术说明

正式证据为full LLD。命令语义冻结为`git push --porcelain <remote> <published_oid>:refs/heads/<default>`，不含`+`/force；2/2聚合前不得写overall PASS。

## 实现执行上下文（implementation_context）

CP5前not-started；若实现无法隔离attempt writer和workflow writer，必须NEEDS_DESIGN_CLARIFICATION并切project-first重开CP2/CP3。

## 验证上下文（validation_context）

计划bare remote、remote rejection、command spy、PARTIAL projection fixture；真实保护平台与独立QA风险保留。

## 量化验收标准（acceptance_criteria）

- success时2/2 defaults=published tips，顺序artifact→project。
- non-FF/authz失败prewrite mutation=0；禁止命令=0。
- artifact PASS/project fail时projection/finish/close=false、2/2 branches retained、rollback=0。
- TC-GB-012..017覆盖6/6。

## 阻塞说明

无当前问题；projection invariant不可实现即设计阻断，不可waive。
