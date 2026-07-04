---
story_id: "CR155-S03-backtest-oos-walkforward-validation"
title: "Historical backtest and OOS/walk-forward validation flow"
story_slug: "backtest-oos-walkforward-validation"
lld_version: "1.0"
tier: "M"
status: "ready-for-review"
confirmed: false
created_by: "meta-dev"
created_at: "2026-07-04T18:37:08+08:00"
confirmed_by: ""
confirmed_at: ""
shared_fragments: []
feature_design_refs:
  - "docs/features/daily-multifactor-baseline-strategy-artifact/DESIGN.md"
  - "docs/features/daily-multifactor-baseline-strategy-artifact/TEST-PLAN.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["historical backtest flow", "OOS walk-forward validation", "CR148 contract consumption"]
  rationale: "Validation flow is the basis for admission and rerun evidence."
open_items: 0
---

# LLD: CR155-S03 — Backtest/OOS/Walk-Forward Validation

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| HLD | `docs/design/HLD-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT.md` | Historical backtest and OOS/walk-forward requirements. |
| ADR | `docs/design/ARCHITECTURE-DECISION-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT.md` | Readonly pipeline and separate admission decisions. |
| S01 | `process/stories/CR155-S01-baseline-artifact-contract-LLD.md` | Artifact validation contract. |
| S02 | `process/stories/CR155-S02-readonly-data-provenance-adapter-LLD.md` | Readonly provenance contract. |

## 1. Goal

Define the validation flow that produces historical backtest, OOS and walk-forward evidence for the baseline artifact.

## 2. Requirements

### 2.1 Functional

- Consume S01 artifact and S02 provenance status.
- Define historical backtest metrics and refs.
- Define OOS/walk-forward split manifest, purge/embargo policy and fold metrics.
- Output validation summary for S04 and S05.

### 2.2 Non-Functional

- No optimizer/model search.
- No external framework execution.
- If provenance fallback is fixture/static, validation cannot claim real-data baseline.

## 3. 模块拆分与职责

| 模块 / 文件组 | 职责 | 说明 |
|---|---|---|
| `engine/daily_multifactor_baseline_artifact.py` | Candidate validation summary classes | Adds refs and status composition after CP5. |
| Existing CR148 contracts | Backtest/report foundation | Consumed, not replaced. |
| `tests/research/test_daily_multifactor_baseline_artifact.py` | Candidate validation tests | Covers split, required metrics and fallback. |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
|---|---|---|
| 创建 | `engine/daily_multifactor_baseline_artifact.py` | Add validation summary and split manifest objects after CP5 approval. |
| 创建 | `tests/research/test_daily_multifactor_baseline_artifact.py` | Add validation and fallback tests. |
| 不修改 | CR148 modules unless CP5 narrows exact integration target | Reuse existing contracts through refs. |

## 5. 数据模型与持久化设计

No persistent store is introduced.

| 对象 / 字段 | 类型 | 约束 | 说明 |
|---|---|---|---|
| `HistoricalBacktestRef` | object | required | run/report/cost/risk refs. |
| `WalkForwardSplitManifest` | object | required | train/test windows, purge, embargo, fold ids. |
| `ValidationMetrics` | object | required | returns, drawdown, turnover, costs, capacity/liquidity summary. |
| `ValidationStatus` | enum | PASS/FAIL/NEEDS_REVIEW/BLOCKED | Used by S04/S05. |

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
|---|---|---|---|---|
| `build_validation_summary` | artifact, provenance, backtest refs | validation summary | S04/S05 | Aggregates historical and OOS refs. |
| `validate_split_manifest` | split manifest | status/reasons | tests/S04 | Ensures fold integrity. |
| `validation_allows_admission` | validation summary | bool/reasons | S04 | Blocks package when validation is blocked. |

## 7. 核心处理流程

1. Validate artifact contract.
2. Validate readonly provenance or fallback mode.
3. Attach historical backtest refs.
4. Attach OOS/walk-forward split and metrics.
5. Emit validation status and reasons.

## 8. 技术设计细节

- Purge/embargo are explicit fields; absence returns `NEEDS_REVIEW` or `BLOCKED` per CP5 policy.
- Metrics are refs/summaries, not raw performance frames.
- Capacity/liquidity summary is required even if status is `n/a-with-reason`.

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
|---|---|---|
| 安全 | Consumes S02 provenance; no writes. | Tests with unsafe provenance. |
| 性能 | Summary stores compact metrics. | Contract tests. |

## 10. 测试设计

| 测试场景 | 前置条件 | 操作 | 预期结果 | 验证方式 |
|---|---|---|---|---|
| Valid historical/OOS refs | S01/S02 pass | build summary | PASS/NEEDS_REVIEW per metrics | unit test |
| Missing split manifest | refs incomplete | validate | BLOCKED | unit test |
| Fixture fallback | S02 fallback | build summary | no real-data claim | unit test |
| Missing cost metric | metric absent | validate | NEEDS_REVIEW/BLOCKED | unit test |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
|---|---|---|---|---|
| TASK-CR155-S03-01 | 创建 | `engine/daily_multifactor_baseline_artifact.py` | Add validation summary and split manifest objects. | validation tests |
| TASK-CR155-S03-02 | 创建 | same | Add metric requiredness and fallback logic. | metric tests |
| TASK-CR155-S03-03 | 创建 | `tests/research/test_daily_multifactor_baseline_artifact.py` | Add split/fallback/metric tests. | all S03 tests |

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 选项与推荐 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
|---|---|---|---|---|---|---|
| LCQ-CR155-S03-01 | Missing purge/embargo should BLOCK or NEEDS_REVIEW | 推荐 BLOCKED for final package, NEEDS_REVIEW for draft validation | agent default | admission | CR154 CV governance | If CP5 requests softer policy, update S04/S05 |

| 风险 / 难点 | 影响 | 缓解措施 / 预研建议 |
|---|---|---|
| OOS semantics drift from CR154 | 中 | Keep split manifest explicit and referenced by S04. |

### OPEN / Spike 跟踪

| ID | 类型 | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| N/A | N/A | No blocking open item. | N/A | N/A |

## 13. 回滚与发布策略

- 发布方式：validation summary only after CP5/CP6.
- 回滚触发条件：split manifest invalid or provenance unsafe.
- 回滚动作：mark validation `BLOCKED` and admission package `paper_candidate=false`.

## 14. Definition of Done

- [ ] Historical and OOS/walk-forward refs are required.
- [ ] Split manifest validates fold integrity.
- [ ] Unsafe provenance blocks real-data claim.
- [ ] CP5 confirms this LLD before implementation.

## 人工确认区

- 结论：`pending`
- 审查人：
- 审查时间：
- 修改意见：
- 风险接受项：
