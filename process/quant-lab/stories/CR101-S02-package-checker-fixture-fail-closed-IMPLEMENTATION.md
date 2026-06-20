---
story_id: CR101-S02-package-checker-fixture-fail-closed
change_id: CR-101
title: Package checker and fixture fail-closed implementation evidence
status: implemented-ready-for-verification
implemented_at: "2026-06-20T09:53:27+08:00"
owner: host-orchestrator
cp5_checkpoint: process/checkpoints/CP5-CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-LLD-BATCH.md
cp6_checkpoint: process/checks/CP6-CR101-S02-package-checker-fixture-fail-closed-CODING-DONE.md
---

# CR101-S02 Package Checker and Fixture Fail-Closed Implementation

## 实现对象清单

| 对象 | 路径 | 状态 | 说明 |
|---|---|---|---|
| package exchange checker | `trading/strategy_runner/package_exchange.py` | implemented | 支持 CR101 manifest schema，复用 S01 manifest validator，并保留 CR100 legacy schema 兼容。 |
| package exchange tests | `tests/test_cr100_package_exchange.py` | implemented | 正向 fixture 改为 CR101 `qmt_terminal_direct` + `miniqmt_gateway_readonly`；新增 fail-closed 负向矩阵。 |

## 设计契约映射

| LLD 验收项 | 实现 | 测试 / 证据 | 状态 |
|---|---|---|---|
| schema fail closed | `SUPPORTED_MANIFEST_SCHEMA_VERSIONS` | 既有 schema mismatch 路径 | PASS |
| target missing / legacy target blocked | `_validate_cr101_manifest_shape()` 调用 `validate_runner_manifest()` | `test_cr101_package_checker_rejects_missing_target`、`test_cr101_package_checker_rejects_legacy_miniqmt_runner_delivery_target` | PASS |
| adapter missing / readonly-only | CR101 execution adapter 校验 | `test_cr101_package_checker_rejects_missing_adapter` | PASS |
| permission nonfalse | `permissions` false flags 校验 | `test_cr101_package_checker_rejects_permission_nonfalse` | PASS |
| checksum mismatch | manifest `hashes` 校验 | `test_validate_package_rejects_checksum_drift` | PASS |
| path escape | `_require_file_under()` + S01 relative path 校验 | `test_cr101_package_checker_rejects_path_escape` | PASS |
| sensitive filename | 扩展 `.env` / secret / credential / token / account / raw order / raw position / qmt log 文件名扫描 | `test_cr101_package_checker_rejects_sensitive_filename` | PASS |
| forbidden counter nonzero | `forbidden_operation_counters` 聚合校验 | `test_cr101_package_checker_rejects_forbidden_counter` | PASS |

## 验证命令

| 命令 | 结果 | 说明 |
|---|---|---|
| `uv run --python 3.11 pytest -q tests/test_cr100_package_exchange.py` | PASS, `14 passed in 0.27s` | S02 聚焦测试。 |
| `uv run --python 3.11 pytest -q tests/test_cr091_strategy_runner_contracts.py tests/test_cr100_package_exchange.py tests/test_cr098_runner_readonly_integration.py` | PASS, `41 passed in 0.39s` | S01/S02/S03 相关回归。 |
| `python3 -m py_compile ... package_exchange.py ... test_cr100_package_exchange.py ...` | PASS | 语法检查。 |
| `git diff --check -- trading/strategy_runner/package_exchange.py tests/test_cr100_package_exchange.py` | PASS | 无空白错误。 |

## 不授权边界

本实现仅使用本地临时目录和 fake exchange marker。未访问真实 NAS，未 mount/list/read/write/copy/publish/pull/delete，未读取凭据、账户、持仓、委托、成交或原始日志，未启动 QMT/MiniQMT/XtQuant/gateway runtime，未执行 simulation/live、交易或 publish。

## 后续交接

S02 可进入离线 CP7 verification；S04 仍需等待 S02 与 S03 均 CP7 PASS。
