---
check_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P2-SIMULATION-ORDER-PLAN-2026-06-25"
workflow_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-2026-06-25"
created_at: "2026-06-25T11:47:06+08:00"
owner: "host-orchestrator"
phase_goal: "P2-simulation-order-plan"
stage_decision: "PASS"
target_status: "simulation-order-plan-ready"
evidence_ref: "process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P2-EVIDENCE-2026-06-25.json"
runtime_authorization_granted: false
---

# P2 Simulation Order Plan 检查结果

## 1. 结论

| 项目 | 内容 |
|---|---|
| 阶段决策 | `PASS` |
| 当前 P2 状态 | `simulation-order-plan-ready` |
| 下一阶段 | `P3-simulation-execution-engine` |
| Evidence | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P2-EVIDENCE-2026-06-25.json` |

P2 已实现并验证 simulation order plan：从 P1 target portfolio、当前持仓摘要、现金和 raw price refs 生成 buy/sell order intents；只生成计划，不提交、不撤单、不触达 QMT runtime。

## 2. Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| P1 target portfolio 已通过 | PASS | `process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P1-MULTIFACTOR-TARGET-PORTFOLIO-2026-06-25.md` | P2 消费 target rows |
| 风控 fixture 合同可用 | PASS | `trading/pretrade_risk.py` | 使用离线 `RiskInputSnapshot` / `RiskProfile` |
| 不授权 runtime | PASS | 用户指令 + evidence | P2 只生成计划 |

## 3. Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | Order plan 输入合同存在 | PASS | `SimulationOrderPlanRequest` | 包含 target rows、capital、risk snapshot、risk profile |
| 2 | buy/sell intent 生成 | PASS | 新增测试 | 覆盖一买一卖 |
| 3 | lot size 与目标数量计算 | PASS | `build_simulation_order_plan` | 先按 lot 向下取整，再由 pretrade risk 校验 |
| 4 | 现金 / 可卖 / 持仓 / 限额 gate | PASS | `evaluate_many` | 风控失败返回 `pretrade_risk_blocked` |
| 5 | 换手限额 gate | PASS | `max_turnover_notional` | 超限在执行前 blocked |
| 6 | no-op 明确处理 | PASS | `orders=()` | 目标等于当前仓位时 generated 且无订单 |
| 7 | 脱敏输出 | PASS | `SimulationOrderPlanResult.to_dict` | 输出 instrument_ref / quantity bucket / price_ref / notional bucket |

## 4. Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| P2 plan 可供 P3 消费 | PASS | `order_intents()` | P3 可读取通过风控的 intents |
| fail-closed 场景覆盖 | PASS | 新增测试 | 现金不足在 gateway 前阻断 |
| 验证通过 | PASS | `29 passed` + py_compile | P0-P3 相关测试通过 |

## 5. Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Order plan 合同 | `trading/strategy_runner/simulation_order_plan.py` | PASS | 新增 P2 API |
| 包导出 | `trading/strategy_runner/__init__.py` | PASS | 导出 P2 API |
| 测试 | `tests/test_runner_multifactor_runtime_pipeline.py` | PASS | 覆盖成功和风控阻断 |
| Evidence | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-P2-EVIDENCE-2026-06-25.json` | PASS | 结构化证据 |

## 6. 阶段决策

结论：`PASS`

P2 已完成。下一步进入 P3：使用 caller-provided simulation gateway interface 执行已通过的 order plan；仍不启动真实 runtime、不读取凭据。
