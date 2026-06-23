# CP6 CR133 Runner Spec CLI Implementation Done

## Entry Criteria

- CR132 hygiene gate 已关闭 READY，runner development gate allowed。
- Strategy Runner Core 是 runner 长期 feature authority。
- 授权范围保持 offline-only，不触碰 runtime/NAS/QMT/credential/provider/lake/catalog/trading。

## Checklist

| 项目 | 结果 | 证据 |
|---|---|---|
| RunSpec 文件入口 | PASS | `trading/strategy_runner/run_spec.py` |
| offline CLI | PASS | `trading/strategy_runner/cli.py` |
| spec file helper API | PASS | `trading/strategy_runner/runner.py`, `trading/strategy_runner/__init__.py` |
| result output 父目录创建 | PASS | `trading/strategy_runner/result.py` |
| CR133 自动化测试 | PASS | `tests/test_cr133_runner_spec_cli.py` |
| feature docs v1.1 | PASS | `process/docs/features/strategy-runner-core/*` |

## Exit Criteria

- YAML/JSON RunSpec 能驱动 fixture package。
- CLI JSON 输出可被机器读取。
- invalid spec fail-closed。
- 外部权限 flags 仍被拒绝。

## Deliverables

- `trading/strategy_runner/run_spec.py`
- `trading/strategy_runner/runner.py`
- `trading/strategy_runner/cli.py`
- `trading/strategy_runner/result.py`
- `tests/test_cr133_runner_spec_cli.py`
- `process/docs/features/strategy-runner-core/DESIGN.md`
- `process/docs/features/strategy-runner-core/TASKS.md`
- `process/docs/features/strategy-runner-core/TEST-PLAN.md`
