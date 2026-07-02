---
story_id: "CR152-S02-purged-embargo-cv-fixture-contract"
title: "Purged embargo CV and fixture contract"
story_slug: "purged-embargo-cv-fixture-contract"
lld_version: "1.0"
tier: "M"
status: "ready-for-review"
confirmed: false
created_by: "host-orchestrator"
created_at: "2026-07-02T10:37:00+08:00"
feature_design_refs:
  - "docs/features/factor-research-loop/DESIGN.md"
  - "docs/features/factor-research-loop/TEST-PLAN.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "purged + embargo CV contract"
    - "fixture schema required to prove split behavior"
  rationale: "CR152 needs deterministic split and fixture semantics before metadata and gate stories can link evidence."
open_items: 0
---

# LLD: CR152-S02 — Purged Embargo CV and Fixture Contract

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| HLD | `process/docs/design/HLD-ML-STRATEGY-E2E-FRAMEWORK.md` | Purged + embargo CV and fixture contract |
| ADR | `process/docs/design/ARCHITECTURE-DECISION-ML-STRATEGY-E2E-FRAMEWORK.md` | deterministic fixture-only validation |
| Feature Matrix | `process/docs/design/FEATURE-DESIGN-MATRIX.md` | S02 full-lld |
| S01 LLD | `process/stories/CR152-S01-pit-feature-label-contracts-LLD.md` | PIT / label fields consumed by split fixtures |
| Existing source anchor | `engine/research_manifest.py` | `ExperimentSplitPolicy` fields for split metadata alignment |

## 1. Goal

Define purged + embargo CV split policy, split audit and deterministic fixture contract that can prove both passing and fail-closed split behavior without real data.

## 2. Requirements（Functional / Non-Functional）

### Functional

- Define `PurgedEmbargoCVSpec` and `PurgedEmbargoSplitAudit`.
- Validate chronological fold windows, purge windows and embargo gaps.
- Require fixture fields sufficient to exercise PIT and split logic.
- Provide positive and negative fixture cases for CP6 tests.

### Non-Functional

- Static/fixture-only; no real lake, provider, NAS or external framework.
- Deterministic issue codes and no random folds.
- Small fixture size but enough chronological blocks to prove semantics.

## 3. 模块拆分与职责

| 模块 / 文件组 | 职责 | 说明 |
|---|---|---|
| `engine/research_production_contracts.py` | CV spec, fixture contract and split audit validators | S02 extends S01 field anchors. |
| `tests/research/test_ml_strategy_e2e_contracts.py` | Positive / negative CV fixtures | Reuse S01 PIT/label fixture helpers if created. |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
|---|---|---|
| 修改 | `engine/research_production_contracts.py` | Add split policy/audit dataclasses and validators. |
| 修改 | `tests/research/test_ml_strategy_e2e_contracts.py` | Add fold overlap, purge and embargo tests. |

## 5. 数据模型与持久化设计

| 对象 / 字段 | 类型 | 约束 | 说明 |
|---|---|---|---|
| `PurgedEmbargoFold` | dataclass | fold id, train/validation/test start/end required | Atomic fold window. |
| `PurgedEmbargoCVSpec` | dataclass | `cv_id`, `folds`, `purge_window_days`, `embargo_days`, `label_horizon_days` | Static split policy. |
| `PurgedEmbargoSplitAudit` | dataclass | `status`, `issues`, `fold_count`, `operation_counts` | Audit output consumed by S04. |
| `MLFixtureDataContract` | dataclass | required fields list and minimum chronological blocks | Describes fixture shape, not real data. |

Minimum fixture fields: `entity_id`, `trade_date`, `decision_time`, `feature_available_at`, `label_available_at`, `label_window_start`, `label_window_end`, `fold_id`, `prediction_timestamp`, `feature_value`, `label_value`.

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
|---|---|---|---|---|
| `validate_purged_embargo_cv_spec(spec)` | dataclass or mapping | tuple issues | tests, S04 | Checks required folds and nonnegative windows. |
| `audit_purged_embargo_splits(spec)` | CV spec | `PurgedEmbargoSplitAudit` | S04 | Detects overlap, purge and embargo violations. |
| `validate_ml_fixture_contract(contract)` | dataclass or mapping | tuple issues | tests | Ensures fixture can exercise CV behavior. |

## 7. 核心处理流程

1. S01 provides PIT/label field names.
2. S02 defines fold windows and fixture requirements.
3. Validator checks fold date ordering.
4. Audit checks train/validation/test separation, purge and embargo.
5. S04 maps missing metadata to `BLOCKED` and semantic split failures to `FAIL` unless mandatory metadata is absent.

## 8. 技术设计细节

- Use parseable ISO date strings in fixtures; avoid pandas dependency unless already imported nearby.
- Treat missing fold metadata as `BLOCKED` evidence issue.
- Treat explicit overlap/embargo violation with complete metadata as `FAIL`.
- Keep fixture rows inline in tests; no external CSV/Parquet required.

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
|---|---|---|
| Safety | Fixture rows only; no real data path. | Tests assert no operation counters. |
| Performance | Small fold list; O(n) pairwise checks acceptable for tiny fixture. | Unit tests only. |
| Determinism | No random split generation. | Stable fixture ids. |

## 10. 测试设计

| 测试场景 | 前置条件 | 操作 | 预期结果 | 验证方式 |
|---|---|---|---|---|
| Valid fold spec passes | 5 chronological blocks | validate/audit | no blocker/fail issue | pytest |
| Missing fold windows blocked | missing dates | validate spec | `BLOCKED` issue | pytest |
| Train/test overlap fails | complete metadata with overlap | audit | `FAIL` issue | pytest |
| Embargo gap violation fails | complete metadata with too-small gap | audit | `FAIL` issue | pytest |
| Fixture contract too small blocked | fewer than minimum blocks | validate fixture contract | `BLOCKED` issue | pytest |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
|---|---|---|---|---|
| CR152-S02-T01 | 修改 | `engine/research_production_contracts.py` | Add fold/CV/fixture dataclasses. | serialization |
| CR152-S02-T02 | 修改 | `engine/research_production_contracts.py` | Add validators and audit function. | overlap/embargo tests |
| CR152-S02-T03 | 修改 | `tests/research/test_ml_strategy_e2e_contracts.py` | Add positive and negative fixture cases. | S02 test slice |

## 12. 风险、难点与预研建议

| 风险 / 难点 | 影响 | 缓解措施 |
|---|---|---|
| Fixture too small to prove embargo | Empty-shell validation | Minimum chronological blocks and explicit negative cases required. |
| Missing metadata confused with failed CV | Wrong gate status | Missing metadata -> `BLOCKED`; complete violation -> `FAIL`. |

### OPEN / Spike 跟踪

| ID | 类型 | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| N/A | OPEN | No blocking open item. | N/A | N/A |

## 13. 回滚与发布策略

Rollback removes S02 dataclasses/validators and tests. No data migration or persistent cleanup is required.

## 14. Definition of Done

- [ ] CV spec and split audit contracts exist.
- [ ] Fixture contract minimum fields are enforced.
- [ ] Passing, overlap and embargo tests exist.
- [ ] Missing metadata is distinguishable from semantic failure.
- [ ] No real data or external framework operation is introduced.

## 人工确认区

| # | 检查项 | 状态 | 证据 |
|---|---|---|---|
| 1 | LLD 覆盖 AC | pending | 第 2 / 10 / 14 节 |
| 2 | Fixture contract 足以验证 CV | pending | 第 5 / 10 节 |
| 3 | 失败路径明确 | pending | 第 7 / 8 / 12 节 |

**人工审查结果回填**：

- 结论：`pending`
- 审查人：
- 审查时间：
- 修改意见：
- 风险接受项：
