---
story_id: "CR152-S04-ml-admission-gate-adapter"
title: "ML admission gate adapter"
story_slug: "ml-admission-gate-adapter"
lld_version: "1.0"
tier: "M"
status: "ready-for-review"
confirmed: false
created_by: "host-orchestrator"
created_at: "2026-07-02T10:37:00+08:00"
feature_design_refs:
  - "docs/features/factor-research-loop/DESIGN.md"
  - "docs/features/factor-research-loop/TEST-PLAN.md"
  - "docs/features/runtime-authorization-safety/DESIGN.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "ML-specific admission gate"
    - "CR151 four-state status adapter"
    - "StrategyAdmissionPackage linkage"
    - "safety boundary"
  rationale: "S04 crosses ML evidence, shared admission semantics and no-runtime safety claims."
open_items: 0
inline_fallback:
  status: "recorded-for-cp5-design-evidence"
  reason: "User approved continuing CP5 evidence prep; no blocking architecture ambiguity remains after CP3. If implementation finds adapter/field mapping ambiguity, return to design or dispatch meta-se."
---

# LLD: CR152-S04 — ML Admission Gate Adapter

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| HLD | `process/docs/design/HLD-ML-STRATEGY-E2E-FRAMEWORK.md` | ML-specific gate + shared four-state adapter |
| ADR | `process/docs/design/ARCHITECTURE-DECISION-ML-STRATEGY-E2E-FRAMEWORK.md` | ADR-CR152-001, ADR-CR152-002 |
| CR151 source | `engine/strategy_admission_statistical_gate.py` | `PASS / FAIL / NEEDS_REVIEW / BLOCKED` semantics |
| Admission package | `engine/strategy_admission_package.py` | Offline package linkage and no-runtime claim blockers |
| CP5 Context | `process/context/CP5-CR152-ML-STRATEGY-E2E-CONTEXT.yaml` | meta-se dispatch or inline-fallback requirement |

## 1. Goal

Define ML-specific gate summary and adapter to shared CR151 four-state semantics and offline admission package linkage without implying runtime readiness.

## 2. Requirements（Functional / Non-Functional）

### Functional

- Define `MLAdmissionGateStatus` values aligned with `PASS / FAIL / NEEDS_REVIEW / BLOCKED`.
- Define `MLAdmissionGateSummary` containing evidence refs, blocked reasons, needs-review reasons, operation counters and package linkage fields.
- Map S01/S02/S03 issues to aggregate status.
- Add or compose admission package summary fields: `gate_present`, `gate_required`, `gate_status`, `gate_ref`, `blocked_reasons`.

### Non-Functional

- No real model performance claim.
- No runtime, simulation, broker, trading or QMT readiness claim.
- No registry/store/catalog write authorization.
- Deterministic status aggregation.

## 3. 模块拆分与职责

| 模块 / 文件组 | 职责 | 说明 |
|---|---|---|
| `engine/ml_strategy_admission_gate.py` | Candidate ML gate companion module | CP6 may place objects elsewhere only if CP5-approved semantics remain. |
| `engine/strategy_admission_package.py` | Optional offline package linkage fields/helper | Must not change runtime authorization semantics. |
| `tests/research/test_ml_strategy_e2e_contracts.py` | Gate status, adapter and package linkage tests | Includes no-runtime/no-registry cases. |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
|---|---|---|
| 新增 | `engine/ml_strategy_admission_gate.py` | ML gate summary, status adapter, aggregate evaluator. |
| 修改 | `engine/strategy_admission_package.py` | Add optional ML gate summary linkage if CP6 source inspection confirms minimal safe extension. |
| 修改 | `tests/research/test_ml_strategy_e2e_contracts.py` | Add status mapping, missing evidence and forbidden counter tests. |

## 5. 数据模型与持久化设计

No persistence is introduced.

| 对象 / 字段 | 类型 | 约束 | 说明 |
|---|---|---|---|
| `MLAdmissionGateStatus` | str enum or constants | `PASS`, `FAIL`, `NEEDS_REVIEW`, `BLOCKED` | Same semantic values as CR151. |
| `MLAdmissionGateIssue` | dataclass | code, severity, message, field, evidence_ref | May reuse shared issue style. |
| `MLAdmissionGateSummary` | dataclass | status, evidence_refs, blocked_reasons, needs_review_reasons, operation_counts, gate_ref | Aggregate output. |
| `MLAdmissionPackageLinkage` | mapping/dataclass | gate_present, gate_required, gate_status, gate_ref, blocked_reasons | Offline package linkage only. |

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
|---|---|---|---|---|
| `evaluate_ml_admission_gate(...)` | S01/S02/S03 metadata/audit objects, counters | `MLAdmissionGateSummary` | tests, future package linkage | Pure metadata evaluation. |
| `adapt_ml_gate_to_admission_package(summary)` | gate summary | mapping | package builder / tests | Adds offline linkage fields. |
| `validate_ml_gate_operation_counters(counters)` | mapping | tuple issues | evaluator | Nonzero forbidden counters -> `BLOCKED`. |

## 7. 核心处理流程

1. Collect S01/S02/S03 validation issues and evidence refs.
2. Normalize forbidden operation counters.
3. If mandatory evidence missing or any forbidden counter nonzero, status is `BLOCKED`.
4. Else if complete evidence contains semantic split/threshold failure, status is `FAIL`.
5. Else if evidence is complete but flagged as review-only, status is `NEEDS_REVIEW`.
6. Else status is `PASS`.
7. Adapter emits admission package linkage fields without changing runtime authorization flags.

## 8. 技术设计细节

- Missing evidence is always `BLOCKED`, not `NEEDS_REVIEW`.
- Active `triple_barrier` method issue from S01 is `BLOCKED`, so aggregate status is `BLOCKED`.
- Keep ML gate schema separate from CR151 report schema; reuse semantics, not report fields.
- Use inline-fallback for CP5 design evidence only: no additional meta-se dispatch is needed now because CP3 resolved architecture choices and this LLD fixes the known ambiguity. If CP6 implementation discovers field mapping ambiguity, stop and reopen design or dispatch meta-se.
- Before modifying `engine/strategy_admission_package.py`, CP6 must inspect the current `StrategyAdmissionPackage` source and prove that `gate_present`, `gate_required`, `gate_status`, `gate_ref` and `blocked_reasons` can be added or composed without changing runtime authorization semantics. If the safe extension is not obvious, do not edit the file; reopen design or dispatch meta-se.

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
|---|---|---|
| Runtime safety | Admission package linkage remains offline evidence only. | Tests verify no runtime-ready claims. |
| Registry safety | S03 forbidden counters flow into gate. | Nonzero registry counter -> `BLOCKED`. |
| Determinism | Fixed aggregation order and dedupe by issue code/field. | Snapshot-like dict assertions. |

## 10. 测试设计

| 测试场景 | 前置条件 | 操作 | 预期结果 | 验证方式 |
|---|---|---|---|---|
| All metadata present | S01/S02/S03 passing inputs | evaluate gate | `PASS` | pytest |
| Missing PIT evidence | S01 missing required fields | evaluate gate | `BLOCKED` | pytest |
| Active triple_barrier | S01 label method active | evaluate gate | `BLOCKED` | pytest |
| Split overlap failure | S02 complete violation | evaluate gate | `FAIL` | pytest |
| Registry counter nonzero | S3 counter nonzero | evaluate gate | `BLOCKED` | pytest |
| Package linkage | gate summary | adapt to package mapping | fields include gate_present/required/status/ref | pytest |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
|---|---|---|---|---|
| CR152-S04-T01 | 新增 | `engine/ml_strategy_admission_gate.py` | Add status, issue and summary contracts. | serialization/status tests |
| CR152-S04-T02 | 新增 | `engine/ml_strategy_admission_gate.py` | Add aggregate evaluator and counter validator. | PASS/BLOCKED/FAIL tests |
| CR152-S04-T03 | 条件修改 | `engine/strategy_admission_package.py` | First inspect source. Add optional offline ML gate linkage helper/fields only if the extension is minimal and does not alter runtime authorization semantics; otherwise stop and reopen design or dispatch meta-se. | package linkage tests / fallback evidence |
| CR152-S04-T04 | 修改 | `tests/research/test_ml_strategy_e2e_contracts.py` | Add adapter and no-runtime claim tests. | S04 test slice |

## 12. 风险、难点与预研建议

| 风险 / 难点 | 影响 | 缓解措施 |
|---|---|---|
| Adapter field mapping ambiguity | Could require HLD-level choice | CP6 must stop and reopen design or dispatch meta-se if ambiguity appears. |
| PASS misread as runtime readiness | Unsafe release claim | Keep package runtime flags and blocked claims unchanged. |
| StrategyAdmissionPackage cannot be safely extended | Could force hidden architecture change in CP6 | Do not edit `engine/strategy_admission_package.py`; return to CP5 / meta-se with source inspection evidence. |

### OPEN / Spike 跟踪

| ID | 类型 | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| N/A | OPEN | No blocking open item. | N/A | N/A |

## 13. 回滚与发布策略

Rollback removes the ML gate companion and package linkage helper. Since no runtime/registry/store operations are performed, rollback is source/test only.

## 14. Definition of Done

- [ ] ML gate summary and evaluator exist.
- [ ] Four-state status semantics match CR151.
- [ ] Active `triple_barrier` aggregates to `BLOCKED`.
- [ ] Package linkage fields are present when adapter is used.
- [ ] Runtime/registry/store/catalog operations remain forbidden.

## 人工确认区

| # | 检查项 | 状态 | 证据 |
|---|---|---|---|
| 1 | LLD 覆盖 AC | pending | 第 2 / 10 / 14 节 |
| 2 | Adapter / package linkage 明确 | pending | 第 5 / 6 / 7 节 |
| 3 | Inline-fallback / meta-se 条件明确 | pending | 第 8 / 12 节 |

**人工审查结果回填**：

- 结论：`pending`
- 审查人：
- 审查时间：
- 修改意见：
- 风险接受项：
