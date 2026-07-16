---
story_id: "ST-GB-003"
title: "重新证明合并事实后安全删除CR分支"
story_slug: "proof-gated-finish"
status: "verified-with-risk"
priority: "P0"
wave: "W4"
depends_on: ["ST-GB-004"]
dependency_contracts: [{story_id: "ST-GB-004", type: "runtime", lld_gate: "declared-contract", dev_gate: "upstream-verified"}]
feature_design_refs: ["process/docs/features/cr050-finish/DESIGN.md", "process/docs/features/cr050-finish/TEST-PLAN.md", "process/docs/features/cr050-finish/TASKS.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["remote-delete", "fresh-proof", "recovery-ref", "protected-ref"], rationale: "删除远端/本地ref必须以fresh proof和独立授权门控。", evidence_path: "process/stories/STORY-ST-GB-003-proof-gated-finish-LLD.md"}
file_ownership: {primary: ["meta_flow/workflow/git_branch_lifecycle.py", "tests/test_git_branch_lifecycle.py"], shared: ["meta_flow/workspace/git_sync.py", "meta_flow/workflow/cr_lifecycle.py", "meta_flow/cli.py"], merge_owner: "ST-GB-003", forbidden: ["force-delete", "CR closed as proof", "patch similarity proof", "automatic recovery-ref push/delete", "real remote delete during CP6"]}
lld_gate: {required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Story"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-ST-GB-003-proof-gated-finish-LLD.md", status: "approved"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-ST-GB-003-proof-gated-finish-IMPLEMENTATION.md", evidence_type: "implementation-md", implementation_objects: ["code", "guardrail-test"], test_plan_refs: ["process/docs/features/cr050-finish/TEST-PLAN.md"], local_validation_results: ["4 finish/freshness fixtures passed", "36 legacy regression passed", "CLI/Ruff/diff PASS"], status: "completed"}
verification_gate: {validation_mode: "mixed", validation_target: {sut_type: "meta-flow-core-code", native_test_required: true, workflow_eval_required: false}, remaining_risks: ["squash-rebase-unprovable", "real-remote-delete-unverified", "no-independent-qa"], cp7_result: "process/checks/CP7-CR050-ST-GB-003-VERIFICATION-DONE-R2.result.json", status: "pass-with-risk"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 3
created_at: "2026-07-16T15:20:00Z"
updated_at: "2026-07-16T16:49:10Z"
---

## 目标

只在current matching 2/2 merge PASS、独立delete authz和fresh identity/tip/ancestry/protected proof全过后，建立recovery refs并安全清理两仓remote/local CR branches。

## 开发上下文（dev_context）

消费ST-GB-004 verified projection contract和FEAT-GB-03 pack；开发必须等待上游verified。禁止force-delete、旧result/CR closed/branch absent单独作proof。

| TASK-ID | 动作 | 描述 |
|---|---|---|
| TASK-GB-003-01 | 创建 | finish gate/fresh proof |
| TASK-GB-003-02 | 创建/修改 | recovery refs与remote→local cleanup |
| TASK-GB-003-03 | 修改/创建 | CLI/CR close与drift/partial/idempotence tests |

## 技术说明

正式证据为full LLD。squash/rebase无法由Git ancestry证明时fail closed，未来trusted forge receipt必须独立CR。

## 实现执行上下文（implementation_context）

实现证据与本地 bare-remote fixture 已完成；真实 remote 删除仍未授权。

## 验证上下文（validation_context）

bare ancestry/recovery/partial/idempotence/fresh-drift fixtures 已通过；真实平台和独立QA风险保留。

## 量化验收标准（acceptance_criteria）

- 2/2 proof前remote/local delete=0。
- recovery refs 2/2在remote delete前建立；两仓remote完成前local delete=0。
- branch -D/force/reset调用=0；dry-run mutation=0。

## 阻塞说明

无；squash/rebase只形成fail-closed风险，不允许猜测证明。
