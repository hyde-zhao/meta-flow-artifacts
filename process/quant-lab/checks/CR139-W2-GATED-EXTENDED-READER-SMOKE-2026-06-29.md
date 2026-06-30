---
checkpoint_id: "CR139-W2-GATE-D-EXTENDED-READER-SMOKE-2026-06-29"
checkpoint_name: "CR139 W2 Gate D Extended Reader Smoke"
type: "runtime_preview_check"
status: "PASS"
owner: "host-orchestrator"
created_at: "2026-06-29T23:59:31+08:00"
checked_at: "2026-06-29T23:59:31+08:00"
---

# CR139 W2 Gate D Extended Reader Smoke

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| Current truth published | PASS | `process/evidence/CR139-W2-GATED-EXTENDED-READER-SMOKE-2026-06-29.json` | Gate D 已发布，当前步骤只读。 |
| No-write boundary | PASS | `process/evidence/CR139-W2-GATED-EXTENDED-READER-SMOKE-2026-06-29.json` | 不写 lake/catalog/manifest，不删除对象。 |

## Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | `dataset_count_17` | PASS | `process/evidence/CR139-W2-GATED-EXTENDED-READER-SMOKE-2026-06-29.json` | 通过 |
| 2 | `published_pointer_ok_17` | PASS | `process/evidence/CR139-W2-GATED-EXTENDED-READER-SMOKE-2026-06-29.json` | 通过 |
| 3 | `reader_supported_available_10` | PASS | `process/evidence/CR139-W2-GATED-EXTENDED-READER-SMOKE-2026-06-29.json` | 通过 |
| 4 | `reader_supported_rows_match_catalog_10` | PASS | `process/evidence/CR139-W2-GATED-EXTENDED-READER-SMOKE-2026-06-29.json` | 通过 |
| 5 | `pointer_level_only_unsupported_7` | PASS | `process/evidence/CR139-W2-GATED-EXTENDED-READER-SMOKE-2026-06-29.json` | 通过 |
| 6 | `parquet_rows_match_catalog_17` | PASS | `process/evidence/CR139-W2-GATED-EXTENDED-READER-SMOKE-2026-06-29.json` | 通过 |
| 7 | `no_write_operation_counts` | PASS | `process/evidence/CR139-W2-GATED-EXTENDED-READER-SMOKE-2026-06-29.json` | 通过 |

## Reader Supported Results

| Dataset | Reader Status | Reader Rows | Catalog Rows | Issues |
|---|---|---:|---:|---|
| adj_factor | available | 17905960 | 17905960 | [] |
| events | available | 350753 | 350753 | [] |
| hs300_index | available | 11511 | 11511 | [] |
| index_members | available | 283500 | 283500 | [] |
| index_weights | available | 283500 | 283500 | [] |
| prices | available | 17013700 | 17013700 | [] |
| prices_limit | available | 19210017 | 19210017 | [] |
| stock_basic | available | 38930 | 38930 | [] |
| trade_calendar | available | 10579 | 10579 | [] |
| trade_status | available | 13957403 | 13957403 | [] |

## Pointer-Level Only Results

| Dataset | Reason | Catalog Rows | Parquet Rows | Reader Issues |
|---|---|---:|---:|---|
| bse_code_mapping | read_dataset_current_api_unknown_dataset | 248 | 248 | ['unknown_dataset'] |
| industry_classification | read_dataset_current_api_unknown_dataset | 11049 | 11049 | ['unknown_dataset'] |
| lifecycle_code_change | read_dataset_current_api_unknown_dataset | 250 | 250 | ['unknown_dataset'] |
| liquidity_capacity | read_dataset_current_api_unknown_dataset | 17093658 | 17093658 | ['unknown_dataset'] |
| market_cap | read_dataset_current_api_unknown_dataset | 17001327 | 17001327 | ['unknown_dataset'] |
| prices_limit_code_change_fixes | read_dataset_current_api_unknown_dataset | 1129 | 1129 | ['unknown_dataset'] |
| prices_limit_coverage_exclusions | read_dataset_current_api_unknown_dataset | 91018 | 91018 | ['unknown_dataset'] |


## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| Failed checks absent | PASS | `process/evidence/CR139-W2-GATED-EXTENDED-READER-SMOKE-2026-06-29.json` | 无阻断项 |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Evidence | `process/evidence/CR139-W2-GATED-EXTENDED-READER-SMOKE-2026-06-29.json` | PASS | 机器证据。 |
| Index | `process/evidence/CR139-W2-GATED-EXTENDED-READER-SMOKE.index.json` | PASS | 证据索引。 |

## 结论

- 结论：`PASS`
- status：`pass_gate_d_extended_reader_smoke`
- summary：`{"active_catalog_sha256": "0ac6c1b8da8b7624f3d0dfbf879f1741254284d1faa03d42a1e01e4188c5c989", "active_manifest_line_count": 6386, "active_manifest_sha256": "32887fd7989959ff7e09daba702ea12262b6fba1837a56d1a800e52248ef54d7", "dataset_count": 17, "pointer_level_only_count": 7, "reader_supported_available_count": 10}`
- 阻断项：无
