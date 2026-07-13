---
story_id: "CR166-S02-fold-validator-adapters"
title: "Fold, leakage and sufficiency validator with daily/ML adapters"
story_slug: "fold-validator-adapters"
lld_version: "0.1"
tier: "L"
status: "confirmed"
confirmed: true
created_by: "host-orchestrator inline meta-dev"
created_at: "2026-07-13T12:31:00+08:00"
confirmed_by: "user"
confirmed_at: "2026-07-13T13:43:35+08:00"
feature_design_refs: ["docs/features/walk-forward-oos-validation/DESIGN.md", "docs/features/walk-forward-oos-validation/TEST-PLAN.md", "docs/features/walk-forward-oos-validation/TASKS.md", "docs/features/strategy-evidence-envelope/DESIGN.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["temporal-integrity", "leakage-risk", "cross-strategy-adapters", "authorization-boundary"], rationale: "A permissive validator would create false OOS evidence."}
open_items: 0
---

# LLD: CR166-S02 — Fold Validator and Adapters

## 0. 上游设计依据（工程依据）

CR166 HLD §5.3/6.1/10、ADR-003/004/005/008/009、Domain RULE-166-01..06/08/13..16、FEAT-166-02 三件套、S01 contract。

## 1. 目标（Goal）

在 `engine/walk_forward_oos_evidence.py` 建立唯一 common validator，输出 typed `validated/typed_unavailable/blocked`，并把 daily/ML 已给事实映射到 common input；event 返回结构化 N/A。

## 2. 需求（Requirements）

7/7 字段族均有 validation result；时间/purge/embargo 3/3 negative blocked；daily/ML 2/2；8 类 P0 reason 非空；external dereference=0；event producer/fixture=0。

## 3. 模块拆分与职责

| 逻辑 | 职责 |
|---|---|
| normalizer | ISO-8601、IDs、metric/policy/value shape；不修复缺失事实 |
| authorization precheck | ref classification 与 counters，必须最先运行 |
| temporal/leakage validator | half-open fold、cutoff、purge/embargo minima |
| metric/lineage validator | finite、mandatory、ref/hash/membership |
| daily/ML adapters | legacy value → common value；不改 source objects |
| event applicability | explicit N/A + owner/trigger |

## 4. 代码结构与文件影响

| 动作 | 文件 | 内容 |
|---|---|---|
| 修改 | `engine/walk_forward_oos_evidence.py` | issue/result types、validation pipeline、daily/ML adapters、event evaluator。 |
| 新建 | `tests/research/test_walk_forward_oos_validation.py` | positive/negative/boundary/permission fixtures。 |
| 只读 | `engine/daily_multifactor_baseline_artifact.py` | `WalkForwardSplitManifest` input。 |
| 只读 | `engine/research_production_contracts.py` | `MLPurgedEmbargoCVPolicy` / `EventTimeSemantics` input。 |

## 5. 数据模型与持久化

`TemporalFold` 显式 train/validation/OOS start/end；`PurgeEmbargoPolicy` 显式 unit/applicability/horizon/required/applied/ref；`MetricPolicy` 明确 id/direction/threshold；`LineageBinding` 明确 ref/hash/membership/sources；`AuthorizationMetadata` 明确 validation mode/ref class/counters。无持久化与 dereference。

## 6. API / Interface

| API | 输入 | 输出 |
|---|---|---|
| `validate_walk_forward_oos_input` | common input | `WalkForwardInputValidation` |
| `adapt_daily_walk_forward_input` | legacy manifest + explicit companion facts | common input/typed issues |
| `adapt_ml_walk_forward_input` | ML policy + explicit OOS folds/metrics/lineage/auth | common input/typed issues |
| `event_walk_forward_applicability` | event semantics fact | N/A result with reason/owner/trigger |

## 7. 核心流程与失败路径

authorization→schema/inventory→temporal→purge/embargo→metric→lineage。缺失且无矛盾为 typed_unavailable；非法、矛盾、tamper、unauthorized 为 blocked。issues 按 `(severity, fold_id, field, code)` 稳定排序。只有 zero issues/complete input 为 validated。

## 8. 技术细节

半开区间要求每 fold `train_start < train_end <= validation_start < validation_end <= oos_start < oos_end`。允许 rolling/expanding train 跨 fold overlap；cutoff 单调。purge/embargo unit 必须相同且 applied≥required。ML `test_*` 只有显式声明为 OOS 才映射。

## 9. 安全与性能

不得 import resolver/client/runtime 模块；不调用 `Path`, env, network。单遍 folds×metrics。operation counter 非 0 在任何解析 external ref 前 blocked。

## 10. 测试设计

daily/ML positives；missing/empty/count mismatch；reversed/illegal overlap；purge missing/insufficient；embargo one-below/exact；metric missing/NaN/Inf；lineage missing/ref/hash/membership mismatch；unauthorized ref；event N/A。

## 11. 实施步骤

T01 authorization/sufficiency；T02 temporal/purge/embargo；T03 metric/lineage；T04 adapters/event N/A；T05 10 test groups 与 reason completeness。

## 12. 风险与 Gotchas

无 OPEN/LCQ。Gotchas：相邻 end=start 合法；purge ref 不证明充分；ML policy 不替代实际 fold；event time 不等于 event fold；missing 与 contradiction 不同。

## 13. 回滚与发布

可回退 adapter/validator，所有 C2 consumer 保持 unavailable；不改 legacy daily/ML contracts，不做数据迁移或发布。

## 14. DoD / Definition of Done

- [x] 7/7 字段族、daily/ML=2/2、8/8 P0 fail-closed 可验证
- [x] temporal/leakage=3/3 blocked；exact-boundary 继续
- [x] event N/A=1/1、event producer/feed access=0
- [x] CP5 已批准，`confirmed=true`；仍禁止真实操作
