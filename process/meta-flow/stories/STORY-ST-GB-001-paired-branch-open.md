---
story_id: "ST-GB-001"
title: "从两仓远端默认分支开启同名CR分支"
story_slug: "paired-branch-open"
status: "verified-with-risk"
priority: "P0"
wave: "W1"
depends_on: []
dependency_contracts: []
feature_design_refs: ["process/docs/features/cr050-open/DESIGN.md", "process/docs/features/cr050-open/TEST-PLAN.md", "process/docs/features/cr050-open/TASKS.md"]
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["cross-repo", "shared-contract", "remote-write"]
  rationale: "同时操作两仓refs并复用CR bootstrap writer，必须冻结失败与权限边界。"
  evidence_path: "process/stories/STORY-ST-GB-001-paired-branch-open-LLD.md"
file_ownership:
  primary: ["meta_flow/workflow/git_branch_lifecycle.py", "tests/test_git_branch_lifecycle.py"]
  shared: ["meta_flow/workspace/git_sync.py", "meta_flow/workflow/cr_lifecycle.py", "meta_flow/cli.py"]
  merge_owner: "ST-GB-001"
  forbidden: ["process/quant-lab/**", "prelink backup", "force/history rewrite", "real remote operations during CP6"]
lld_gate: {required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Story"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-ST-GB-001-paired-branch-open-LLD.md", status: "approved"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-ST-GB-001-paired-branch-open-IMPLEMENTATION.md", evidence_type: "implementation-md", implementation_objects: ["code", "guardrail-test", "docs-handoff"], test_plan_refs: ["process/docs/features/cr050-open/TEST-PLAN.md"], local_validation_results: ["4 open fixtures passed", "36 legacy regression passed", "Ruff/diff PASS"], status: "completed"}
verification_gate: {validation_mode: "mixed", validation_target: {sut_type: "meta-flow-core-code", native_test_required: true, workflow_eval_required: false}, remaining_risks: ["real-remote-open-unverified", "no-independent-qa"], cp7_result: "process/checks/CP7-CR050-ST-GB-001-VERIFICATION-DONE-R2.result.json", status: "pass-with-risk"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 3
created_at: "2026-07-16T15:20:00Z"
updated_at: "2026-07-16T16:49:10Z"
---

## 目标

提供显式paired open：两仓从各自fresh remote default exact OID创建同名CR branch、建立upstream并产出逐仓attempt；任何全仓preflight失败时新增ref为0。

## 开发上下文（dev_context）

- 输入：CP3 R3 HLD/ADR、FEAT-GB-01 pack、现有`git_sync.py`/`cr_lifecycle.py`。
- 输出：typed lifecycle module、adapter/CLI最小接线、bare-remote tests。
- 设计约束：旧`cr bootstrap`默认行为不变；无auto commit/force；PARTIAL不自动删除。
- 平台目标：native Git subprocess；无forge API。

### 依赖与文件所有权

本Story为DAG root。共享文件只由本Story建立基础contract，后续Story串行扩展；禁止并行实现同一lifecycle模块。

### AI可执行任务清单

| TASK-ID | 动作 | 目标 | 描述 |
|---|---|---|---|
| TASK-GB-001-01 | 创建/修改 | lifecycle/git_sync | typed intent/plan/attempt和probe |
| TASK-GB-001-02 | 修改 | cr_lifecycle/cli | open coordinator与显式命令 |
| TASK-GB-001-03 | 创建 | tests | fresh/negative/partial/dry-run fixtures |

## 技术说明

正式证据为对应full LLD。失败路径为BLOCKED或PARTIAL；任何真实remote操作不在CP6授权内。

## 实现执行上下文（implementation_context）

implementation evidence、切片和局部验证均待CP5批准后填写。

## 验证上下文（validation_context）

计划运行bare remote和command-spy测试；真实平台open与独立QA风险保留。

## 量化验收标准（acceptance_criteria）

- 2/2成功base/upstream exact；全仓preflight失败new ref=0。
- dry-run local/remote mutation=0；隐式commit/force=0。
- TC-GB-001..003/009..010映射覆盖100%。

## 阻塞说明

无；若bootstrap复用只能靠隐式remote副作用，路由NEEDS_DESIGN_CLARIFICATION。
