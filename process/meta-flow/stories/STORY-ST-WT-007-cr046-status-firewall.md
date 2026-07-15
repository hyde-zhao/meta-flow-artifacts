---
story_id: "ST-WT-007"
title: "以 protected-object firewall 收敛 CR-046 current status"
story_slug: "cr046-status-firewall"
cr_id: "CR-047"
status: "verified-with-risk"
priority: "P0"
wave: "W1"
lld_wave: "LW1"
qa_wave: "QW2"
depends_on: ["ST-WT-001"]
dependency_contracts: [{story_id: "ST-WT-001", type: "contract", lld_gate: "declared-contract", dev_gate: "upstream-contract-frozen"}]
feature_refs: ["FEAT-WT-01", "FEAT-WT-05"]
feature_design_refs: ["process/docs/features/cr047-operator-status/DESIGN.md", "process/docs/features/cr047-operator-status/TEST-PLAN.md", "process/docs/features/cr047-operator-status/TASKS.md", "process/docs/features/cr047-truth/DESIGN.md"]
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["protected-history", "object-identity", "evidence-ceiling"]
  rationale: "The Story projects current truth while proving that closed CR-046 originals remain byte-identical."
  waiver_reason: ""
  revisit_condition: "Any protected original mutation requires a child CR/HLD."
  evidence_path: "process/stories/STORY-ST-WT-007-cr046-status-firewall-LLD.md"
file_ownership:
  primary: ["process/evidence/CR047-ST-WT-007.protected-originals.json", "process/docs/product/TEST-MATRIX.md", "tests/test_cr047_operator_status.py"]
  shared: ["process/docs/product/REQUIREMENTS.md", "process/docs/product/STORY-MAP.md", "process/docs/release/**", "meta_flow/checks/cr_tracking.py"]
  merge_owner: "ST-WT-007"
  forbidden: ["CR-046 protected originals", "CR-046 ledger event prefix mutation", "path-prefix-only identity", "backup/**", "process/quant-lab/**", "runtime/receipt claims"]
lld_gate: {required_inputs: ["accepted HLD", "accepted ADR", "CR047-FEATURE-DESIGN-MATRIX", "Feature designs", "ST-WT-001 truth contract"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-ST-WT-007-cr046-status-firewall-LLD.md", status: "approved"}
implementation_gate: {evidence_required: true, evidence_path: "process/stories/STORY-ST-WT-007-cr046-status-firewall-IMPLEMENTATION.md", evidence_type: "implementation-md", status: "passed"}
verification_gate: {validation_mode: "mixed", cp7_result: "PASS_WITH_RISK", remaining_risks: ["independent QA unavailable", "platform receipt unavailable", "token telemetry unavailable", "real pilot unauthorized", "working-tree-only"], status: "verified-with-risk"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 3
created_at: "2026-07-14T15:40:00Z"
updated_at: "2026-07-15T13:22:00Z"
---

## 目标

把 CR-046 的 current projection 收敛为 `closed / READY_WITH_RISK`，同时用对象身份制 manifest 和三时点 hash 检查证明历史原件没有被改写。

## 开发上下文（dev_context）

| 项目 | 内容 |
|---|---|
| 输入 | REQ-WT-007..008/016..017、ADR-WT-005、CP3-DQ-05/06、ST-WT-001 truth contract |
| 输出 | CP6 pre-implementation protected-object manifest、projection/correction、CP6/CP7 hash verification |
| 设计约束 | identity=`path+object_type+original_sha256+immutable`；禁止路径前缀白名单；原件变化即 blocker+child CR |
| 结论上限 | 无独立 QA 时 CP7≤PASS_WITH_RISK、CP8≤READY_WITH_RISK；与 CR-046 继承风险叠加且独立 |

### 依赖与文件所有权

manifest 是 CP6 pre-implementation 产物，CP5 不提前生成。执行前从明确对象索引/引用解析对象身份；CR-046 原件、原 ledger prefix 只读。允许新增 CR-047 evidence、append-only correction 和 current projection。

### AI 可执行任务清单

| TASK-ID | 动作 | 目标 | 描述 |
|---|---|---|---|
| TASK-WT-007-01 | 创建 | protected-object manifest | CP6 pre-implementation 枚举对象身份和原 hash |
| TASK-WT-007-02 | 修改/追加 | current projection/correction | 只写允许对象，不改 CR-046 原件 |
| TASK-WT-007-03 | 验证 | CP6 completion/CP7 | 两次重算 hash，差异为零才可继续 |

## 技术说明

设计证据类型为 full-lld；正式证据见 `process/stories/STORY-ST-WT-007-cr046-status-firewall-LLD.md`。

## 实现执行上下文

CP5 尚未批准且 manifest 必须等 CP6 pre-implementation 才创建。发现必须修改 protected original 时立即阻断，不在本 CR 内继续。

## 验证上下文

CP6 completion 首验与 CP7 再验均要求 `protected_original_hash_changes=0`、`unauthorized_cr046_path_changes=0`；同时验证 projection 为 closed/READY_WITH_RISK。

## 量化验收标准

- [ ] manifest 对象 100% 具有 path/object_type/original_sha256/immutable/allowed_operation。
- [ ] 仅按路径前缀识别的对象数=0。
- [ ] CP6/CP7 两次 protected original hash changes=0。
- [ ] CR-046 current projection=closed/READY_WITH_RISK，7/7 Story 风险状态不被美化。
- [ ] 越界时 child-CR route=100% 触发且本 Story阻断。

## 阻塞说明

任一 protected original hash 变化、对象身份无法解析、或需要改写历史原件时，状态进入 BLOCKED/NEEDS_DESIGN_CLARIFICATION，并创建独立子 CR；不得以内联修补继续。
