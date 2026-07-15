---
story_id: "ST-WT-005"
title: "收敛 Ruff 并建立完整回归质量门"
story_slug: "ruff-regression-gate"
cr_id: "CR-047"
status: "verified-with-risk"
priority: "P0"
wave: "W2"
lld_wave: "LW2"
qa_wave: "QW4"
depends_on: ["ST-WT-003", "ST-WT-004"]
dependency_contracts: [{story_id: "ST-WT-003", type: "file-conflict", lld_gate: "declared-contract", dev_gate: "upstream-merged"}, {story_id: "ST-WT-004", type: "file-conflict", lld_gate: "declared-contract", dev_gate: "upstream-merged"}]
feature_refs: ["FEAT-WT-04"]
feature_design_refs: ["process/docs/features/cr047-delivery/DESIGN.md", "process/docs/features/cr047-delivery/TEST-PLAN.md", "process/docs/features/cr047-delivery/TASKS.md"]
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["repo-wide-lint", "regression", "shared-preflight"]
  rationale: "Repo-wide mechanical rewrites and release gate composition can affect every Python module."
  waiver_reason: ""
  revisit_condition: "Ruff remediation exposes a public behavior or design defect."
  evidence_path: "process/stories/STORY-ST-WT-005-ruff-regression-gate-LLD.md"
file_ownership:
  primary: ["pyproject.toml", "tests/test_cr047_delivery_gate.py"]
  shared: ["meta_flow/**", "scripts/check_delivery_guardrails.py", "tests/**"]
  merge_owner: "ST-WT-005"
  forbidden: ["semantic auto-fix without review", "test baseline reduction", "backup/**", "process/quant-lab/**"]
lld_gate: {required_inputs: ["accepted HLD", "accepted ADR", "CR047-FEATURE-DESIGN-MATRIX", "Feature design", "ST-WT-003/004 merged contracts"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-ST-WT-005-ruff-regression-gate-LLD.md", status: "approved"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-ST-WT-005-ruff-regression-gate-IMPLEMENTATION.md", evidence_type: "implementation-md", status: "passed"}
verification_gate: {validation_mode: "mixed", cp7_result: "PASS_WITH_RISK", remaining_risks: [], status: "verified-with-risk"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 3
created_at: "2026-07-14T15:40:00Z"
updated_at: "2026-07-15T13:22:00Z"
---

## 目标

在不降低测试基线的前提下使 Ruff 错误数归零，并把 Ruff、测试、guardrail、Doctor 与 CR tracking 组合成确定性的发布前置检查。

## 开发上下文（dev_context）

| 项目 | 内容 |
|---|---|
| 输入 | REQ-WT-011..012/017、ADR-WT-004、ST-WT-003/004 已合并结果 |
| 输出 | Ruff clean baseline、组合 preflight、失败分类与可复现命令 |
| 设计约束 | 自动修复仅限语义安全项；B/F 项人工复核；不减少测试数量或放宽现有门禁 |
| 执行顺序 | 必须等待 ST-WT-003 与 ST-WT-004 合并，避免重写共享文件时覆盖其改动 |

### 依赖与文件所有权

本 Story 是共享 Python 与 guardrail 文件的最终 merge owner。实现前必须确认上游无未合并 diff，并保存 Ruff 错误分类前后对照。

### AI 可执行任务清单

| TASK-ID | 动作 | 目标 | 描述 |
|---|---|---|---|
| TASK-WT-005-01 | 修改 | Python/Ruff findings | 安全自动修复并人工处理剩余 B/F 项 |
| TASK-WT-005-02 | 修改 | preflight/CI contract | 组合五项发布门并保留原始退出码 |
| TASK-WT-005-03 | 创建/执行 | regression evidence | 全量测试、Ruff、guardrail、Doctor、CR tracking |

## 技术说明

设计证据类型为 full-lld；正式证据见 `process/stories/STORY-ST-WT-005-ruff-regression-gate-LLD.md`。

## 实现执行上下文

CP5 尚未批准；不得执行会改写源码的 Ruff `--fix`。实现阶段需先记录基线，再小批量修复和回归。

## 验证上下文

最终至少验证 Ruff errors=0、既有 377 tests + 70 subtests 不回退，并运行组合 preflight 的成功与失败 fixture。

## 量化验收标准

- [ ] Ruff errors=0。
- [ ] 测试通过数不低于 377 tests + 70 subtests，且新增 CR-047 tests 全通过。
- [ ] 五项发布门逐项有退出码和证据引用。
- [ ] 上游共享文件未被回退，未减少既有测试基线。

## 阻塞说明

ST-WT-003 或 ST-WT-004 未合并、Ruff 修复改变公共行为、或测试基线回退时阻断并回到设计澄清/回修。
