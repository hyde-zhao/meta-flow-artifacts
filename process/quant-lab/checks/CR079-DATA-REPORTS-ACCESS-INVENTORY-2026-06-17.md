---
check_id: "CR079-DATA-REPORTS-ACCESS-INVENTORY-2026-06-17"
change_id: "CR-079"
checkpoint_type: "CP6"
status: "PASS_WITH_RISK"
created_at: "2026-06-17T06:12:28+08:00"
owner: "host-orchestrator"
authorization_gate: "CP2/CP3/CP5-CR079 approved"
---

# CR079 Data/reports Access Inventory

## Entry Criteria

| 条目 | 结果 | 证据 |
|---|---|---|
| CP2/CP3/CP5 人工门禁已批准 | PASS | 用户回复“好的，先完成CR079的迁移”；三份 checkpoint 已回填 approved。 |
| CR tracking preflight | PASS | `uv run --python 3.11 python scripts/check_cr_tracking_consistency.py --project-root .` |
| Human-gate preflight | PASS | CP2/CP3/CP5 三份 `scripts/check_human_gate_decision_brief.py` 均 PASS，decision_count=5。 |
| 执行范围限定 | PASS | 只执行 metadata-only 只读清点。 |

## 执行命令说明

执行方式为本地 metadata-only Python 清点脚本：

- 使用 `os.scandir()` / `os.walk()` / `os.lstat()` 读取目录元数据。
- 不调用 `open()`。
- 不读取文件内容。
- 不解析 parquet/csv/json。
- 不计算内容 hash。
- 不复制、恢复、移动、重命名或删除任何文件。
- 不读取 `.env` 或凭据。

## Inventory Summary

| 路径 | 存在 | 顶层项数 | 目录数 | 文件数 | symlink | other | 总字节数 | mtime 范围 | 风险命名命中 | 结论 |
|---|---:|---:|---:|---:|---:|---:|---:|---|---:|---|
| `/home/hyde/workspace/quant-lab/reports` | false | 0 | 0 | 0 | 0 | 0 | 0 | N/A | 0 | target 缺失 |
| `/home/hyde/workspace/quant-lab/data` | false | 0 | 0 | 0 | 0 | 0 | 0 | N/A | 0 | target 缺失 |
| `/home/hyde/workspace/local_backtest/reports` | true | 52 | 126 | 772 | 0 | 0 | 4529789958 | 2026-05-14T21:59:43..2026-06-13T23:18:23 | 24 | legacy retained assets 存在 |
| `/home/hyde/workspace/local_backtest/data` | true | 6 | 10 | 8 | 0 | 0 | 61723552 | 2026-05-14T21:59:43..2026-05-18T20:00:50 | 4 | legacy retained assets 存在 |

## 顶层项摘要

> 仅记录有限顶层元数据；疑似敏感命名已脱敏为 `[REDACTED-RISK-NAME]`。本节不包含文件内容、样本或深层完整清单。

| 路径 | 顶层摘要 |
|---|---|
| `/home/hyde/workspace/quant-lab/reports` | N/A，路径不存在。 |
| `/home/hyde/workspace/quant-lab/data` | N/A，路径不存在。 |
| `/home/hyde/workspace/local_backtest/reports` | `chapter3_factor_panel`, `chapter4_factor_models`, `chapter5_anomalies`, `chapter6_factor_robustness`, `chapter7_factor_practice`, `charts`, `cr029_stage6_precheck`, `data_lake_readiness_2020_2024`, `data_lake_readiness_2020_2024_cr013`, `data_lake_readiness_limited_2025_2026`, `experiment_08`..`experiment_36_stage5_summary`, `experiments_06_07`, `factor_evaluation`, `factor_research`, `multifactor_strategy_candidates`, `previous_data`, `production_current_truth`, `research_catalog` 等。 |
| `/home/hyde/workspace/local_backtest/data` | `manifests`, `raw`, `.gitkeep`, `[REDACTED-RISK-NAME]`, `index_members.parquet`, `prices.parquet`。 |

## Operation Counts

| 操作 | 计数 | 结果 |
|---|---:|---|
| file_content_read | 0 | PASS |
| sample_print | 0 | PASS |
| parquet_csv_json_parse | 0 | PASS |
| content_hash | 0 | PASS |
| copy_restore_move_delete | 0 | PASS |
| credential_or_env_read | 0 | PASS |
| provider_fetch | 0 | PASS |
| lake_write | 0 | PASS |
| catalog_publish | 0 | PASS |
| remote_git_write | 0 | PASS |
| qmt_miniqmt_runtime | 0 | PASS |
| old_root_retirement | 0 | PASS |

## 结果

- 清点动作：PASS。
- readiness 结论：PASS_WITH_RISK。
- 关键事实：target root 缺少 `reports/` 和 `data/`；legacy root 保留 `reports/` 和 `data/`。
- 下一步：执行 CP7 验证；若用户要真正迁移 / 恢复，需把 `CR079-restore-candidate` 转为正式 CR。
