---
story_id: "CR152-S01-pit-feature-label-contracts"
title: "PIT feature matrix and label policy contracts"
story_slug: "pit-feature-label-contracts"
lld_version: "1.0"
tier: "M"
status: "ready-for-review"
confirmed: false
created_by: "host-orchestrator"
created_at: "2026-07-02T10:37:00+08:00"
confirmed_by: ""
confirmed_at: ""
feature_design_refs:
  - "docs/features/factor-research-loop/DESIGN.md"
  - "docs/features/factor-research-loop/TEST-PLAN.md"
  - "docs/features/factor-research-loop/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "PIT feature matrix contract"
    - "label policy and leakage guard"
    - "future triple_barrier / meta_label extension slots"
  rationale: "CR152 first wave introduces reusable ML feature/label metadata contracts and leakage enforcement."
open_items: 0
---

# LLD: CR152-S01 — PIT Feature Matrix and Label Policy Contracts

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| HLD | `process/docs/design/HLD-ML-STRATEGY-E2E-FRAMEWORK.md` | PIT matrix、label policy、leakage guard、triple_barrier slot boundary |
| ADR | `process/docs/design/ARCHITECTURE-DECISION-ML-STRATEGY-E2E-FRAMEWORK.md` | metadata-only boundary、extend existing contracts |
| Feature Matrix | `process/docs/design/FEATURE-DESIGN-MATRIX.md` | S01 full-lld、FEAT-03 ownership、first-wave / later-wave boundary |
| CP5 Context | `process/context/CP5-CR152-ML-STRATEGY-E2E-CONTEXT.yaml` | active `triple_barrier` must be `BLOCKED` in first wave |
| Existing source anchor | `engine/research_production_contracts.py` | `ResearchDatasetSpec` and `LeakagePolicy` extension anchors |

## 1. Goal

Define metadata-only PIT feature matrix and label policy contracts for ML strategy E2E without replacing `ResearchDatasetSpec` or implementing later-wave labeling algorithms.

## 2. Requirements（Functional / Non-Functional）

### 2.1 Functional

- Add or compose local dataclasses / validators for `PITFeatureMatrixSpec`, `MLLabelPolicySpec` and `MLLabelMethodSupport`.
- Preserve `ResearchDatasetSpec` as the dataset anchor and `LeakagePolicy` as the leakage policy anchor.
- Validate feature availability before decision time and label availability after decision time.
- Reserve `fixed_window`, `triple_barrier` and `meta_label` label method slots.
- Enforce active `triple_barrier` or `meta_label` as `BLOCKED` in CR152 first wave because algorithms are not implemented.

### 2.2 Non-Functional

- Metadata-only: no file IO, no provider, no lake/NAS, no model training.
- Deterministic: validation returns stable issue codes and JSON-safe dictionaries.
- Backward compatible: existing `ResearchDatasetSpec` behavior remains unchanged unless callers opt into ML companion validators.

## 3. 模块拆分与职责

| 模块 / 文件组 | 职责 | 说明 |
|---|---|---|
| `engine/research_production_contracts.py` | Add ML companion contracts and validators | Keep near `ResearchDatasetSpec` / `LeakagePolicy` to avoid parallel contracts. |
| `tests/research/test_ml_strategy_e2e_contracts.py` | Unit fixtures for PIT, label methods and leakage failures | Shared by S01/S02/S03/S04; S01 owns feature/label cases. |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
|---|---|---|
| 修改 | `engine/research_production_contracts.py` | Add `PITFeatureMatrixSpec`, `MLLabelPolicySpec`, `MLLabelMethodSupport`, validators and `to_dict` helpers. |
| 新增 | `tests/research/test_ml_strategy_e2e_contracts.py` | Add S01 contract serialization, leakage and label method support tests. |

## 5. 数据模型与持久化设计

No persistence is introduced. All objects are metadata contracts.

| 对象 / 字段 | 类型 | 约束 | 说明 |
|---|---|---|---|
| `PITFeatureMatrixSpec` | dataclass | `matrix_id`, `research_dataset_spec_id`, `entity_key_fields`, `decision_time_field`, `feature_available_at_field`, `feature_schema_hash`, `lineage_refs` required | Companion to `ResearchDatasetSpec`. |
| `MLLabelPolicySpec` | dataclass | `policy_id`, `label_method`, `label_horizon_days`, `decision_time_field`, `label_available_at_field`, `method_spec` | Companion to `LeakagePolicy`. |
| `MLLabelMethodSupport` | dataclass / mapping | `method`, `supported_in_first_wave`, `unsupported_reason` | Makes reserved-but-unimplemented methods explicit. |
| `MLContractIssue` | dataclass / mapping | `code`, `severity`, `message`, `field`, `evidence_ref` | Reuse simple issue style from CR151 where practical. |

### Label Method Enforcement

| `label_method` | First-wave status | Enforcement |
|---|---|---|
| `fixed_window` | supported | Validate required horizon and availability fields. |
| `triple_barrier` | reserved only | Return `BLOCKED` issue `ml_label_method_not_implemented` when active. |
| `meta_label` | reserved only | Return `BLOCKED` issue `ml_label_method_not_implemented` when active. |

This resolves the HLD §9 ambiguity: active `triple_barrier` is not `NEEDS_REVIEW` in CR152 first wave.

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
|---|---|---|---|---|
| `validate_pit_feature_matrix_spec(spec)` | dataclass or mapping | tuple issues | S02/S03/S04, tests | Validates required fields, primary keys and immutable-looking refs. |
| `validate_ml_label_policy_spec(spec)` | dataclass or mapping | tuple issues | S01/S02/S04, tests | Includes `triple_barrier` / `meta_label` BLOCKED enforcement. |
| `to_dict()` | instance | JSON-safe dict | evidence/tests | No filesystem writes. |

## 7. 核心处理流程

1. Caller builds `ResearchDatasetSpec` as usual.
2. Caller builds `PITFeatureMatrixSpec` referencing the research dataset spec id.
3. Caller builds `MLLabelPolicySpec`.
4. Validators run PIT required-field checks and label method support checks.
5. S02 consumes fields for purged/embargo fixture requirements.
6. S04 consumes issues to determine ML gate status.

## 8. 技术设计细节

- Follow existing `dataclass(frozen=True, slots=True)` style where practical.
- Keep issue payload JSON-safe; avoid custom exception flow for expected invalid metadata.
- Do not add real data readers. Any fixture data rows in tests remain inline/local test data.
- Do not modify existing `ResearchDatasetSpec` constructor requirements unless unavoidable; prefer companion objects.

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
|---|---|---|
| Safety | No credential, lake, NAS, provider, registry, runtime or broker access. | Static unit tests and forbidden counter checks in downstream stories. |
| Performance | Linear validation over metadata fields only. | No benchmark required. |
| Compatibility | Companion objects avoid changing existing call sites. | Existing CR147/CR151 targeted tests remain green. |

## 10. 测试设计

| 测试场景 | 前置条件 | 操作 | 预期结果 | 验证方式 |
|---|---|---|---|---|
| PIT spec serializes | valid metadata | call `to_dict()` | stable JSON-safe dict | pytest |
| Missing decision time field | missing field | validate PIT spec | `BLOCKED` issue | pytest |
| Feature available after decision | fixture row metadata | validate leakage condition | leakage `BLOCKED` issue | pytest |
| Fixed-window label accepted | method fixed_window | validate label policy | no method support issue | pytest |
| Active triple_barrier blocked | method triple_barrier | validate label policy | `ml_label_method_not_implemented` with severity blocker | pytest |
| Active meta_label blocked | method meta_label | validate label policy | `ml_label_method_not_implemented` with severity blocker | pytest |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
|---|---|---|---|---|
| CR152-S01-T01 | 修改 | `engine/research_production_contracts.py` | Add PIT feature matrix dataclass and mapping normalization. | PIT serialization |
| CR152-S01-T02 | 修改 | `engine/research_production_contracts.py` | Add label policy dataclass and method support table. | label method tests |
| CR152-S01-T03 | 修改 | `engine/research_production_contracts.py` | Add validators and issue helpers. | leakage / missing-field tests |
| CR152-S01-T04 | 新增 | `tests/research/test_ml_strategy_e2e_contracts.py` | Add S01 unit coverage. | full S01 test slice |

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 推荐方案 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
|---|---|---|---|---|---|---|
| LCQ-CR152-S01-001 | HLD §9 allowed `NEEDS_REVIEW or BLOCKED` for active `triple_barrier`. | Use `BLOCKED` in first wave. | Resolved by user CP3 approval note and this LLD. | Label validator / tests / release wording | CP5 context `CP5-FOCUS-CR152-001` | If future CR implements triple-barrier algorithm. |

| 风险 / 难点 | 影响 | 缓解措施 |
|---|---|---|
| Companion object drifts from `ResearchDatasetSpec` | Duplicated contract semantics | Store anchor ids/fingerprints and keep validators small. |
| Reserved methods are misread as implemented | False capability claim | `MLLabelMethodSupport` and tests return `BLOCKED` when active. |

### OPEN / Spike 跟踪

| ID | 类型 | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| N/A | OPEN | No blocking open item. | N/A | N/A |

## 13. 回滚与发布策略

- Rollback: remove companion objects and tests if CP6 implementation violates anchors.
- Compatibility: no migration needed; no persistent data.
- Release wording: first wave supports fixed-window only and reserves later-wave methods.

## 14. Definition of Done

- [ ] PIT feature matrix contract exists and serializes deterministically.
- [ ] Label policy contract includes three method slots.
- [ ] Active `triple_barrier` and `meta_label` return `BLOCKED`.
- [ ] Leakage guard tests cover positive and negative cases.
- [ ] No real IO, training, runtime, registry or external framework operation is introduced.

## 人工确认区

| # | 检查项 | 状态 | 证据 |
|---|---|---|---|
| 1 | LLD 覆盖 AC | pending | 第 2 / 10 / 14 节 |
| 2 | triple_barrier enforcement 明确 | pending | 第 5 / 12 节 |
| 3 | 文件影响范围明确 | pending | 第 4 / 11 节 |
| 4 | 测试与 dev_gate 可计算 | pending | 第 10 / 14 节 |

**人工审查结果回填**：

- 结论：`pending`
- 审查人：
- 审查时间：
- 修改意见：
- 风险接受项：
