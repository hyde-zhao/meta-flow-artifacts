# CR140 Phase 2 Helper Consolidation

## Scope

- 新增 `engine/experiment_report_helpers.py`：
  - `markdown_table(rows, fields, percent_fields=...)`
  - `format_header(field, percent_fields=...)`
  - `format_value(value, field, percent_fields=...)`
  - `resolve_date_range(data_dir, start_date=None, end_date=None)`
- 收敛实验 06/07、08、09、10、12 的 Markdown table / value formatting 逻辑为共享 helper 包装。
- 收敛实验 06/07、08、09、10、12、13 的 `_resolve_date_range` 本地副本。
- 更新 `docs/components/EXPERIMENTS.md` 实验编号索引。

## Equivalence Diff

| 文件 | `_markdown_table` / `_format_value` | `_resolve_date_range` | 处理 |
|---|---|---|---|
| `experiments/run_experiment_08.py` | 函数体与 06/07 等价，但 `PERCENT_FIELDS` 不同 | 等价 | 删除本地实现，改为共享 helper + 本地 `PERCENT_FIELDS` 包装。 |
| `experiments/run_experiment_09.py` | 函数体与 06/07 等价，但 `PERCENT_FIELDS` 不同 | 等价 | 删除本地实现，改为共享 helper + 本地 `PERCENT_FIELDS` 包装。 |
| `experiments/run_experiment_10.py` | 函数体与 06/07 等价，但 `PERCENT_FIELDS` 不同 | 等价 | 删除本地实现，改为共享 helper + 本地 `PERCENT_FIELDS` 包装。 |
| `experiments/run_experiment_12.py` | 函数体与 06/07 等价，但 `PERCENT_FIELDS` 不同 | 旧签名无 start/end override；共享 helper 默认参数可保持一参调用语义 | 删除本地实现，改为共享 helper + 本地 `PERCENT_FIELDS` 包装。 |
| `experiments/run_experiment_13.py` | 非等价：中文 `对比维度` / `PERCENT_DIMENSIONS` / `—` 处理 | 等价 | 保留本地 Markdown/value helper，只收敛 date range。 |

关键决策：未直接复用 06/07 私有 `_format_value`，因为各实验的 `PERCENT_FIELDS` 集合不同。共享 helper 显式传入 `percent_fields`，避免改变报告百分比列。

## Boundary

- 未访问真实数据湖。
- 未访问 NAS。
- 未触发 provider fetch。
- 未写 lake / catalog pointer。
- 未触发 QMT / MiniQMT / xtquant / gateway / simulation / live / trading runtime。

## Commands

| 命令 | 结果 | 说明 |
|---|---:|---|
| AST helper diff | PASS | 08/09/10 table/value/date body 与 06/07 等价；12 table/value 等价、date 一参语义由共享默认参数保持；13 table/value 非等价并保留。 |
| `uv run --python 3.11 python -m py_compile engine/experiment_report_helpers.py experiments/run_experiment_06_07.py experiments/run_experiment_08.py experiments/run_experiment_09.py experiments/run_experiment_10.py experiments/run_experiment_12.py experiments/run_experiment_13.py` | PASS | 无语法错误。 |
| `uv run --python 3.11 pytest -q -p no:cacheprovider tests/test_story_004_013.py tests/test_market_data_experiment_readers.py tests/test_cr007_experiment_real_benchmark_consumption.py tests/test_market_data_hs300_benchmark.py` | `2 failed, 36 passed` | 2 个 `tests/test_story_004_013.py` 失败属于 Phase 0 已知基线失败。 |
| `uv run --python 3.11 pytest -q -p no:cacheprovider tests/test_market_data_contracts.py::test_market_data_does_not_import_engine_family` | FAIL | Phase 0 已知基线失败。 |
| `uv run --python 3.11 python -c 'from engine.experiment_report_helpers import ...'` | PASS | Markdown 百分比和值格式 smoke 输出符合旧逻辑。 |
| `git diff --check` | PASS | 无 whitespace error。 |

## Baseline Comparison

新增失败：无。

保持失败：

- `tests/test_story_004_013.py::test_story_004_loader_reads_offline_parquet_and_quality_gate`
- `tests/test_story_004_013.py::test_t_logging_minimal_01_cli_diagnostics`
- `tests/test_market_data_contracts.py::test_market_data_does_not_import_engine_family`

修复的基线失败：未在本切片验证集中确认。

## Decision

Phase 2 可以进入提交与全量 baseline comparison。全量 pytest 必须在 source / artifact clean 后执行，避免 CR132 dirty-workspace hygiene 产生临时新增失败。
