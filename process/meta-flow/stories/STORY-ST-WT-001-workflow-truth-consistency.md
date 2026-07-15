---
story_id: "ST-WT-001"
title: "校验 State、CR 与 CURRENT workflow truth"
story_slug: "workflow-truth-consistency"
cr_id: "CR-047"
status: "verified-with-risk"
priority: "P0"
wave: "W1"
lld_wave: "LW1"
qa_wave: "QW1"
depends_on: []
dependency_contracts: []
feature_refs: ["FEAT-WT-01"]
feature_design_refs: ["process/docs/features/cr047-truth/DESIGN.md", "process/docs/features/cr047-truth/TEST-PLAN.md", "process/docs/features/cr047-truth/TASKS.md"]
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["cross-module", "state-lifecycle", "shared-contract"]
  rationale: "State/CR/CURRENT ownership and lifecycle rejection are architecture-major public governance contracts."
  waiver_reason: ""
  revisit_condition: "Only if the approved schemas cannot represent a required object."
  evidence_path: "process/stories/STORY-ST-WT-001-workflow-truth-consistency-LLD.md"
file_ownership:
  primary: ["meta_flow/checks/cr_tracking.py", "meta_flow/state/current.py", "tests/test_cr047_truth_consistency.py"]
  shared: ["meta_flow/workflow/cr_lifecycle.py", "meta_flow/cli.py", "tests/test_state_v2.py", "tests/test_cr_lifecycle.py"]
  merge_owner: "ST-WT-001"
  forbidden: ["CR-046 historical result/ledger mutation", "backup/**", "process/quant-lab/**", "credentials/runtime/publish"]
lld_gate: {required_inputs: ["accepted HLD", "accepted ADR", "CR047-FEATURE-DESIGN-MATRIX", "Feature design", "Story"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-ST-WT-001-workflow-truth-consistency-LLD.md", status: "approved"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-ST-WT-001-workflow-truth-consistency-IMPLEMENTATION.md", evidence_type: "implementation-md", status: "passed"}
verification_gate: {validation_mode: "mixed", cp7_result: "PASS_WITH_RISK", remaining_risks: [], status: "verified-with-risk"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 3
created_at: "2026-07-14T15:40:00Z"
updated_at: "2026-07-15T13:22:00Z"
---

## 目标

扩展现有状态与 CR checker，使 closed/missing lifecycle、legacy canonical index 和 CURRENT drift 都可被确定拒绝，同时保持 source-owned single-writer graph。

## 开发上下文（dev_context）

| 项目 | 内容 |
|---|---|
| 输入 | REQ-WT-001..003、ADR-WT-001、FEAT-WT-01 |
| 输出 | relation checker、JSON index migration、CR-033 candidate、回归 fixtures |
| 设计约束 | CURRENT 只投影；不得从 STATE.md/CURRENT 反写；unknown fail-closed |
| 平台 | Python 3.11 / uv；无 runtime/credential |

### 依赖与文件所有权

本 Story 无上游 Story。primary 只由本 Story 写；shared `cli/cr_lifecycle` 在实现前与后续 owner 合并。禁止修改 CR-046 历史原件、backup 和 quant-lab。

### AI 可执行任务清单

| TASK-ID | 动作 | 目标 | 描述 |
|---|---|---|---|
| TASK-WT-001-01 | 修改 | State/CR/CURRENT checker | 增加 lifecycle/relation typed findings |
| TASK-WT-001-02 | 修改/迁移 | canonical CR index | JSON-only，CR-033 candidate=1，legacy canonical=0 |
| TASK-WT-001-03 | 创建/修改 | tests | 覆盖 stale/closed/missing/current drift/status-sync |

## 技术说明

设计证据类型为 full-lld；正式证据见 `process/stories/STORY-ST-WT-001-workflow-truth-consistency-LLD.md`。接口、数据、权限、失败、测试和风险不得在实现阶段绕过该 LLD。

## 实现执行上下文

CP5 尚未批准；implementation evidence、切片与局部验证均为 pending。

## 验证上下文

计划运行目标测试、`meta-flow check cr-tracking`、State v2 enforce 和 CURRENT drift fixtures；inline fallback 下 CP7 最高 PASS_WITH_RISK。

## 量化验收标准

- [ ] closed/cancelled/superseded/missing active fixtures 拒绝率=100%。
- [ ] valid active CR-047 + JSON index + CURRENT 的 conflict count=0。
- [ ] legacy canonical YAML count=0，CR-033 candidate count=1。
- [ ] forbidden touched path count=0。

## 阻塞说明

无；若需要新总状态或修改 CR-046 原件，进入 NEEDS_DESIGN_CLARIFICATION。
