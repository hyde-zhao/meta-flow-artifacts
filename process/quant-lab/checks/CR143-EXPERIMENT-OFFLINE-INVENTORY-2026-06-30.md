# CR143 Experiment Offline Inventory

## 结论

本轮盘点确认 CR140/141/142 之后，`experiments/` 顶层只剩 13 个 `run_experiment_*` 实验入口；E3 非 lake 剩余工作集中在 IC/Rank IC 统计合同、实验动态因子归属和后续交接，不需要连接真实数据湖。

## 顶层实验入口

| 入口 | 当前归属判断 | 本轮处理 |
|---|---|---|
| `run_experiment_06_07.py` | 早期实验入口，已复用共享 report helper | 不改 |
| `run_experiment_08.py` | 早期实验入口，已复用共享 report helper | 不改 |
| `run_experiment_09.py` | 早期实验入口，已复用共享 report helper | 不改 |
| `run_experiment_10.py` | 早期实验入口，已复用共享 report helper | 不改 |
| `run_experiment_12.py` | 早期实验入口，已复用共享 report helper | 不改 |
| `run_experiment_13.py` | 保留非等价本地 report helper | 不改，CR140 已记录理由 |
| `run_experiment_14.py` | 数据和 benchmark 实验入口 | 不改 |
| `run_experiment_15_factor_framework.py` | 因子工程基础框架；动态因子定义仍为 experiment-local | 本轮写入 factor ownership 规则 |
| `run_experiment_16_momentum_factor.py` | 动量因子有效性检验；存在本地 IC/Rank IC 汇总 | 本轮迁移到 engine IC 合同 |
| `run_experiment_17_21_factor_suite.py` | 多因子评估套件；CR142 已接 engine 统计合同核心切片 | 不扩大 |
| `run_experiment_23_29_ml_factor_suite.py` | ML 因子套件；CR142 已完成 downstream 回归 | 只作为后续 ML 框架输入 |
| `run_experiment_30_36_stage5.py` | stage5 实验入口 | 不改 |
| `run_experiment_turnover_factor.py` | turnover 实验；CR140 已完成 synthetic parity | E4 真实 lake 验证候选 |

## IC / Rank IC 盘点

| 位置 | 状态 | 本轮处理 |
|---|---|---|
| `engine.factor_statistics` | 原有 sort、long-short、Fama-MacBeth、Newey-West；缺通用 IC/Rank IC | 增加 `calculate_information_coefficient_timeseries`、`summarize_information_coefficient`、`spearman_rank_correlation` |
| `run_experiment_16_momentum_factor.py` | 本地 `calculate_ic_timeseries`、`summarize_ic`、`_spearman_corr` | 保留函数名，内部调用 engine |
| `run_experiment_17_21_factor_suite.py` | 仍有实验报告/策略编排相关私有 helper | 不在本轮扩大，后续按多因子生产框架处理 |

## Factor Definition 盘点

| 类型 | 例子 | 归属 |
|---|---|---|
| engine core | `value_bm`、`momentum_12_1`、`abnormal_turnover_21_252` | 已在 `engine.factor_library` core set，且需要 calculator 可用 |
| experiment-local 动态因子 | `momentum_20d`、`volume_ratio_20d`、`volatility_20d`、`rsi_14`、`macd_diff` | 不进入 core；保留研究入口私有归属 |
| research candidate | 未匹配 core 或实验模式的自定义 alpha | 需要补定义、calculator 和真实数据语义后再决定 |
| future production candidate | 已有长期语义但 calculator 或真实数据语义未稳定 | 不直接生产化 |

## 不授权确认

本轮未连接真实 lake，未执行 NAS/provider/lake write/catalog pointer/runtime/trading 操作。
