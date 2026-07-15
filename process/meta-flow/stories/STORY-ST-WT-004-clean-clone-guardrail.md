---
story_id: "ST-WT-004"
title: "修复 clean-clone rule 与 cache guardrail"
story_slug: "clean-clone-guardrail"
cr_id: "CR-047"
status: "verified-with-risk"
priority: "P0"
wave: "W2"
lld_wave: "LW2"
qa_wave: "QW3"
depends_on: ["ST-WT-002"]
dependency_contracts: [{story_id: "ST-WT-002", type: "contract", lld_gate: "declared-contract", dev_gate: "upstream-contract-frozen"}]
feature_refs: ["FEAT-WT-04"]
feature_design_refs: ["process/docs/features/cr047-delivery/DESIGN.md", "process/docs/features/cr047-delivery/TEST-PLAN.md", "process/docs/features/cr047-delivery/TASKS.md"]
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["guardrail", "installer-contract", "package-security"]
  rationale: "Canonical rule and cache severity affect clean-clone/package release correctness."
  waiver_reason: ""
  revisit_condition: "Package semantics require an independent release owner."
  evidence_path: "process/stories/STORY-ST-WT-004-clean-clone-guardrail-LLD.md"
file_ownership:
  primary: ["scripts/check_delivery_guardrails.py", ".gitignore", "tests/test_cr047_delivery_gate.py"]
  shared: ["delivery/rules/AGENTS.md", "delivery/scripts/install.py"]
  merge_owner: "ST-WT-004"
  forbidden: ["ignored root AGENTS.md as canonical", "all-cache-ignore", "backup/**", "process/quant-lab/**"]
lld_gate: {required_inputs: ["accepted HLD", "accepted ADR", "CR047-FEATURE-DESIGN-MATRIX", "Feature design", "ST-WT-002 contract"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-ST-WT-004-clean-clone-guardrail-LLD.md", status: "approved"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-ST-WT-004-clean-clone-guardrail-IMPLEMENTATION.md", evidence_type: "implementation-md", status: "passed"}
verification_gate: {validation_mode: "mixed", cp7_result: "PASS_WITH_RISK", remaining_risks: [], status: "verified-with-risk"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 2
created_at: "2026-07-14T15:40:00Z"
updated_at: "2026-07-15T13:22:00Z"
---

## 目标

让 clean archive 使用 tracked canonical rule 通过 guardrail，并将 cache 按 tracked/package/ignored 输入分类。

## 开发上下文（dev_context）

| 项目 | 内容 |
|---|---|
| 输入 | REQ-WT-009..010/014、ADR-WT-004、ST-WT-002 routing contract |
| 输出 | root-rule contract、cache classifier、archive/local fixtures |
| 设计约束 | `delivery/rules/AGENTS.md` canonical；root wrapper generated；package precedence 高于 ignore |
| 平台 | clean Git archive 与本机 ignored-cache 两类输入 |

### 依赖与文件所有权

开发等待 ST-WT-002 contract frozen。ST-WT-005 后续负责共享文件最终 Ruff merge；本 Story不得全局重写 Python imports。

### AI 可执行任务清单

| TASK-ID | 动作 | 目标 | 描述 |
|---|---|---|---|
| TASK-WT-004-01 | 修改 | guardrail/rule/gitignore | canonical source 与 generated wrapper 语义 |
| TASK-WT-004-02 | 修改/创建 | cache classifier/tests | tracked/package block，ignored-only warn/cleanup |

## 技术说明

设计证据类型为 full-lld；正式证据见 `process/stories/STORY-ST-WT-004-clean-clone-guardrail-LLD.md`。

## 实现执行上下文

CP5 尚未批准；不会清理或修改用户本机文件，只设计/测试分类器。

## 验证上下文

计划运行 clean archive guardrail、三类 cache fixture 与 generated wrapper drift 检查。

## 量化验收标准

- [ ] clean archive guardrail exit=0。
- [ ] tracked/package cache 拒绝率=100%。
- [ ] ignored-only cache 不形成永久 blocker，warning/cleanup 可见。
- [ ] canonical tracked rule count=1。

## 阻塞说明

若 package input 集合无法确定，cache 保持 blocker。
