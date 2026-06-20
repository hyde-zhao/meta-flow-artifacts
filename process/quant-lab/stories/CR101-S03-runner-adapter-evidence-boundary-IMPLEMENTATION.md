---
story_id: CR101-S03-runner-adapter-evidence-boundary
change_id: CR-101
title: Runner adapter and evidence boundary implementation evidence
status: implemented-ready-for-verification
implemented_at: "2026-06-20T09:53:27+08:00"
owner: host-orchestrator
cp5_checkpoint: process/checkpoints/CP5-CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-LLD-BATCH.md
cp6_checkpoint: process/checks/CP6-CR101-S03-runner-adapter-evidence-boundary-CODING-DONE.md
---

# CR101-S03 Runner Adapter and Evidence Boundary Implementation

## 实现对象清单

| 对象 | 路径 | 状态 | 说明 |
|---|---|---|---|
| adapter contract | `trading/strategy_runner/adapters.py` | implemented | `AdapterResult` 携带 target / execution adapter metadata；StrategyPackageAdapter 对 CR101 adapter contract fail closed。 |
| evidence summary | `trading/strategy_runner/evidence.py` | implemented | `EvidenceSummary` 增加 target / adapter / capabilities / sensitive hit 字段。 |
| contract tests | `tests/test_cr091_strategy_runner_contracts.py` | implemented | 增加 adapter boundary、order-write capability 阻断和 evidence metadata 断言。 |

`readonly_gateway.py` 未修改；现有 CR098 回归继续证明 fake transport 默认路径和 order-write endpoint 阻断。

## 设计契约映射

| LLD 验收项 | 实现 | 测试 / 证据 | 状态 |
|---|---|---|---|
| adapter payload 包含 target / adapter 字段 | S01 loader 输出 + `AdapterResult` metadata | `test_strategy_package_adapter_dispatches_loaded_payload` | PASS |
| unknown adapter fail closed | `StrategyPackageAdapter` 校验 `execution_adapter_id` | `test_strategy_package_adapter_rejects_unknown_execution_adapter` | PASS |
| order-write capability 阻断 | `ORDER_WRITE_EXECUTION_CAPABILITIES` | `test_strategy_package_adapter_rejects_order_write_capability` | PASS |
| evidence 表达双边界 | `EvidenceSummary.delivery_target_id` / `execution_adapter_id` / `execution_adapter_capabilities` | `test_evidence_records_cr101_target_and_adapter_boundary` | PASS |
| sensitive field hits 为 0 | `sensitive_field_hits=0` 且 `assert_redacted()` 保持 fail closed | evidence tests | PASS |
| readonly gateway allowlist 不扩大 | 未修改 allowlist，复跑 CR098 | `test_cr098_readonly_gateway_still_blocks_order_like_endpoints` | PASS |
| 缺 runtime authorization_ref 阻断真实 client path | 未修改 CR098 gate | `test_cr098_qmt_client_path_requires_explicit_runtime_authorization` | PASS |

## 验证命令

| 命令 | 结果 | 说明 |
|---|---|---|
| `uv run --python 3.11 pytest -q tests/test_cr091_strategy_runner_contracts.py tests/test_cr100_package_exchange.py tests/test_cr098_runner_readonly_integration.py` | PASS, `41 passed in 0.39s` | S01-S03 + CR098 readonly 回归。 |
| `python3 -m py_compile ... adapters.py evidence.py ...` | PASS | 语法检查。 |
| `git diff --check -- trading/strategy_runner/adapters.py trading/strategy_runner/evidence.py tests/test_cr091_strategy_runner_contracts.py` | PASS | 无空白错误。 |

## 不授权边界

本实现没有新增 SDK import，没有启动或连接 QMT/MiniQMT/XtQuant/gateway runtime，没有读取账户、持仓、委托、成交、原始日志或凭据，没有执行 simulation/live、交易、NAS 或 publish。

## 后续交接

S03 可进入离线 CP7 verification；S04 只有在 S02 与 S03 均 CP7 PASS 后解锁。
