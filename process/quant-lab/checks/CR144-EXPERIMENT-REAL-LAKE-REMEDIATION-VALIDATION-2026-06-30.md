# CR144 Experiment Real Lake Remediation Validation

## 结论

CR144 已完成 E4 真实 lake experiment smoke 和由 smoke 触发的代码整改。未发现需要写入真实 lake canonical / quality / catalog / pointer 的数据缺口；本轮真实数据湖操作为读取验证，研究产物写入本机研究目录。

## 授权与实际操作

| 操作类型 | 授权状态 | 实际执行 | 说明 |
|---|---:|---:|---|
| real lake read | authorized | yes | 读取 `MARKET_DATA_LAKE_ROOT=/home/hyde/data/quant-lab/data-lake`。 |
| research report write | authorized | yes | 写入 `/home/hyde/data/quant-lab/research/reports/cr144_e4/`。 |
| research run write | authorized | yes | turnover 过程文档写入 `/home/hyde/data/quant-lab/research/runs/cr144_e4/turnover/`。 |
| canonical / quality / catalog lake write | authorized if needed | no | 代码合同修复后无需改湖。 |
| provider fetch | conditional | no | 未触发 provider 缺口。 |
| catalog pointer publish | authorized if needed | no | 无 publish 需求。 |
| QMT / simulation / live / trading | not in default scope | no | 未执行。 |

## 发现与整改

| ID | 分类 | 现象 | 根因 | 整改 |
|---|---|---|---|---|
| D-CR144-001 | 字段语义 | `index_members` real lake reader 对 experiment 输入报缺 `symbol` | canonical 合同字段为 `con_code`，experiment 期望 `symbol` | `engine.experiment_lake_input_contract` 改为按 `trade_date + con_code` PIT 去重，并将 `con_code` 规范化为 `symbol`。 |
| D-CR144-002 | 数据形状 | experiment 16 real lake smoke 生成空动量因子面板 | experiment lake reader 对 `prices` 只按 `symbol` 去重，压缩掉时间序列 | `prices` PIT key 改为 `trade_date + symbol`，保留日度面板。 |
| D-CR144-003 | 字段语义 | `trade_calendar` / `index_members` mixed `available_at` 字符串被判不可解析 | pandas 对混合小数秒、时区格式整列推断失败 | `market_data.readers` 的 decision/as-of 解析改为 `utc=True, format="mixed"`。 |
| D-CR144-004 | 输出路径 | turnover smoke 在 CR030 附属产物阶段尝试写 `/mnt/quant-lab-primary` | turnover 脚本部分输出路径硬编码 `research_report_path` | 新增 `--output-root` / `--process-dir`，CR030 附属产物和过程文档跟随显式路径。 |

## 评审后补改

| ID | 评审级别 | 问题 | 处理结果 |
|---|---|---|---|
| F1 | medium | `index_members` PIT key 使用 `trade_date + con_code`，会把同一股票同日不同 `index_code` 的成员关系折叠掉。 | 已改为复用 canonical `DATASET_KEY_COLUMNS[DATASET_INDEX_MEMBERS] = trade_date + index_code + con_code`；新增冲突 fixture，验证 HS300/ZZ500 同股同日成员关系均保留。 |
| F2 | low | turnover 只传 `--output-root` 时，`PROCESS_DIR` 仍可能落到默认 NAS run path。 | 新增 `resolve_output_roots()`；当只传 `--output-root` 时，过程目录派生为 `<output-root>/<run-id>/process`。 |
| F3 | low | `read_panel_as_of` 在 keys 已含 `trade_date` 时重复插入排序列。 | 增加 `trade_date not in keys` 守卫，避免重复 sort key。 |
| F4 | low | experiment 15 执行路径保留不可达 legacy `data_dir` 回退。 | `_frames_from_explicit_inputs()` 改为直接消费 `load_experiment_lake_frames()`；保留 `load_local_frames()` 仅供显式 fixture 测试构造，保留 `require_explicit_data_dir()` 作为 legacy boundary guard 测试对象。 |

## Smoke 结果

| 入口 | 命令范围 | 结果 | 关键产物 |
|---|---|---|---|
| experiment 15 | `2025-01-01..2025-06-30`，3 个基础因子 | PASS | `/home/hyde/data/quant-lab/research/reports/cr144_e4/experiment_15/factor_panel.parquet`，123693 rows |
| experiment 16 | `2025-01-01..2025-06-30`，`momentum_20d` / `momentum_60d` | PASS | `/home/hyde/data/quant-lab/research/reports/cr144_e4/experiment_16/ic_summary.csv`，8 rows |
| experiment 17_21 | `2025-01-01..2025-06-30`，`max_symbols=20` | PASS | `/home/hyde/data/quant-lab/research/reports/cr144_e4/experiment_17_21/smoke-202506/factor_strategy_report.md` |
| experiment 23_29 | `2025-01-01..2026-06-26`，`max_symbols=20`，最小模型参数 | PASS | `/home/hyde/data/quant-lab/research/reports/cr144_e4/experiment_23_29/experiment_29_ml_strategy_and_summary/stage4_summary.md` |
| turnover | `2025-01-01..2025-06-30`，`warmup_start=2024-01-02`，3 组 smoke | PASS | `/home/hyde/data/quant-lab/research/reports/cr144_e4/turnover/cr144-e4-turnover-smoke-20250101-20250630/turnover_factor_replication_report.md` |

## 验证命令

| 命令 | 结果 |
|---|---|
| `uv run --python 3.11 pytest -q tests/test_cr139_ml_lake_onboard_no_bypass.py tests/test_cr139_decision_time_lookahead_gate.py` | `10 passed` |
| `uv run --python 3.11 pytest -q tests/test_cr139_ml_lake_onboard_no_bypass.py tests/test_experiment_15_factor_framework.py tests/test_experiment_16_momentum_factor.py` | `10 passed` |
| `uv run --python 3.11 pytest -q tests/test_cr008_factor_auxiliary_data_contract.py::test_t11_experiment_15_schema_report_and_summary_include_auxiliary_contract tests/test_cr139_ml_lake_onboard_no_bypass.py tests/test_cr139_pit_as_of_reader.py tests/test_cr139_decision_time_lookahead_gate.py tests/test_cr140_turnover_engine_adapter.py tests/test_experiment_15_factor_framework.py tests/test_experiment_16_momentum_factor.py tests/test_experiment_17_21_factor_suite.py` | `30 passed` |
| E4 real lake reader precheck for `prices` / `index_members` / `trade_calendar` | all `available` |

## 剩余风险

- turnover 图表生成时当前环境缺少可用 CJK 字体，PNG 中文 glyph 有 matplotlib warning；不影响 CSV / parquet / Markdown 研究数据。
- `/mnt/quant-lab-primary` 当前不可写，本轮使用 `/home/hyde/data/quant-lab/research` 作为研究产物输出根。
- 本轮 smoke 为 bounded research validation，不是 production admission，不授权 simulation/live/trading。
