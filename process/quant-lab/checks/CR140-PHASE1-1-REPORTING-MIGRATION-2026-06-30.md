# CR140 Phase 1.1 Reporting Helper Migration

## Scope

- 迁移 `experiments/reporting.py` 到 `engine/research_reporting.py`。
- 更新实验入口与测试中的导入：
  - `experiments/run_experiment_14.py`
  - `experiments/run_experiment_15_factor_framework.py`
  - `tests/test_cr008_research_input_metadata.py`
  - `tests/test_cr010_consumer_boundary.py`
  - `tests/test_cr010_experiments_realism_metadata.py`
  - `tests/test_cr013_unsupported_register_claim_boundary.py`
  - `tests/test_cr014_research_consumer_boundary.py`

## Boundary

- 未访问真实数据湖。
- 未访问 NAS。
- 未触发 provider fetch。
- 未写 lake / catalog pointer。
- 未触发 QMT / MiniQMT / xtquant / gateway / simulation / live / trading runtime。

## Commands

| 命令 | 结果 | 说明 |
|---|---:|---|
| `git mv experiments/reporting.py engine/research_reporting.py` | PASS | 保留文件历史。 |
| `uv run --python 3.11 pytest -q -p no:cacheprovider tests/test_cr008_research_input_metadata.py tests/test_cr010_consumer_boundary.py tests/test_cr010_experiments_realism_metadata.py tests/test_cr013_unsupported_register_claim_boundary.py tests/test_cr014_research_consumer_boundary.py` | `5 failed, 26 passed` | 5 个失败均属于 Phase 0 已登记基线失败。 |
| `uv run --python 3.11 python -m py_compile engine/research_reporting.py experiments/run_experiment_14.py experiments/run_experiment_15_factor_framework.py tests/test_cr008_research_input_metadata.py tests/test_cr010_consumer_boundary.py tests/test_cr010_experiments_realism_metadata.py tests/test_cr013_unsupported_register_claim_boundary.py tests/test_cr014_research_consumer_boundary.py` | PASS | 无语法错误。 |
| `git diff --check -- engine/research_reporting.py experiments/run_experiment_14.py experiments/run_experiment_15_factor_framework.py tests/test_cr008_research_input_metadata.py tests/test_cr010_consumer_boundary.py tests/test_cr010_experiments_realism_metadata.py tests/test_cr013_unsupported_register_claim_boundary.py tests/test_cr014_research_consumer_boundary.py` | PASS | 无 whitespace error。 |
| `rg -n "experiments\.reporting\|experiments/reporting\.py" .` | PASS | 当前源码与测试无旧 import / 旧路径引用；命中仅历史 process 文档和 CR 影响面描述。 |

## Baseline Comparison

新增失败：无。

保持失败：

- `tests/test_cr008_research_input_metadata.py::test_experiment_cli_data_dir_must_be_explicit`
- `tests/test_cr008_research_input_metadata.py::test_no_connector_runtime_storage_imports_or_credentials_in_target_files`
- `tests/test_cr013_unsupported_register_claim_boundary.py::test_unsupported_register_exact_rows_and_excluded_denominator`
- `tests/test_cr013_unsupported_register_claim_boundary.py::test_claim_boundary_summary_merges_s01_s02_and_excludes_denominator`
- `tests/test_cr013_unsupported_register_claim_boundary.py::test_docs_and_report_show_supported_research_only_unsupported_blocked_boundaries`

修复的基线失败：未在本切片验证集中确认。

## Decision

Phase 1.1 可以继续进入 Phase 1.2。该切片只改变 reporting helper 的归属和导入路径，不改变报告生成语义。
