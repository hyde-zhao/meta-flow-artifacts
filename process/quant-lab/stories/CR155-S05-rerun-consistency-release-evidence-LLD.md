---
story_id: "CR155-S05-rerun-consistency-release-evidence"
title: "Rerun consistency and release evidence wording"
story_slug: "rerun-consistency-release-evidence"
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
  - "docs/features/daily-multifactor-baseline-strategy-artifact/TEST-PLAN.md"
  - "docs/features/daily-multifactor-baseline-strategy-artifact/TASKS.md"
lld_policy:
  required_level: "full-lld"
  trigger_reasons: ["two-run metric comparison", "release evidence", "overclaim prevention"]
  rationale: "Rerun consistency is a CR155 acceptance criterion and release wording must block readiness overclaim."
open_items: 0
---

# LLD: CR155-S05 — Rerun Consistency and Release Evidence

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| HLD | `docs/design/HLD-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT.md` | Mandatory two-rerun evidence. |
| ADR | `docs/design/ARCHITECTURE-DECISION-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT.md` | ADR-CR155-004 and no-runtime boundary. |
| S03 | `process/stories/CR155-S03-backtest-oos-walkforward-validation-LLD.md` | Validation metrics and refs. |
| S04 | `process/stories/CR155-S04-admission-gate-composition-package-LLD.md` | Admission package status. |

## 1. Goal

Define rerun consistency evidence and release wording that prove the baseline artifact is reproducible enough for research admission without claiming runtime readiness.

## 2. Requirements

### 2.1 Functional

- Compare two reruns over return, drawdown, turnover, cost, capacity/liquidity summary and admission status.
- Produce structured drift reasons when metrics differ.
- Block `paper_candidate=true` or mark `needs_review` when unexplained drift exists.
- Generate release wording that states research artifact only.

### 2.2 Non-Functional

- Deterministic comparison keys.
- Numeric tolerance policy must be explicit.
- No actual publish/production deployment.

## 3. 模块拆分与职责

| 模块 / 文件组 | 职责 | 说明 |
|---|---|---|
| `engine/daily_multifactor_baseline_artifact.py` | Candidate rerun diff objects | Adds comparison logic after CP5 approval. |
| `tests/research/test_daily_multifactor_baseline_artifact.py` | Candidate rerun/release tests | Covers metric equality, drift and wording. |
| Release evidence refs | Later CP7/CP8 input | Research-only wording. |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
|---|---|---|
| 创建 | `engine/daily_multifactor_baseline_artifact.py` | Add rerun comparison and release wording helpers after CP5 approval. |
| 创建 | `tests/research/test_daily_multifactor_baseline_artifact.py` | Add rerun drift and wording tests. |
| 不修改 | deployment/publish/runtime files | No publish or production operation. |

## 5. 数据模型与持久化设计

No persistent release registry/store write is introduced.

| 对象 / 字段 | 类型 | 约束 | 说明 |
|---|---|---|---|
| `RerunMetricSnapshot` | object | required twice | metric values and run refs. |
| `RerunConsistencyReport` | object | required | metric diffs, status and reasons. |
| `metric_tolerance` | object | required | default exact for status/enums; numeric tolerance explicit. |
| `release_wording` | list[str] | required | research-only wording. |

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
|---|---|---|---|---|
| `compare_rerun_metrics` | snapshot A/B, tolerance | consistency report | admission/release | Compares required metrics. |
| `rerun_allows_candidate` | consistency report | bool/reasons | S04/release | Blocks unexplained drift. |
| `build_release_wording` | artifact, package, rerun report | wording refs | CP8 docs | Research-only claim boundary. |

## 7. 核心处理流程

1. Validate both rerun snapshots have required metrics.
2. Compare metric keys with tolerance.
3. Emit PASS/FAIL/NEEDS_REVIEW with structured reasons.
4. Update final evidence wording.

## 8. 技术设计细节

- Default tolerance: exact match for statuses/enums/refs; numeric tolerance defaults to zero unless CP5 explicitly changes it.
- Any unexplained metric drift forces `NEEDS_REVIEW` or `paper_candidate=false`.
- Release wording must include no runtime/trading/production authorization.

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
|---|---|---|
| 安全 | Release wording deny-list for paper/live/trading claims. | Wording tests. |
| 性能 | Comparison over compact metric snapshots. | Unit tests. |

## 10. 测试设计

| 测试场景 | 前置条件 | 操作 | 预期结果 | 验证方式 |
|---|---|---|---|---|
| Identical reruns | snapshots equal | compare | PASS | unit test |
| Drawdown drift | numeric difference | compare | FAIL/NEEDS_REVIEW | unit test |
| Status drift | admission status differs | compare | FAIL | unit test |
| Wording overclaim | wording says live-ready | validate wording | FAIL | unit test |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
|---|---|---|---|---|
| TASK-CR155-S05-01 | 创建 | `engine/daily_multifactor_baseline_artifact.py` | Add metric snapshot and comparison report objects. | rerun tests |
| TASK-CR155-S05-02 | 创建 | same | Add tolerance and release wording helpers. | wording tests |
| TASK-CR155-S05-03 | 创建 | `tests/research/test_daily_multifactor_baseline_artifact.py` | Add drift/status/wording tests. | all S05 tests |

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 选项与推荐 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
|---|---|---|---|---|---|---|
| LCQ-CR155-S05-01 | Numeric tolerance default | 推荐 zero tolerance until implementation proves deterministic rounding needs epsilon | agent default | tests/admission | HLD G5 | If flaky numeric rounding appears, CP7 may propose tolerance CR |

| 风险 / 难点 | 影响 | 缓解措施 / 预研建议 |
|---|---|---|
| Flaky rerun metrics | 高 | Structured drift reason and fail-closed candidate policy. |

### OPEN / Spike 跟踪

| ID | 类型 | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| N/A | N/A | No blocking open item. | N/A | N/A |

## 13. 回滚与发布策略

- 发布方式：research release evidence only after CP7.
- 回滚触发条件：rerun comparison fails or wording overclaims.
- 回滚动作：mark admission `needs_review` or `paper_candidate=false`.

## 14. Definition of Done

- [ ] Required metric keys are compared.
- [ ] Status drift fails closed.
- [ ] Release wording blocks paper/live/runtime/trading claims.
- [ ] CP5 confirms this LLD before implementation.

## 人工确认区

- 结论：`pending`
- 审查人：
- 审查时间：
- 修改意见：
- 风险接受项：
