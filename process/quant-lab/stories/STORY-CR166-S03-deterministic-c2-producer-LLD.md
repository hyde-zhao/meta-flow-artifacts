---
story_id: "CR166-S03-deterministic-c2-producer"
title: "Deterministic walk-forward/OOS C2 producer"
story_slug: "deterministic-c2-producer"
lld_version: "0.1"
tier: "L"
status: "confirmed"
confirmed: true
created_by: "host-orchestrator inline meta-dev"
created_at: "2026-07-13T12:32:00+08:00"
confirmed_by: "user"
confirmed_at: "2026-07-13T13:43:35+08:00"
feature_design_refs: ["docs/features/walk-forward-oos-producer/DESIGN.md", "docs/features/walk-forward-oos-producer/TEST-PLAN.md", "docs/features/walk-forward-oos-producer/TASKS.md", "docs/features/walk-forward-oos-validation/DESIGN.md", "docs/features/strategy-evidence-envelope/DESIGN.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["claim-sensitive-computation", "declared-denominator", "deterministic-hash", "provenance"], rationale: "Fold outcome and pass-rate evidence must be reproducible and self-validating."}
open_items: 0
---

# LLD: CR166-S03 — Deterministic C2 Producer

## 0. 上游设计依据（工程依据）

CR166 HLD §5.4/10/11、ADR-005/006/009、Domain RULE-166-03/06/07/09/10、FEAT-166-03 三件套、S01/S02 contracts。

## 1. 目标（Goal）

实现纯函数 producer，从 validated input 重算 fold/metric decisions 与 aggregate，生成 content-addressed C2 component/envelope，并以 self-validation 保证所有展示值可重算。

## 2. 需求（Requirements）

Functional：fold evidence、declared/observed/passed counts、pass rate、outcome、reasons、hash/provenance。NFR：10→1、recompute difference=0、non-finite=0、I/O=0、`O(folds×metrics)`。

## 3. 模块拆分与职责

| 逻辑 | 职责 |
|---|---|
| metric evaluator | direction/threshold → metric decision |
| fold evaluator | mandatory metric decisions → fold outcome/reasons |
| aggregator | declared denominator、counts、pass rate/outcome |
| identity builder | input/config/component/envelope domains 与 refs |
| self-validator | 重算 counts/outcome/hash，阻断 tamper |

## 4. 代码结构与文件影响

| 动作 | 文件 | 内容 |
|---|---|---|
| 修改 | `engine/walk_forward_oos_evidence.py` | `produce_walk_forward_oos_evidence`、fold/aggregate/hash/self-validation。 |
| 新建 | `tests/research/test_walk_forward_oos_producer.py` | producer positive/mixed/missing/determinism/tamper/precondition tests。 |

S03 在 W3 独占 C2 production file；不得修改 consumer modules。

## 5. 数据模型与持久化

`FoldEvidence`：fold_id、source refs、metric decisions、outcome/reasons。`WalkForwardOOSComponent`：availability/outcome、declared/observed/validated/passed count、pass_rate、folds、input/config/component hash/ref、provenance/limitations/reasons。无 persistence；created_at 不进入 hash，当前时钟不读取。

## 6. API / Interface

| API | 前置 | 输出 / 降级 |
|---|---|---|
| `produce_walk_forward_oos_evidence` | validation status=validated | present component/envelope；否则 non-present typed result |
| `validate_walk_forward_oos_component` | component + expected identity | PASS issues=0；mismatch/tamper blocked |
| `recompute_walk_forward_summary` | component folds/policies | counts/pass-rate/outcome for audit |

## 7. 核心流程

确认 validated → 遍历 manifest declared order → 重算每 metric/fold → aggregate using declared denominator → 构造 unsigned component → explicit-domain hashes/ref → envelope → self-recompute。任一步矛盾返回 blocked，不调用 projection。

## 8. 技术细节与决策表

missing/invalid fold 不从 denominator 删除；若 observed≠declared 则 availability non-present、pass_rate null。present+fail 允许显示 0..1 pass rate；needs_review 仅来自显式 metric/fold policy。stability/degradation 无 policy 时不推断。ref 形式由 type/version/hash 派生，不是 path。

## 9. 安全与性能

不 import filesystem/network/runtime；单次遍历；不生成随机 seed/current time。任何 authorization counter 已由 S02 阻断，S03 仍在 self-validation 中复核为 0。

## 10. 测试设计

all-pass、mixed fail、missing fold、ML input、10 reruns、tamper old hash、non-validated precondition、operation instrumentation；counts/pass rate/outcome 全部独立重算断言。

## 11. 实施步骤

T01 metric/fold evaluator；T02 declared aggregate；T03 domains/ref/provenance；T04 self-validator；T05 8 test groups/QAC evidence。

## 12. 风险与 Gotchas

无 OPEN/LCQ。Gotchas：pass-rate fact≠gate threshold；caller-provided `passed` 不是真相；missing fold 不能 shrink denominator；hash 不含 clock/path；consumer 不得回写 producer。

## 13. 回滚与发布

移除 producer 后 C2 保持 typed_unavailable，S01/S02 contract 可保留；不影响 C1、无数据迁移/发布。

## 14. DoD / Definition of Done

- [x] fold/aggregate 100% 可重算，difference=0
- [x] 10→1，tamper acceptance=0，false present=0
- [x] declared denominator 不因 bad fold 缩小
- [x] CP5 已批准，`confirmed=true`；允许进入 repository-local 实现
