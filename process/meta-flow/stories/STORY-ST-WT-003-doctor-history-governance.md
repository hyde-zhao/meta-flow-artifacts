---
story_id: "ST-WT-003"
title: "实现 lifecycle-aware Doctor 与历史治理"
story_slug: "doctor-history-governance"
cr_id: "CR-047"
status: "verified-with-risk"
priority: "P0"
wave: "W2"
lld_wave: "LW2"
qa_wave: "QW3"
depends_on: ["ST-WT-001"]
dependency_contracts: [{story_id: "ST-WT-001", type: "contract", lld_gate: "declared-contract", dev_gate: "upstream-contract-frozen"}]
feature_refs: ["FEAT-WT-03"]
feature_design_refs: ["process/docs/features/cr047-quality/DESIGN.md", "process/docs/features/cr047-quality/TEST-PLAN.md", "process/docs/features/cr047-quality/TASKS.md"]
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["policy-schema", "history-integrity", "cross-module"]
  rationale: "Doctor exit semantics, artifact lifecycle and append-only provenance are shared release contracts."
  waiver_reason: ""
  revisit_condition: "Classification cannot be derived deterministically."
  evidence_path: "process/stories/STORY-ST-WT-003-doctor-history-governance-LLD.md"
file_ownership:
  primary: ["meta_flow/checks/token_budget.py", "meta_flow/checks/quality_governance.py", "process/policies/QUALITY-MODEL.yaml", "tests/test_cr047_quality_governance.py"]
  shared: ["meta_flow/context_pack/builder.py", "meta_flow/context_pack/read_expansion.py", "process/state/READ-EXPANSION-LEDGER.ndjson", "process/state/RUN-LEDGER.ndjson"]
  merge_owner: "ST-WT-003"
  forbidden: ["historical result/ledger in-place rewrite", "fabricated actor/time/receipt", "backup/**", "process/quant-lab/**"]
lld_gate: {required_inputs: ["accepted HLD", "accepted ADR", "CR047-FEATURE-DESIGN-MATRIX", "Feature design", "ST-WT-001 contract"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-ST-WT-003-doctor-history-governance-LLD.md", status: "approved"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-ST-WT-003-doctor-history-governance-IMPLEMENTATION.md", evidence_type: "implementation-md", status: "passed"}
verification_gate: {validation_mode: "mixed", cp7_result: "PASS_WITH_RISK", remaining_risks: [], status: "verified-with-risk"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 4
created_at: "2026-07-14T15:40:00Z"
updated_at: "2026-07-15T13:22:00Z"
---

## 目标

在不截断历史证据的前提下，使 Doctor 对 current blocker、classified warning 和 unavailable legacy evidence 给出可重放结论。

## 开发上下文（dev_context）

| 项目 | 内容 |
|---|---|
| 输入 | REQ-WT-006..008/017、ADR-WT-003、ST-WT-001 lifecycle contract |
| 输出 | typed classification、B0 delta、Quality source/correction、Run contract、fixtures |
| 设计约束 | `B0_pre=21` 只作历史锚；B0_cp7 动态；active/default-required 永远 blocker |
| 平台 | 静态/本地 Doctor；无 runtime/receipt |

### 依赖与文件所有权

LLD 可先行，开发等待 ST-WT-001 contract frozen。历史 ledgers 只允许 append event；primary checker/policy 归本 Story。

### AI 可执行任务清单

| TASK-ID | 动作 | 目标 | 描述 |
|---|---|---|---|
| TASK-WT-003-01 | 修改 | budget/quality checker | lifecycle/read classification + typed severity |
| TASK-WT-003-02 | 修改/追加 | Quality Model/corrections | 修正 source，保留原 hash |
| TASK-WT-003-03 | 修改 | workflow/run contract | future true run、legacy-unverified |
| TASK-WT-003-04 | 创建/修改 | tests | 双基线、delta、source、forgery fixtures |

## 技术说明

设计证据类型为 full-lld；正式证据见 `process/stories/STORY-ST-WT-003-doctor-history-governance-LLD.md`。

## 实现执行上下文

CP5 尚未批准。当前 observed=22 只作为设计输入，不得预写为最终 B0_cp7。

## 验证上下文

计划运行 Token/Artifact/Quality/Workflow Doctor、fixture 和 read/run ledger checks；warning count 必须披露。

## 量化验收标准

- [ ] `blocking_active=0`、`unclassified=0`。
- [ ] 所有 observed 有 lifecycle/read class 与 remediation ref。
- [ ] B0_cp7→终态 delta 解释率=100%。
- [ ] Quality source error=0，6 个历史缺口均有 correction/unavailable 解释。
- [ ] 伪造 timestamp/receipt fixture 拒绝率=100%。

## 阻塞说明

任何 unknown/current-required overage 保持 blocker；不得 waiver 成 warning。
