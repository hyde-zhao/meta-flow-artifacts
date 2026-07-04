---
story_id: "CR155-S01-baseline-artifact-contract"
title: "Baseline artifact contract and claim boundary"
story_slug: "baseline-artifact-contract"
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
  trigger_reasons: ["new strategy artifact contract", "cross-story shared schema", "admission package consumer"]
  rationale: "The artifact contract is the CR155 audit surface and every downstream Story consumes it."
open_items: 0
---

# LLD: CR155-S01 — Baseline Artifact Contract

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| HLD | `docs/design/HLD-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT.md` | Standalone artifact contract, non-optimal baseline boundary. |
| ADR | `docs/design/ARCHITECTURE-DECISION-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT.md` | ADR-CR155-001, ADR-CR155-003, ADR-CR155-005. |
| Feature Matrix | `docs/design/FEATURE-DESIGN-MATRIX.md#cr155-cp4-增量daily-multifactor-baseline-strategy-artifact` | `full-lld` policy and FEAT-16 ownership. |
| Feature DESIGN | `docs/features/daily-multifactor-baseline-strategy-artifact/DESIGN.md` | Required artifact field groups and CP5 focus items. |

## 1. Goal

Define the typed daily multifactor baseline strategy artifact contract that all CR155 validation, admission and rerun evidence attaches to.

## 2. Requirements

### 2.1 Functional

- Define required fields for identity, universe, factors, signal, portfolio policy, validation refs, admission refs and evidence refs.
- Define fail-closed validation for missing required refs.
- Define claim boundary text: research baseline only, non-optimal, no paper/live/trading readiness.

### 2.2 Non-Functional

- JSON-safe serialization must be deterministic.
- Artifact schema must be stable enough for rerun comparison.
- No lake, credential, provider, NAS or runtime operation is part of this Story.

## 3. 模块拆分与职责

| 模块 / 文件组 | 职责 | 说明 |
|---|---|---|
| `engine/daily_multifactor_baseline_artifact.py` | Candidate contract module | Defines dataclasses/validators after CP5 approval. |
| `tests/research/test_daily_multifactor_baseline_artifact.py` | Candidate contract tests | Covers required fields and fail-closed behavior after CP5 approval. |
| Process evidence | CP5 LLD + later CP6 evidence | Long-term audit trail. |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
|---|---|---|
| 创建 | `engine/daily_multifactor_baseline_artifact.py` | Add artifact contract classes and validation helpers after CP5 approval. |
| 创建 | `tests/research/test_daily_multifactor_baseline_artifact.py` | Add schema and validation tests after CP5 approval. |
| 不修改 | lake / catalog / runtime paths | No write, publish or runtime integration. |

## 5. 数据模型与持久化设计

No persistent store is introduced.

| 对象 / 字段 | 类型 | 约束 | 说明 |
|---|---|---|---|
| `DailyMultifactorBaselineArtifact.strategy_id` | str | required, stable | Unique research strategy identity. |
| `schema_version` | str | required | Contract version. |
| `universe_ref` | str | required | PIT universe evidence reference, not inline data. |
| `factor_specs` | list[ref] | non-empty | Factor refs and deterministic weights/policy. |
| `signal_spec` | object | required | Standardization, lag and ranking rules. |
| `portfolio_policy` | object | required | Sizing, rebalance, cost/slippage and risk policy refs. |
| `validation_refs` | object | required | Historical/OOS/walk-forward refs from S03. |
| `admission_refs` | object | required | Statistical, reliability and package refs from S04. |
| `rerun_refs` | object | required before final package | Two-run evidence from S05. |
| `claim_boundary` | str | required | Must state research baseline only. |

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
|---|---|---|---|---|
| `validate_baseline_artifact` | Artifact object | validation report | S03/S04/S05 and tests | Fail-closed required field validator. |
| `artifact_to_json_dict` | Artifact object | dict | admission package writer | Deterministic serialization. |
| `build_claim_boundary` | strategy metadata | string/list | release evidence | No overclaim wording. |

## 7. 核心处理流程

1. Build artifact with identity and policy refs.
2. Validate all required ref groups.
3. Serialize deterministic artifact summary.
4. Expose validation report to S03/S04/S05.

## 8. 技术设计细节

- Required fields are explicit, not inferred from backtest reports.
- Missing mandatory ref returns `BLOCKED`, not `NEEDS_REVIEW`.
- `paper_candidate` is not stored in S01 artifact; it belongs to S04 admission package.

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
|---|---|---|
| 安全 | Contract-only; forbidden operation counters remain zero. | Unit tests assert no IO hooks. |
| 性能 | Data object is small, refs-only. | Serialization tests. |

## 10. 测试设计

| 测试场景 | 前置条件 | 操作 | 预期结果 | 验证方式 |
|---|---|---|---|---|
| Complete artifact validates | All required refs present | call validator | PASS | unit test |
| Missing universe blocks | `universe_ref` absent | call validator | BLOCKED with reason | unit test |
| Claim boundary blocks overclaim | text includes live/paper readiness | call validator | FAIL/BLOCKED | unit test |
| Serialization deterministic | same input twice | serialize | equal dict/hash | unit test |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
|---|---|---|---|---|
| TASK-CR155-S01-01 | 创建 | `engine/daily_multifactor_baseline_artifact.py` | Add artifact dataclasses and status enums. | contract tests |
| TASK-CR155-S01-02 | 创建 | same | Add validator and deterministic serializer. | validation tests |
| TASK-CR155-S01-03 | 创建 | `tests/research/test_daily_multifactor_baseline_artifact.py` | Add required-field and claim-boundary tests. | all S01 tests |

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 选项与推荐 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
|---|---|---|---|---|---|---|
| LCQ-CR155-S01-01 | Artifact 是否包含 `paper_candidate` 字段 | 推荐不包含；由 S04 admission package 负责 | agent default | 接口 | ADR-CR155-003 | 若用户要求 artifact 即 package，再回 CP5 |

| 风险 / 难点 | 影响 | 缓解措施 / 预研建议 |
|---|---|---|
| Artifact 被误读为可交易策略 | 高 | Required claim boundary and release wording tests. |

### OPEN / Spike 跟踪

| ID | 类型 | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| N/A | N/A | No blocking open item. | N/A | N/A |

## 13. 回滚与发布策略

- 发布方式：research artifact code path after CP5/CP6 only.
- 回滚触发条件：schema causes downstream ambiguity or overclaim.
- 回滚动作：disable artifact creation and keep CR155 at `needs_review`.

## 14. Definition of Done

- [ ] Contract validates required refs fail-closed.
- [ ] Serialization is deterministic.
- [ ] Claim boundary prevents paper/live/trading readiness wording.
- [ ] CP5 confirms this LLD before implementation.

## 人工确认区

- 结论：`pending`
- 审查人：
- 审查时间：
- 修改意见：
- 风险接受项：
