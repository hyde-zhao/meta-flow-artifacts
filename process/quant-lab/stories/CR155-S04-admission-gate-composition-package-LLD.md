---
story_id: "CR155-S04-admission-gate-composition-package"
title: "Statistical/reliability gate composition and admission package"
story_slug: "admission-gate-composition-package"
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
  trigger_reasons: ["admission package", "CR151 and CR154 gate composition", "paper_candidate reason"]
  rationale: "Admission decisions must remain separately auditable and cannot be collapsed into one status."
open_items: 0
---

# LLD: CR155-S04 — Admission Gate Composition Package

## 0. 上游设计依据

| 来源 | 路径 / ID | 被本 LLD 消费的内容 |
|---|---|---|
| HLD | `docs/design/HLD-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT.md` | Admission package and paper_candidate scope. |
| ADR | `docs/design/ARCHITECTURE-DECISION-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT.md` | ADR-CR155-003. |
| S03 | `process/stories/CR155-S03-backtest-oos-walkforward-validation-LLD.md` | Validation summary input. |
| CR151/CR154 | summaries and closed evidence refs | Statistical and reliability gate contracts. |

## 1. Goal

Define the admission package contract that combines validation, statistical admission and cross-strategy reliability results without losing individual reasons.

## 2. Requirements

### 2.1 Functional

- Preserve statistical gate status and reasons.
- Preserve reliability gate status and reasons.
- Compute final admission package status and `paper_candidate=true|false`.
- Include blockers, risks, evidence refs and non-authorization wording.

### 2.2 Non-Functional

- Fail closed on missing mandatory gate refs.
- No production, runtime, trading or paper-readiness claim.

## 3. 模块拆分与职责

| 模块 / 文件组 | 职责 | 说明 |
|---|---|---|
| `engine/daily_multifactor_baseline_artifact.py` | Candidate admission package objects | Composes refs and statuses. |
| CR151 evaluator refs | Statistical status | Consumed, not reimplemented. |
| CR154 reliability refs | Reliability status | Consumed, not reimplemented. |
| tests | Package status tests | Covers missing gates and paper_candidate. |

## 4. 代码结构与文件影响范围

| 动作 | 文件路径 | 变更内容 |
|---|---|---|
| 创建 | `engine/daily_multifactor_baseline_artifact.py` | Add admission package composition after CP5 approval. |
| 创建 | `tests/research/test_daily_multifactor_baseline_artifact.py` | Add composition and fail-closed tests. |
| 不修改 | CR151/CR154 closed modules unless CP5 identifies exact adapter hooks | Consume existing contracts. |

## 5. 数据模型与持久化设计

No persistent registry/store write is introduced.

| 对象 / 字段 | 类型 | 约束 | 说明 |
|---|---|---|---|
| `statistical_gate` | object | required | status, reasons, evidence refs. |
| `reliability_gate` | object | required | status, reasons, evidence refs. |
| `validation_summary` | object | required | S03 output. |
| `paper_candidate` | bool | required | Research field only, not authorization. |
| `package_status` | enum | PASS/FAIL/NEEDS_REVIEW/BLOCKED | Final package decision. |
| `non_authorization` | list[str] | required | Blocks overclaim. |

## 6. API / Interface 设计

| 接口 / 入口 | 输入 | 输出 | 调用方 | 说明 |
|---|---|---|---|---|
| `compose_admission_package` | artifact, validation, stat gate, reliability gate | package | S05/release evidence | Maintains separate reasons. |
| `derive_paper_candidate` | package inputs | bool/reasons | tests/release wording | Research admission only. |
| `validate_admission_package` | package | status/reasons | CP7 tests | Fail-closed missing mandatory refs. |

## 7. 核心处理流程

1. Validate artifact and validation summary.
2. Attach CR151 statistical gate status/ref.
3. Attach CR154 reliability gate status/ref.
4. Derive final package status and `paper_candidate`.
5. Add blockers, risks and non-authorization wording.

## 8. 技术设计细节

- `paper_candidate=true` requires no mandatory gate `FAIL` or `BLOCKED` and no unhandled `NEEDS_REVIEW`.
- `paper_candidate` never authorizes paper trading.
- Missing statistical or reliability ref yields `BLOCKED`.

## 9. 安全与性能设计

| 维度 | 设计措施 | 验证方式 |
|---|---|---|
| 安全 | Required non-authorization list in package. | Wording tests. |
| 性能 | Package is refs-only. | Serialization test. |

## 10. 测试设计

| 测试场景 | 前置条件 | 操作 | 预期结果 | 验证方式 |
|---|---|---|---|---|
| All gates pass | stat/reliability pass | compose | candidate true with reasons | unit test |
| Statistical missing | no stat ref | compose | BLOCKED | unit test |
| Reliability fail | reliability fail | compose | candidate false | unit test |
| Needs review | one gate needs_review | compose | package needs_review or candidate false | unit test |

## 11. 实施步骤

| TASK-ID | 动作 | 目标文件 | 详细描述 | 对应测试 |
|---|---|---|---|---|
| TASK-CR155-S04-01 | 创建 | `engine/daily_multifactor_baseline_artifact.py` | Add package dataclass and composition function. | package tests |
| TASK-CR155-S04-02 | 创建 | same | Add `paper_candidate` derivation and non-authorization wording. | wording tests |
| TASK-CR155-S04-03 | 创建 | `tests/research/test_daily_multifactor_baseline_artifact.py` | Add missing/ref/status composition tests. | all S04 tests |

## 12. 风险、难点与预研建议

### 12.1 实现灰区与取舍记录

| Clarification ID | 问题 | 选项与推荐 | 决策 / 答案 | 影响面 | 证据 | 重访条件 |
|---|---|---|---|---|---|---|
| LCQ-CR155-S04-01 | `NEEDS_REVIEW` 是否允许 `paper_candidate=true` | 推荐不允许，除非 CP5 明确接受风险 | agent default | admission | HLD G4 | If user accepts risk, add CP5 risk item |

| 风险 / 难点 | 影响 | 缓解措施 / 预研建议 |
|---|---|---|
| Separate gate reasons get collapsed | 高 | Package schema stores each gate block separately. |

### OPEN / Spike 跟踪

| ID | 类型 | 问题 | 下一动作 | 责任方 |
|---|---|---|---|---|
| N/A | N/A | No blocking open item. | N/A | N/A |

## 13. 回滚与发布策略

- 发布方式：admission package generated after validation evidence exists.
- 回滚触发条件：gate refs missing or package wording overclaims.
- 回滚动作：set package `BLOCKED` and `paper_candidate=false`.

## 14. Definition of Done

- [ ] Statistical and reliability gate reasons are separately visible.
- [ ] Missing gate refs fail closed.
- [ ] `paper_candidate` wording is research-only.
- [ ] CP5 confirms this LLD before implementation.

## 人工确认区

- 结论：`pending`
- 审查人：
- 审查时间：
- 修改意见：
- 风险接受项：
