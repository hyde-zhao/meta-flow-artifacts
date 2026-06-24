# Rollback CR137

## 回滚目标

回退到 `pre-CR137 runner run registry state`。

## 回滚范围

| 对象 | 处理 |
|---|---|
| `trading/strategy_runner/run_registry.py` | 删除新增 registry 合同。 |
| `trading/strategy_runner/run_spec.py` | 移除 `run_registry_output_path`。 |
| `trading/strategy_runner/runner.py` | 移除 registry append 调用。 |
| `trading/strategy_runner/cli.py` | 移除 registry CLI 参数。 |
| `trading/strategy_runner/__init__.py` | 移除 registry public exports。 |
| `tests/test_cr137_runner_run_registry.py` | 删除 CR137 专项测试。 |
| `process/docs/features/strategy-runner-core/*` | 回退 v1.5 修订到 v1.4。 |

## 验证

回滚后运行 CR128/CR133/CR134/CR135/CR136 runner regression，确保既有 single-run evidence index、bundle、inspect/replay 和 validation 仍通过。

## 不可回滚项

无外部状态、数据库、服务或真实 runtime 变更；无不可逆迁移。
