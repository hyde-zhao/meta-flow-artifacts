---
status: "ready-for-cp5-review"
version: "0.1"
cr_id: "CR-166"
feature_id: "FEAT-166-04"
owner: "meta-se-inline"
---

# FEAT-166-04 Existing-consumer Projections 设计

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-13 | host-orchestrator inline meta-se | 冻结三个既有 consumer 的薄投影、同一 identity、worse-state merge 与 CR155 回归。 |

## 1. 边界与文件

新增 `engine/walk_forward_oos_projections.py` 作为 mapping-only 层；修改以下三个 policy owner 只增加 typed identity/reason 消费面，不转移 policy：

| Consumer 文件 | 调用时机 | 同步修改 |
|---|---|---|
| `engine/strategy_admission_statistical_gate.py` | C2 self-validation 后、现有 statistical gate build 前 | `WalkForwardValidationPlan` 兼容投影增加 evidence ref/hash/availability/reasons；现有 pass-rate threshold owner 不变。 |
| `engine/cross_strategy_reliability_gates.py` | Gate 2 CV governance 前 | 映射 split/walk-forward/OOS/purge/embargo/leakage 与同一 C2 identity；Gate 2 policy 不变。 |
| `engine/strategy_admission_package.py` | package build/attach 阶段 | 增加 `attach_walk_forward_oos_evidence` 与 C2 blocked reason；runtime authorization flags 不变。 |

## 2. Projection API

| API | 输入 | 输出 |
|---|---|---|
| `project_to_statistical_walk_forward_plan` | validated C2 component | legacy-compatible plan mapping + evidence identity/reasons |
| `project_to_reliability_gate2` | same component | existing Gate 2 evidence mapping |
| `attach_walk_forward_oos_evidence` | package + same component | package copy，附 ref/reasons/limitations，状态只保持或变差 |

三个输出必须携带完全相同的 component ref/hash/availability/reason set；mismatch 直接 blocked，不能各自重算 raw folds。

## 3. 保守状态表

| C2 状态 | Projection 行为 | 禁止行为 |
|---|---|---|
| blocked | 三者添加 blocker；package/gate 不提升 | 生成 legacy PASS metrics |
| typed_unavailable | 保留 missing/unavailable reason | 当作 N/A 或 PASS |
| not_applicable | 只允许已冻结 strategy applicability；event 保持 N/A | 用 N/A 满足 mandatory daily/ML C2 |
| present+fail | 提供真实 fold summary，consumer 仍 fail/worse | 由其他 PASS 覆盖 |
| present+needs_review | consumer 可保持或变差 | 自动升级 PASS |
| present+pass | 仅 eligible；现有 policy 继续评估 | 声称 overall/paper/runtime readiness |

严重度 merge 使用各 consumer 已有枚举/blocked reason 机制，不引入第四个 gate 或全新全局 admission enum。

## 4. CR155 与 claim ceiling

CR155 historical package 的缺失/不足 evidence 必须保持 `paper_candidate=false` 和 blocked 1/1；不得回填或伪造 C2。C2 PASS 不改变 `not_qmt_authorization/not_simulation_authorization/not_live_authorization/not_broker_order`，Stage3/runtime/real-evidence flags 仍 false。

## 5. 错误、回滚与 Gotchas

projection input 未自校验、hash mismatch、unknown mandatory 或 consumer mapping 不完整时 fail closed。回滚可移除三处 attach/mapping，C2 component 独立保留为未消费证据。Gotcha：legacy `fold_metrics[*].passed` 只是兼容输出，必须来自 C2 重算；consumer 不能反向修改 component。Gotcha：`present+pass` 不是 paper candidate。
