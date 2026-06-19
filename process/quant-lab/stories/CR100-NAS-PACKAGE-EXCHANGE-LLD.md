# CR100 NAS Package Exchange LLD

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v1.0 | 2026-06-19 | host-orchestrator | 定义本地 fake exchange 实现、测试和 runbook。 |

## 1. Goal

交付 CR100 offline readiness，不连接真实 NAS。实现应能在本地临时目录中证明 package manifest、hash、approval、fake publish、fake pull、immutable cache 和 active pointer 合同。

## 2. Scope

In scope:

- `trading/strategy_runner/package_exchange.py`
- `scripts/cr100_package_exchange.py`
- `tests/test_cr100_package_exchange.py`
- CR100 HLD / runbook / 质量证据

Out of scope:

- 真实 NAS access/list/read/copy/write/publish/delete
- Windows/Linux `.env`、凭据、账户、资金、持仓、委托、成交、原始日志
- QMT / MiniQMT / XtQuant / gateway / runner runtime
- submit/cancel、buy/sell、simulation/live
- provider/lake/catalog publish

## 3. Data Model

| 对象 | 字段 | 说明 |
|---|---|---|
| `PackageIdentity` | `package_id`, `package_version` | package 唯一标识 |
| `PackageValidation` | `identity`, `manifest`, `checked_files` | manifest/hash 校验结果 |
| `ExchangeOperationResult` | `operation`, `passed`, `errors`, `forbidden_operation_counters` | CLI 可 JSON 输出的结果 |
| `active.json` | `package_id`, `package_version`, `package_path`, `not_authorization` | 本地 cache active pointer |

## 4. File Ownership

| 文件 | 权限 |
|---|---|
| `trading/strategy_runner/package_exchange.py` | 新增 / 主写 |
| `trading/strategy_runner/__init__.py` | 小范围导出 |
| `scripts/cr100_package_exchange.py` | 新增 / 主写 |
| `tests/test_cr100_package_exchange.py` | 新增 / 主写 |
| `docs/qmt/CR100-*` | 新增 / 主写 |
| `process/*CR100*` | 新增 / 主写 |

## 5. Contract Mapping

| 设计契约 | 实现位置 | 验证 |
|---|---|---|
| fake root marker 必须存在 | `_require_fake_exchange_root` | `test_fake_publish_requires_fake_exchange_marker` |
| manifest schema / target / entrypoint / approval / permissions | `_validate_manifest_shape` | `test_validate_package_accepts_manifest_and_hashes` |
| hash mismatch fail closed | `_verify_manifest_hashes` | `test_validate_package_rejects_checksum_drift` |
| forbidden file fail closed | `_reject_forbidden_file_names` | `test_validate_package_rejects_env_or_secret_markers` |
| fake publish 写 index | `fake_publish_package` | `test_fake_publish_pull_and_exchange_check_stay_local` |
| fake pull 写 immutable cache + active pointer | `fake_pull_package` | 同上 |
| CLI 不导入 runtime/network/env | `scripts/cr100_package_exchange.py` | AST test |

## 6. Error Handling

所有错误返回 `blocked_*` 或结构化 `ExchangeOperationResult(passed=false)`。缺 fake marker、缺 index、package 缺失、未批准、hash mismatch、symlink、路径逃逸均 fail closed。

## 7. Test Plan

| 测试 | 目的 |
|---|---|
| `test_validate_package_accepts_manifest_and_hashes` | 成功校验 manifest/hash |
| `test_fake_publish_pull_and_exchange_check_stay_local` | 本地 fake publish/pull/check 成功路径 |
| `test_fake_publish_requires_fake_exchange_marker` | 防误触真实 NAS |
| `test_validate_package_rejects_checksum_drift` | hash drift fail closed |
| `test_validate_package_rejects_unapproved_package` | approval fail closed |
| `test_validate_package_rejects_env_or_secret_markers` | 敏感文件 fail closed |
| `test_cli_does_not_import_runtime_network_or_env_modules` | CLI 边界扫描 |

## 8. Rollback

回滚方式为删除 CR100 新增文件并从 `__init__.py` 移除导出；不会影响已有 CR091/CR089/CR099 runtime 路径。
