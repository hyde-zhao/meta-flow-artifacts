# CR140 Post-Close Risk Supplement

## 结论

PASS_WITH_RISK_ACCEPTED。用户对 CR140 关闭结论的二次评审指出 5 个遗漏风险，其中 C 影响 Phase 3 等价性证据强度，已用新增测试补强；A/B/D/E 已补登为审计记录或 follow-up candidate。

本补登不重开 CR140，不授权真实 lake、NAS、provider、lake/catalog write、QMT/live/runtime、simulation/trading 或 Git remote write。

## 遗漏风险处理表

| ID | 遗漏风险 | 处理方式 | 当前结论 |
|---|---|---|---|
| RISK-CR140-A-SCOPE-EXPANSION | Phase 1 从计划中的 3 个文件扩到 5 个文件，执行时未显式登记范围修订 | 在本文件和 CP8 post-close review 补登范围修订记录 | 接受。扩张理由是 `lake_input_contract.py` 与 `training_snapshot_contract.py` 同属 CR139 非实验契约 helper，一并归位避免半截整改。 |
| RISK-CR140-B-HELPER-EQUIVALENCE-EVIDENCE | Phase 2 helper 等价性证据不够显眼，grep 仍能看到 `_markdown_table` / `_format_value` / `_resolve_date_range` | 补充说明：08/09/10/12 保留的是薄包装函数，用于传入本地 `PERCENT_FIELDS`；旧本地函数体已删除；13 的 table/value helper 因非等价保留 | 接受。`CR140-PHASE2-HELPER-CONSOLIDATION` 已有等价 diff 表，本文件补充 grep 命中解释。 |
| RISK-CR140-C-QUANTILE-EDGE-COVERAGE | Phase 3 adapter 测试缺少低样本和重复值横截面显式用例 | 新增 `test_turnover_sorting_adapter_keeps_low_sample_quantile_as_no_data` 与 `test_turnover_sorting_adapter_matches_legacy_logic_with_repeated_values` | 已补强。targeted regression 从 `11 passed` 扩展为 `13 passed`。 |
| RISK-CR140-D-BASELINE-DEBT-OWNERSHIP | 46 条红基线债只作为风险口头记录，未进入 follow-up 台账 | 新增 `process/changes/CR-140-FOLLOW-UP-TRACKING-2026-06-30.md`，登记 `FU-CR140-003` 红基线债 triage candidate | 已补登。正式 CR 不预创建，等待用户选择。 |
| RISK-CR140-E-LOCAL-COMMITS-NOT-PUSHED | source repo 本地 commit 未 push，存在本地工作丢失风险 | 登记 `FU-CR140-004` remote persistence gate candidate；不执行 push，因为 Git remote write 未授权 | 已补登。当前 source branch ahead 7，artifact main ahead 8；需要用户单独授权 remote write。 |

## Phase 1 范围修订记录

原计划核心迁移项是 reporting、production current truth rerun、data lake readiness audit 三类文件。执行中纳入了：

- `experiments/lake_input_contract.py` -> `engine/experiment_lake_input_contract.py`
- `experiments/training_snapshot_contract.py` -> `engine/training_snapshot_contract.py`

修订理由：

- 两者是 CR139 引入的 lake / training 契约 helper，不是实验入口。
- 若不一并归位，`experiments/` 顶层仍保留非实验契约工具，Phase 1 目标会半截完成。
- 迁移后目标更贴合用户端到端视角评审中的“experiments/ 只放实验入口”要求。

审计结论：这是执行范围扩张，但属于同一 CR140 目标内的更完整归位；已补登，不重开 CR140。

## Phase 2 Helper 证据补充

grep 命中 `_markdown_table` / `_format_value` / `_resolve_date_range` 的原因：

- 08/09/10/12 保留的是 thin wrapper，调用 `engine.experiment_report_helpers` 的共享实现，并显式传入本地 `PERCENT_FIELDS`。
- `_resolve_date_range` 在 08/09/10/12/13 中已经从共享 helper import，不再保留旧本地函数体。
- 13 的 `_markdown_table` / `_format_value` 因中文 `对比维度`、`PERCENT_DIMENSIONS` 和 `—` 处理非等价，按 Phase 2 证据保留。

当前结论：

- `experiments/run_experiment_08.py`：本地 table/value 旧函数体已删除，只保留 wrapper。
- `experiments/run_experiment_09.py`：本地 table/value 旧函数体已删除，只保留 wrapper。
- `experiments/run_experiment_10.py`：本地 table/value 旧函数体已删除，只保留 wrapper。
- `experiments/run_experiment_12.py`：本地 table/value 旧函数体已删除，只保留 wrapper；date range 一参语义由共享默认参数保持。
- `experiments/run_experiment_13.py`：table/value 非等价，保留本地实现；date range 收敛到共享 helper。

## Phase 3 Adapter 边界补强

新增 source commit：

```text
a1f9026 test(experiments): cover turnover adapter quantile edges
```

新增测试：

- `test_turnover_sorting_adapter_keeps_low_sample_quantile_as_no_data`
- `test_turnover_sorting_adapter_matches_legacy_logic_with_repeated_values`

验证命令：

```bash
uv run --python 3.11 pytest -q -p no:cacheprovider tests/test_cr140_turnover_engine_adapter.py tests/test_factor_statistics.py tests/test_factor_calculators.py tests/test_chapter3_factor_replication.py::test_weighted_sort_double_sort_summary_and_newey_west_are_available
```

结果：

```text
13 passed in 3.75s
```

全量验证：

```bash
uv run --python 3.11 pytest -q --tb=no -p no:cacheprovider
```

结果：

```text
46 failed, 1433 passed in 44.38s
```

对比 Phase 0：

- 新增失败：0
- 保持失败：46
- 新增通过：5，其中 3 个来自原 CR140 adapter 测试，2 个来自本次低样本 / 重复值边界补强。

## 后续候选归属

后续候选已登记在 `process/changes/CR-140-FOLLOW-UP-TRACKING-2026-06-30.md`：

- `FU-CR140-001`：真实 lake readonly turnover semantic validation。
- `FU-CR140-002`：剩余 experiments engine convergence。
- `FU-CR140-003`：red baseline debt triage and closure。
- `FU-CR140-004`：local commits remote persistence gate。

## 不授权范围

- 不授权真实 lake 只读验证。
- 不授权 NAS 访问。
- 不授权 provider fetch。
- 不授权 lake write。
- 不授权 catalog pointer write。
- 不授权 QMT/MiniQMT/xtquant/gateway runtime。
- 不授权 simulation/live/trading。
- 不授权 Git remote write。
