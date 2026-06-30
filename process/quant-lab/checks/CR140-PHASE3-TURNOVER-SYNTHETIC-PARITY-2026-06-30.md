# CR140 Phase 3 Turnover Synthetic Parity

## 结论

PASS。Phase 3 已将 `experiments/run_experiment_turnover_factor.py` 的评估侧排序计算切到 `engine.factor_statistics`，并将 abnormal turnover 计算委托到 `engine.factor_calculators.calculate_abnormal_turnover_21_252`。

本验证只证明：新 adapter 与冻结的旧实现逻辑在合成 fixture 上重构等价。

本验证不证明：turnover 因子在真实数据湖上的研究语义已被验证。真实湖只读验证仍需单独授权；本阶段未链接真实 lake、未访问 NAS、未调用 provider、未写 catalog pointer、未进行 QMT/live/runtime 操作。

## 改动范围

- `engine/factor_calculators.py`
  - 新增 `calculate_abnormal_turnover_21_252(...)`，供实验层以旧窗口和 clip 语义复用 engine 计算。
  - 既有 registry 内部 `_calculate_abnormal_turnover_21_252` 继续使用 engine 默认 `min_period_ratio` 语义，不引入实验层 clip。
- `engine/factor_statistics.py`
  - `_quantile_groups` 补齐旧 turnover 实验的低样本 / 全常数语义：样本数不足或有效值唯一时返回空分组。
- `experiments/run_experiment_turnover_factor.py`
  - `calculate_abnormal_turnover` 委托 engine calculator，并保持旧实验 `short_min_periods=15`、`long_min_periods=60`、`clip(0.01, 10.0)`。
  - `run_experiment_a` 改为调用 `single_sort_returns`。
  - `run_experiment_b` 改为调用 `independent_double_sort_returns`。
  - `run_experiment_c` 改为调用 `conditional_double_sort_returns`。
  - 新增 adapter，将 engine 输出列转换回旧实验的 `date/group/mean_return/n_stocks` 与 `date/size_group/turnover_group/mean_return/n_stocks`。
- `tests/test_cr140_turnover_engine_adapter.py`
  - 使用测试内冻结旧逻辑作为 golden oracle，不读取真实数据湖。

## 覆盖的等价点

- abnormal turnover rolling window / min periods / clip 语义。
- `valid_mask` 先应用到 factor、forward return、market cap，再按 `rebalance_dates` 选取调仓日。
- 单变量排序的低样本门槛、分组和输出列。
- 独立双重排序的 size group 与 turnover group 输出列。
- 条件双重排序的 size 内部 turnover 分组。
- 全常数因子不产生有效分组，保持旧实验的 `RuntimeError("无有效分组数据")` 行为。
- 双重排序 spread 继续使用旧实验的 low-minus-high 方向。
- Newey-West t-test 仍保留在实验层；本阶段不把 NW 迁入 engine。

## 验证命令

```bash
uv run --python 3.11 python -m py_compile engine/factor_calculators.py engine/factor_statistics.py experiments/run_experiment_turnover_factor.py tests/test_cr140_turnover_engine_adapter.py
```

结果：PASS。

```bash
uv run --python 3.11 pytest -q -p no:cacheprovider tests/test_cr140_turnover_engine_adapter.py tests/test_factor_statistics.py tests/test_factor_calculators.py tests/test_chapter3_factor_replication.py::test_weighted_sort_double_sort_summary_and_newey_west_are_available
```

结果：`11 passed in 3.69s`。

```bash
git diff --check
```

结果：PASS。

## 剩余边界

- 合成 fixture 覆盖低样本、全常数、mask、单排序、独立双排序、条件双排序和 rolling clip，但不覆盖真实 lake 的所有数据形状。
- 真实数据湖只读验证是后续候选事项，不属于本轮已授权范围。
- 本阶段不触发模拟盘、实盘、QMT、live、broker、runtime 或 catalog publish 链路。

## Post-Close Coverage Supplement

2026-06-30 用户风险评审指出：原始 Phase 3 测试显式覆盖了全常数，但低样本与重复值横截面没有独立用例。已补 source commit：

```text
a1f9026 test(experiments): cover turnover adapter quantile edges
```

新增测试：

- `test_turnover_sorting_adapter_keeps_low_sample_quantile_as_no_data`
- `test_turnover_sorting_adapter_matches_legacy_logic_with_repeated_values`

补强后 targeted regression：

```bash
uv run --python 3.11 pytest -q -p no:cacheprovider tests/test_cr140_turnover_engine_adapter.py tests/test_factor_statistics.py tests/test_factor_calculators.py tests/test_chapter3_factor_replication.py::test_weighted_sort_double_sort_summary_and_newey_west_are_available
```

结果：`13 passed in 3.75s`。

补强后全量 pytest：`46 failed, 1433 passed in 44.38s`，新增失败 0，失败集合仍与 Phase 0 一致。

详细补登见 `process/checks/CR140-POST-CLOSE-RISK-SUPPLEMENT-2026-06-30.md`。
