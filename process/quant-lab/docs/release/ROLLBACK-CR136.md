# Rollback CR136

## 回滚目标

回退到 `pre-CR136 runner bundle validation state`，即保留 CR135 bundle writer / inspect / replay，但移除 CR136 的 validation API、CLI validate 和相关测试/文档。

## 回滚范围

- `trading/strategy_runner/artifact_bundle.py`
- `trading/strategy_runner/cli.py`
- `trading/strategy_runner/__init__.py`
- `tests/test_cr136_runner_bundle_validation.py`
- `scripts/check_process_artifact_hygiene.py`
- `tests/test_cr132_process_artifact_hygiene.py`
- `process/docs/features/strategy-runner-core/*`

## 验证

回滚后至少运行 CR135 runner regression 和 process artifact hygiene，确认 CR135 bundle inspect/replay 仍通过。

## 不可回滚项

无外部状态、无迁移、无真实运行或交易动作。
