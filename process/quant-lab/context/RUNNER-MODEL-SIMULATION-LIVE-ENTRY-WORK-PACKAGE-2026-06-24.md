---
work_package_id: "RUNNER-MODEL-SIMULATION-LIVE-ENTRY-2026-06-24"
status: "verified-implementation-slice-completed"
workflow_mode: "standard"
meta_flow_route: "new-goal-package"
legacy_cr_queue_used: false
created_at: "2026-06-24T00:00:00+08:00"
owner: "host-orchestrator"
---

# Runner Model / Simulation / Live Entry Work Package

## 1. 目标

本目标包按新的 meta-flow 流程处理，不消费旧 CR 拆分队列作为执行顺序。

目标是把 runner 从离线策略包执行推进到模型盘 / 模拟盘 / 实盘入口条件：

- 保留既有 offline runner 的 `not_authorization=true` 合同。
- 新增已授权 simulation 阶段的独立入口，不把 simulation 授权混入旧 offline `RunSpec`。
- 支持目标组合进入模拟账户前的 stage gate、pretrade risk、OMS order intent、gateway submit/cancel、HMAC scope 和脱敏证据。
- 实盘资金阶段仍需要独立资金上限、live/small_live per-run authorization 和人工确认；本轮用户说明“模拟账户操作”，因此本实现默认授权范围到 simulation 真实操作。

## 2. 用户场景确认

| ID | 问题 | 用户选择 | 收敛理解 |
|---|---|---|---|
| SGQ-001 | “完全具备进入模型盘和实盘的入口条件”按哪种范围执行？ | 含小额实盘；用户备注：“我会在模拟账户的操作，你需要完成所有真实的操作。” | 本轮实现按模拟账户真实操作授权推进：允许完成真实 QMT/gateway runtime、账户只读、simulation submit/cancel 的入口实现与验证路径；不自动扩展到真实资金 live/small_live。 |

## 3. 本轮实现切片

| 层 | 文件 | 结果 |
|---|---|---|
| Gateway contract | `trading/qmt_gateway_contracts.py` | 新增 simulation submit/cancel endpoint 常量、请求对象、脱敏 operation payload 和 allowed result builder。 |
| Runtime gateway | `trading/qmt_runtime.py` | 新增 HMAC scope、HTTP route、`XtQuantRuntimeAdapter.submit_simulation_order()` 与 `cancel_simulation_order()`，调用 `order_stock` / `cancel_order_stock`。 |
| C-side client | `trading/qmt_client.py` | 新增 `QmtClientConfig.allow_simulation_transport`，默认 false；显式 true 时 simulation submit/cancel 走 REST transport。 |
| Runner orchestration | `trading/strategy_runner/simulation_activation.py` | 新增目标组合到 simulation order 的受控激活入口，串联 stage gate、pretrade risk、OMS intent 和 gateway submit。 |
| Public Python API | `trading/strategy_runner/__init__.py` | 导出 simulation activation API。 |
| Hygiene gate | `scripts/check_process_artifact_hygiene.py` / `tests/test_cr132_process_artifact_hygiene.py` | 新增 `current_runner_simulation_entry_asset` 分类，按新目标包识别本轮 runner 模拟盘入口资产。 |

## 4. 验收边界

本轮完成：

- 模型盘目标组合可以转换为 order intent。
- simulation stage gate 缺证据时阻断。
- pretrade risk 失败时阻断且不触达 gateway。
- simulation submit/cancel 支持真实 XtQuant adapter 方法调用。
- runtime response 只输出脱敏 broker order ref，不输出账号、证券代码、原始订单号或 secret。
- runner simulation activation result 只输出稳定 symbol ref，不输出原始证券代码。
- process artifact hygiene gate 通过，当前无未分类工作区变更。
- 旧 offline runner、默认 client later-gated 行为、stage gate / OMS / risk / endpoint matrix 回归保持通过。

本轮不自动完成：

- 真实资金 live/small_live submit/cancel。
- 实盘资金上限、实盘账户授权、live 事故演练和资金回滚确认。
- 在当前 Linux/CI 环境直接启动真实 MiniQMT/XtQuant；真实 runtime 需在交易机由用户显式执行。

## 5. 下一步

推荐下一步继续同一目标包：

1. 增加 runtime CLI / runbook 示例，把 `QmtClientConfig(allow_simulation_transport=True)` 与 gateway HMAC provider 串起来。
2. 在交易机用模拟账户执行一次 redacted smoke：health -> positions -> simulation submit -> simulation cancel。
3. 将 smoke 的脱敏摘要写入 process evidence。
4. 若用户要进入真实资金 live/small_live，再开启独立 high-risk authorization gate。
