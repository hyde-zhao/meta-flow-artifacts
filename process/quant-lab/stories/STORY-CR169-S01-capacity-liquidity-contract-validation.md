---
story_id: "CR169-S01-capacity-liquidity-contract-validation"
title: "C4 合同、关联头与输入校验"
story_slug: "capacity-liquidity-contract-validation"
status: "verified"
priority: "P0"
wave: "CR169-W1-CONTRACT-CORRELATION"
depends_on: []
feature_design_refs: ["docs/features/capacity-liquidity-evidence/DESIGN.md", "docs/features/capacity-liquidity-evidence/TEST-PLAN.md", "docs/features/capacity-liquidity-evidence/TASKS.md", "docs/features/c3-c4-correlation-boundary/DESIGN.md", "docs/features/c3-c4-correlation-boundary/TEST-PLAN.md"]
feature_refs: ["capacity.liquidity.evidence", "c3.c4.correlation.boundary"]
feature_reference_ids: ["FEAT-169-01", "FEAT-169-02"]
lld_policy: "full-lld"
lld_policy_detail: {required_level: "full-lld", trigger_reasons: ["public_contract", "hash_identity", "security", "cross_module"], rationale: "C4 public schema、13-field join、N01..N12 与 hash 分域是后续四 Story 公共合同。", waiver_reason: "", revisit_condition: "schema/header/hash/reason 变化", evidence_path: "process/stories/STORY-CR169-S01-capacity-liquidity-contract-validation-LLD.md"}
file_ownership: {primary: ["engine/capacity_liquidity_evidence.py", "tests/research/test_capacity_liquidity_contracts.py"], shared: [], merge_owner: "CR169-S01-capacity-liquidity-contract-validation", forbidden: ["engine/strategy_evidence.py", "engine/cross_strategy_reliability_gates.py", "engine/economic_cost_gate4_projection.py", "engine/strategy_admission_package.py"]}
lld_gate: {required_inputs: ["HLD", "ADR", "FEATURE-DESIGN-MATRIX", "Story"], design_evidence_type: "full-lld", design_evidence_path: "process/stories/STORY-CR169-S01-capacity-liquidity-contract-validation-LLD.md", status: "approved"}
dev_gate: {design_evidence_confirmed: true, lld_confirmed: true, dependencies_satisfied: true, file_conflict_free: true}
task_count: 3
created_at: "2026-07-14T19:05:00+08:00"
updated_at: "2026-07-15T09:32:00+08:00"
---

## 目标

建立 `capacity_liquidity@v1` immutable contract、13 字段 correlation header、N01..N12 ordered issues 与 component/envelope identity 分域。

## 开发上下文与边界

S01 输出 typed normalized input、attachment/header context、ordered issues 与 semantic hash primitives；S02 是唯一 producer 编排 owner。任何 issues 非空时 calculator 调用数必须为 0。禁止修改 neutral envelope、canonical Gate4、CR168 adapter 或 admission package。

## AI 可执行任务清单

| TASK-ID | 动作 | 目标文件 | 完成准则 |
|---|---|---|---|
| CR169-S01-T01 | 创建 | `engine/capacity_liquidity_evidence.py` | input/header/issue/result/domain constants。 |
| CR169-S01-T02 | 创建 | 同上 | normalize/validate/hash/self-validation；N01..N12 稳定排序。 |
| CR169-S01-T03 | 创建 | `tests/research/test_capacity_liquidity_contracts.py` | 12/12、13-field、10→1、tamper/identity tests。 |

## 量化验收标准

- [ ] type/schema=1/1；header fields=13/13；P0 reasons=12/12。
- [ ] 相同 computational body 10 次 hash distinct=1；identity 不进入 component body。
- [ ] invalid input false PASS=0；外部操作=0。
