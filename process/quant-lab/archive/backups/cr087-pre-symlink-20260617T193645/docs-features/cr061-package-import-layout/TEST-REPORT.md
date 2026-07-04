# CR061 Batch B Test Report

## 测试范围

| 范围 | 状态 | 说明 |
|---|---|---|
| package metadata | covered | `pyproject.toml` project name asserted as `quant-lab`。 |
| lock metadata | covered | `uv lock` 成功；lock diff 为 project package rename。 |
| legacy roots | covered | `engine` / `market_data` / `strategies` / `trading` import smoke。 |
| `quant_lab` namespace | covered | namespace and alias imports smoke。 |
| external/runtime | excluded | 按 CP5 不授权项禁止。 |
| full pytest | not run | 当前验证范围为 offline subset；记录为剩余风险。 |

## 测试结果

| Test ID | 命令 / 检查 | 结果 | 证据 |
|---|---|---|---|
| TR-CR061B-001 | `uv lock` | PASS | `Resolved 149 packages... Removed local-backtest v0.1.0; Added quant-lab v0.1.0` |
| TR-CR061B-002 | `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr061_package_import_layout.py` | PASS | `4 passed in 0.03s` |
| TR-CR061B-003 | `PYTHONPYCACHEPREFIX=/tmp/cr061-batch-b-pycompile uv run --python 3.11 python -m py_compile ...` | PASS | exit code 0 |
| TR-CR061B-004 | `uv run --python 3.11 python -c pyproject-name-check` | PASS | `PASS pyproject name quant-lab` |
| TR-CR061B-005 | `git diff --check` | PASS | exit code 0 |

## 覆盖缺口

| 缺口 | 严重度 | 原因 | 处理 |
|---|---|---|---|
| full pytest 未运行 | MEDIUM | 当前门禁授权的是离线 import smoke 和必要 subset；全量可能触发外部依赖。 | CP8 风险接受或后续单独扩大离线回归。 |
| 没有验证 `quant_lab.<root>.<submodule>` 深层路径 | LOW | 当前 alias 设计提供 package-level module alias；深层 import rewrite 未授权。 | bulk relayout 前另行设计。 |

## 阶段结论

测试结论：`PASS_WITH_RISK`
