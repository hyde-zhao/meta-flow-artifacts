---
story_id: "ST-WT-002"
title: "建立 portable artifact routing 与唯一内部 docs canonical"
story_slug: "portable-artifact-routing"
cr_id: "CR-047"
status: "verified-with-risk"
priority: "P0"
wave: "W1"
lld_wave: "LW1"
qa_wave: "QW1"
depends_on: []
dependency_contracts: []
feature_refs: ["FEAT-WT-02"]
feature_design_refs: ["process/docs/features/cr047-routing/DESIGN.md", "process/docs/features/cr047-routing/TEST-PLAN.md", "process/docs/features/cr047-routing/TASKS.md"]
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["cross-repo-routing", "portability", "shared-contract"]
  rationale: "Symlink/metadata/docs ownership affects every process consumer and clean-clone reproducibility."
  waiver_reason: ""
  revisit_condition: "Only for a proven external contract requiring a generated root view."
  evidence_path: "process/stories/STORY-ST-WT-002-portable-artifact-routing-LLD.md"
file_ownership:
  primary: ["meta_flow/workspace/routing.py", "tests/test_cr047_artifact_routing.py", "delivery/rules/DIRECTORY-CONTRACT.md", "delivery/rules/DIRECTORY-CONTRACT.yaml"]
  shared: ["meta_flow/cli.py", "tests/test_workspace_routing.py"]
  merge_owner: "ST-WT-002"
  forbidden: ["/home/hyde/workspace/meta-flow.process-prelink-backup-20260713T100930/**", "process/quant-lab/**", "root internal docs writable copies"]
lld_gate: {required_inputs: ["accepted HLD", "accepted ADR", "CR047-FEATURE-DESIGN-MATRIX", "Feature design", "Story"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-ST-WT-002-portable-artifact-routing-LLD.md", status: "approved"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-ST-WT-002-portable-artifact-routing-IMPLEMENTATION.md", evidence_type: "implementation-md", status: "passed"}
verification_gate: {validation_mode: "mixed", cp7_result: "PASS_WITH_RISK", remaining_risks: [], status: "verified-with-risk"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 3
created_at: "2026-07-14T15:40:00Z"
updated_at: "2026-07-15T13:22:00Z"
---

## 目标

让 clean source/artifact clone 从源码根一次 link/check 后得到 relative、可审计且唯一的 process/docs 路由。

## 开发上下文（dev_context）

| 项目 | 内容 |
|---|---|
| 输入 | REQ-WT-004..005、ADR-WT-002、FEAT-WT-02 |
| 输出 | portable routing contract、duplicate docs check、clean-clone fixtures、目录说明 |
| 设计约束 | persistent metadata 只用 anchor+relative path；regular path 不自动迁移 |
| 平台 | Linux symlink fixture；跨设备路径不得固化 |

### 依赖与文件所有权

无上游 Story。命令必须从源码仓根执行；artifact 仓已有 `process/` 是正常目标目录。prelink backup 与 quant-lab 永久 forbidden。

### AI 可执行任务清单

| TASK-ID | 动作 | 目标 | 描述 |
|---|---|---|---|
| TASK-WT-002-01 | 修改 | routing/CLI | 强化 relative metadata、health 与错误模型 |
| TASK-WT-002-02 | 创建 | tests | clean clone、regular path、duplicate docs fixtures |
| TASK-WT-002-03 | 修改 | directory contracts | 同步 verified route behavior |

## 技术说明

设计证据类型为 full-lld；正式证据见 `process/stories/STORY-ST-WT-002-portable-artifact-routing-LLD.md`。

## 实现执行上下文

CP5 尚未批准；不得创建/移动实际 backup 或执行 destructive migration。

## 验证上下文

计划运行 workspace routing 测试和 clean snapshot link/check；实际本机 dirty 只作为信息。

## 量化验收标准

- [ ] clean clone `process_link_health=ok`。
- [ ] persisted device absolute path count=0。
- [ ] internal canonical writable copy count=1。
- [ ] backup 与 quant-lab touched path count=0。

## 阻塞说明

发现 regular `process` path 时只报告并停止，不自动迁移。
