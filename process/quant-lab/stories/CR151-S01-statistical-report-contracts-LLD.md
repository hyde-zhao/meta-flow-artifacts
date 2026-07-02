---
story_id: "CR151-S01-statistical-report-contracts"
title: "Statistical report contracts"
story_slug: "statistical-report-contracts"
lld_version: "1.1"
tier: "M"
status: "ready-for-review"
confirmed: false
created_by: "meta-dev"
created_at: "2026-07-01T23:46:00+08:00"
confirmed_by: ""
confirmed_at: ""
shared_fragments: []
feature_design_refs:
  - "docs/features/factor-research-loop/DESIGN.md"
  - "docs/features/factor-research-loop/TEST-PLAN.md"
  - "docs/features/factor-research-loop/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "new shared contract module"
    - "typed report objects"
    - "JSON-safe serialization"
  rationale: "CR151 introduces reusable statistical admission report contracts consumed by later gate evaluator and linkage stories."
open_items: 0
---

# LLD: CR151-S01 — Statistical Report Contracts

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| HLD | `process/docs/design/HLD-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE.md` | Wave A objects、status rules、static-only boundary |
| ADR | `process/docs/design/ARCHITECTURE-DECISION-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE.md` | dedicated metadata-only module、Wave A only、no runtime |
| Feature Matrix | `process/docs/design/FEATURE-DESIGN-MATRIX.md` | S01 full-lld、FEAT-03 ownership、Wave B deferred scope |
| Feature DESIGN | `docs/features/factor-research-loop/DESIGN.md` | StrategyAdmissionStatisticalGate contract boundary |
| Feature TEST-PLAN | `docs/features/factor-research-loop/TEST-PLAN.md` | CR151 contract serialization and static-only fixtures |
| Feature TASKS | `docs/features/factor-research-loop/TASKS.md` | FEAT-03-T01I task mapping |

## 1. Goal

Create the metadata-only report contracts for CR151 Wave A statistical admission so later stories can evaluate and link them without reading real data or invoking runtime systems.

## 2. Requirements（Functional / Non-Functional）

### 2.1 Functional

- Define JSON-safe typed structures for `MultipleTestingReport`, `RobustFactorStatisticsReport`, `WalkForwardValidationPlan`, `BacktestOverfitRiskReport` and `StrategyAdmissionStatisticalGate`.
- Provide report-level validation helpers that return structured issues instead of raising for expected missing/invalid evidence.
- Include operation counters on the aggregate gate object so downstream stories can block forbidden operations.
- Keep object names generic enough for future ML/event strategy adapters.

### 2.2 Non-Functional

- Deterministic serialization: repeated fixture construction returns stable dictionaries.
- Safety: no `.env`, lake, NAS, provider, QMT, broker, external framework or Git remote access.
- Maintainability: one dedicated module; no parallel replacement for existing `ResearchDatasetSpec` or `BacktestRunSpec`.
- Compatibility: use stdlib dataclasses / enum / typing and existing project style.

## 3. 模块拆分与职责

| 模块 / 文件组 | 职责 | 说明 |
|---|---|---|
| `engine/strategy_admission_statistical_gate.py` | New report contracts, issue structures, serialization helpers | S01 owns module skeleton and dataclass fields. |
| `tests/test_cr151_strategy_admission_statistical_gate.py` | Contract construction, serialization and validation fixtures | S01 tests focus on object shape and validation issues; S02 extends status aggregation tests. |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
|---|---|---|
| 创建 | `engine/strategy_admission_statistical_gate.py` | Define enums, dataclasses, `to_dict()` helpers and report-level validation helpers. |
| 创建 | `tests/test_cr151_strategy_admission_statistical_gate.py` | Add contract serialization fixtures and missing-field validation tests. |

## 5. 数据模型与持久化设计

No persistent storage is introduced. All objects are in-memory metadata contracts serialized to JSON-safe dictionaries for evidence.

| 对象 / 字段 | 类型 | 约束 | 说明 |
|---|---|---|---|
| `StatisticalGateStatus` | enum str | `PASS`, `FAIL`, `NEEDS_REVIEW`, `BLOCKED` | Defined in S01 for S02 to consume. |
| `StatisticalValidationIssue` | dataclass | `code`, `severity`, `message`, `field` | Report-level issue, not a runtime exception. |
| `MultipleTestingReport` | dataclass | `family_id`, `candidate_count`, `raw_p_values`, `adjusted_p_values`, `alpha`, `method`, `rejected_count` | FDR / multiple testing evidence. |
| `RobustFactorStatisticsReport` | dataclass | `metric_id`, `sample_count`, `ic_mean`, `rank_ic_mean`, `robust_t_stat`, `p_value`, `autocorrelation_lags` | Newey-West or equivalent robust stats evidence. |
| `WalkForwardValidationPlan` | dataclass | `folds`, `train_window`, `validation_window`, `oos_window`, `embargo_days`, `fold_metrics` | Walk-forward/OOS manifest. |
| `BacktestOverfitRiskReport` | dataclass | `trial_count`, `pbo`, `dsr`, `observed_sharpe`, `skew`, `kurtosis`, `sample_length` | PBO/DSR evidence. |
| `StrategyAdmissionStatisticalGate` | dataclass | `status`, `report_refs`, `blocked_reasons`, `needs_review_reasons`, `operation_counts` | Aggregate holder; S02 owns evaluator logic. |

### Wave B Evaluation Diagnostics Explicitly Deferred

`RobustFactorStatisticsReport` is intentionally limited to Wave A robust-statistical admission evidence. It does not implement the broader factor-evaluation diagnostics below. They remain follow-up / CR154 evaluation artifacts and must not be inferred as covered by Newey-West or robust t-stat fields.

| Deferred diagnostic | Wave A treatment | Follow-up trigger |
|---|---|---|
| IC decay by lag | Not a required S01 field. | CR154 or production-grade factor evaluation. |
| IC half-life | Not a required S01 field. | CR154 or production-grade factor evaluation. |
| Turnover | Not a required S01 field. | CR154, capacity / impact, or transaction-cost evaluation story. |
| Orthogonalization against known risk factors | Not a required S01 field. | CR154 or factor redundancy / exposure analysis. |
| Monotonicity | Not a required S01 field. | CR154 or full factor quality report. |
| Quantile spread | Not a required S01 field. | CR154 or full factor quality report. |

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
|---|---|---|---|---|
| `report.to_dict()` | report instance | JSON-safe dict | tests, S02 evaluator, evidence writers | Must not include non-deterministic objects. |
| `validate_multiple_testing_report(report)` | `MultipleTestingReport` | list of `StatisticalValidationIssue` | S02 evaluator | Detects missing family, inconsistent counts or invalid alpha. |
| `validate_robust_statistics_report(report)` | `RobustFactorStatisticsReport` | list of issues | S02 evaluator | Detects nonpositive sample count and missing p/t fields. |
| `validate_walk_forward_plan(plan)` | `WalkForwardValidationPlan` | list of issues | S02 evaluator | Detects no OOS folds or invalid fold metrics. |
| `validate_overfit_risk_report(report)` | `BacktestOverfitRiskReport` | list of issues | S02 evaluator | Detects missing trial count, pbo or dsr. |

## 7. 核心处理流程

1. Caller constructs report dataclasses from fixture or static metadata inputs.
2. Caller invokes `to_dict()` for evidence serialization or report validation helper for issue collection.
3. Validation helpers return structured issues for missing/invalid fields.
4. S02 consumes reports and issues to produce aggregate status.

No Mermaid diagram is required: this story touches two files and has no asynchronous or compensation path.

## 8. 技术设计细节

- Key rule: S01 only defines shapes and local report validation. It does not decide final admission status except for the enum type.
- Reuse: prefer `dataclasses.dataclass(frozen=True)` and `enum.Enum` or string constants consistent with project style.
- Serialization: convert tuples/enums to plain JSON-safe values; preserve numeric values without rounding in S01.
- Compatibility: expose stable names so S02/S03 tests do not duplicate local ad-hoc structures.
- Diagram type: not applicable.

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
|---|---|---|
| 安全 | No file IO, env read, network, provider, NAS, QMT, broker or external framework import. | Static test import plus operation-counter tests in S02. |
| 性能 | Linear validation over small fixture lists; no heavy computation. | Unit tests use tiny fixtures; no benchmark needed. |
| Determinism | Pure dataclass serialization. | Repeated `to_dict()` equality assertion. |

## 10. 测试设计

| 测试场景 | 前置条件 | 操作 | 预期结果 | 验证方式 |
|---|---|---|---|---|
| Multiple testing contract serializes | fixture p-values | build report and call `to_dict()` | JSON-safe dict includes family/candidate/alpha/method fields | `pytest` |
| Robust statistics contract serializes | fixture IC metrics | build report and call `to_dict()` | robust t and p-value preserved | `pytest` |
| Walk-forward plan detects missing OOS folds | empty folds | `validate_walk_forward_plan` | structured issue with blocking severity | `pytest` |
| Overfit report detects missing trial count | trial_count missing/zero | `validate_overfit_risk_report` | structured issue | `pytest` |
| Serialization deterministic | same report twice | call `to_dict()` twice | dicts equal | `pytest` |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
|---|---|---|---|---|
| CR151-S01-T01 | 创建 | `engine/strategy_admission_statistical_gate.py` | Add enum/status constants and `StatisticalValidationIssue`. | contract import test |
| CR151-S01-T02 | 创建 | `engine/strategy_admission_statistical_gate.py` | Add five Wave A dataclasses and JSON-safe `to_dict()` helpers. | serialization tests |
| CR151-S01-T03 | 创建 | `engine/strategy_admission_statistical_gate.py` | Add report-level validation helpers. | missing-field tests |
| CR151-S01-T04 | 创建 | `tests/test_cr151_strategy_admission_statistical_gate.py` | Add fixture tests for serialization and validation issues. | all S01 tests |

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 选项与推荐 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
|---|---|---|---|---|---|---|
| N/A | No blocking clarification. | Use CP3-approved dedicated module. | Resolved by CP3 approval. | Interface / tests | ADR-CR151-001 | If CP5 requests merged module. |
| LCQ-CR151-S01-01 | Should IC decay, half-life, turnover, orthogonalization, monotonicity and quantile spread be fields on `RobustFactorStatisticsReport`? | Recommendation: no for CR151 Wave A; defer them as evaluation diagnostics to CR154 / follow-up. Alternative: add optional fields now. | Resolved by CP3 Wave A only boundary; no user decision required unless CP5 expands scope. | Data model / tests / release wording | HLD DAI-CR151-04; Feature Matrix CR151 Wave B boundary | If user requires production-grade factor evaluation before CR152/CR153. |

| 风险 / 难点 | 影响 | 缓解措施 / 预研建议 |
|---|---|---|
| Dataclasses duplicate existing validation report semantics | Confusing API | Keep CR151 names admission-specific and consume existing reports by refs in S03. |
| Too much evaluator logic leaks into S01 | Story overlap | S01 only validates shape; S02 owns aggregate thresholds/status. |

### OPEN / Spike 跟踪

| ID | 类型（OPEN / Spike） | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| N/A | OPEN | No open item. | N/A | N/A |

## 13. 回滚与发布策略

- 发布方式：normal source/test change after CP5 approval and CP6 implementation.
- 回滚触发条件：contract tests fail, S02/S03 integration cannot consume types, or CP5 changes module boundary.
- 回滚动作：remove new module/tests or reduce to narrower report dataclasses before implementation merge.

## 14. Definition of Done

- [ ] Five Wave A report contracts exist.
- [ ] JSON-safe serialization tests pass.
- [ ] Missing/invalid field validation tests pass.
- [ ] No forbidden IO/import/runtime path is introduced.
- [ ] S02 can consume report contracts without local duplicate structures.

## 人工确认区

**CP5 — Story 设计证据可实现性门**

| # | 检查项 | 状态 | 证据 |
|---|---|---|---|
| 1 | LLD 覆盖 AC | pending | 第 2 / 10 / 14 节 |
| 2 | 与 HLD / ADR 一致 | pending | 第 0 / 8 / 12 节 |
| 3 | 文件影响范围明确 | pending | 第 4 / 11 节 |
| 4 | 接口契约完整 | pending | 第 6 节 |
| 5 | 测试与 dev_gate 可计算 | pending | 第 10 / 14 节 |
| 6 | clarification queue 已收敛 | pending | 第 12.1 节 |

**人工审查结果回填**：

- 结论：`pending`
- 审查人：
- 审查时间：
- 修改意见：
- 风险接受项：
