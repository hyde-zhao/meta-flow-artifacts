---
story_id: CR101-S01-target-taxonomy-manifest-contract
change_id: CR-101
title: Target taxonomy and manifest contract implementation evidence
status: implemented-ready-for-verification
implemented_at: "2026-06-20T09:40:30+08:00"
owner: host-orchestrator
cp5_checkpoint: process/checkpoints/CP5-CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-LLD-BATCH.md
cp6_checkpoint: process/checks/CP6-CR101-S01-target-taxonomy-manifest-contract-CODING-DONE.md
---

# CR101-S01 Target Taxonomy and Manifest Contract Implementation

## 实现对象清单

| 对象 | 路径 | 状态 | 说明 |
|---|---|---|---|
| manifest loader | `trading/strategy_runner/package_loader.py` | implemented | 新增 CR101 manifest schema、delivery target / execution adapter 常量和校验。 |
| contract tests | `tests/test_cr091_strategy_runner_contracts.py` | implemented | 新增 CR101 正向 / 负向合同测试与 runner core 静态 import 边界测试。 |

未修改 S02 owner 文件 `trading/strategy_runner/package_exchange.py`；未修改 runtime launcher、NAS 脚本、凭据或真实数据路径。

## 设计契约映射

| LLD 验收项 | 实现 | 测试 / 证据 | 状态 |
|---|---|---|---|
| `delivery_targets[]` 与 `execution_adapters[]` 双边界 | `CR101_MANIFEST_SCHEMA_VERSION`、`select_delivery_target()`、`select_execution_adapter()` | `test_cr101_manifest_accepts_qmt_direct_target_and_miniqmt_readonly_adapter` | PASS |
| 当前唯一 implemented delivery target 为 `qmt_terminal_direct` | `DELIVERY_TARGET_QMT_TERMINAL_DIRECT` 与 implemented count 校验 | 正向测试 + `blocked_delivery_target_not_qmt_terminal_direct` 失败路径 | PASS |
| `miniqmt_runner` 不再作为 delivery target | `LEGACY_MINIQMT_RUNNER_TARGET` 阻断 | `test_cr101_manifest_rejects_legacy_miniqmt_runner_delivery_target` | PASS |
| future target 只能作为未实现 slot | `FUTURE_DELIVERY_TARGETS` 且 `implemented=true` fail closed | `test_cr101_manifest_rejects_future_delivery_target_marked_implemented` | PASS |
| MiniQMT 只作为 readonly execution adapter contract | `EXECUTION_ADAPTER_MINIQMT_GATEWAY_READONLY` + readonly capability allowlist | 正向测试断言 adapter payload | PASS |
| 不授权 flags 必须为 false | 沿用 `REQUIRED_FALSE_FLAGS` 与 `forbidden_operations` zero 校验 | 既有 CR091 contract tests | PASS |
| runner core 不直接 import QMT / MiniQMT / XtQuant SDK | `package_loader.py` 无 SDK import | `test_package_loader_has_no_real_qmt_sdk_imports` | PASS |

## 兼容性说明

- 保留旧 `cr091-strategy-runner-package-manifest-v1` 读取能力，`MANIFEST_SCHEMA_VERSION` 仍指向 CR091，避免破坏现有 fixture。
- CR101 作为新增 schema 进入 `SUPPORTED_MANIFEST_SCHEMA_VERSIONS`。
- `StrategyPackage.to_adapter_payload()` 保留旧 `adapter_type` 与 false authorization flags，同时新增 `delivery_target_id`、`execution_adapter_id`、`execution_adapter_capabilities`，供 S02 / S03 消费。
- `not_authorization` 当前保持历史 payload 布尔合同 `true`，未在 S01 中改成对象形态，避免破坏既有 adapter 消费；CR101 的权限边界继续由 false flags 和 forbidden operation counters 校验。

## 验证命令

| 命令 | 结果 | 说明 |
|---|---|---|
| `uv run --python 3.11 pytest -q tests/test_cr091_strategy_runner_contracts.py tests/test_cr100_package_exchange.py tests/test_cr098_runner_readonly_integration.py` | PASS, `31 passed in 0.28s` | S01 合同测试 + CR100 / CR098 相邻回归。 |
| `python3 -m py_compile /home/hyde/workspace/quant-lab/trading/strategy_runner/package_loader.py /home/hyde/workspace/quant-lab/tests/test_cr091_strategy_runner_contracts.py` | PASS | 语法检查。 |
| `uv run --python 3.11 meta-flow workspace check --project-root /home/hyde/workspace/quant-lab` | PASS_WITH_INFO | `process_link_health: ok`；artifact git dirty 为当前过程文件变更。 |
| `uv run --python 3.11 meta-flow check cr-tracking --project-root /home/hyde/workspace/quant-lab --strict-warnings` | PASS | active formal CR 仍为 CR-101。 |

## 不授权边界

本实现未访问 NAS，未读取 `.env`、凭据、账户、资金、持仓、委托、成交或原始日志，未启动 QMT / MiniQMT / XtQuant / gateway runtime，未执行 simulation/live、submit/cancel、buy/sell、provider/lake/catalog publish。

## 后续交接

- CP6 通过后，S01 可进入离线 CP7 verification。
- S02 / S03 依赖 S01 contract，只有在 CP6 记录通过后才允许进入后续 dev-ready 计算；真实系统操作仍需独立 runtime authorization gate。
