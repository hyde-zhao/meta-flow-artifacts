---
checkpoint_id: "CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29"
checkpoint_name: "CR139 W2 Gate E-1 Full 17 Dataset Copy Migration Execution"
type: "runtime_execution_check"
status: "PASS"
owner: "host-orchestrator"
created_at: "2026-06-29T23:03:27+08:00"
checked_at: "2026-06-29T23:03:27+08:00"
target:
  phase: "CR139-W2-DATA-CONTRACTS Gate E"
  story_id: null
  artifacts:
    - "process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json"
    - "process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION.index.json"
---

# CR139 W2 Gate E-1 Full 17 Dataset Copy Migration Execution

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| Human authorization captured | PASS | `process/checkpoints/CR139-W2-GATEE-FULL17-COPY-MIGRATION-AUTHORIZATION-2026-06-29.md` | 用户已明确授权 Gate E-1 copy-only migration。 |
| Gate E-0 preview ready | PASS | `process/evidence/CR139-W2-GATEE-FULL17-NO-WRITE-MIGRATION-PREVIEW-2026-06-29.json` | 24 个 planned object，source 缺失 0，target collision 0。 |
| Active metadata baseline known | PASS | `process/evidence/CR139-W2-GATEE-FULL17-NO-WRITE-MIGRATION-PREVIEW-2026-06-29.json` | active catalog/manifest hash 已冻结。 |

## Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | `preflight_preview_status_pass` | PASS | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json` | 通过 |
| 2 | `preflight_object_count_24` | PASS | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json` | 通过 |
| 3 | `preflight_main_count_17` | PASS | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json` | 通过 |
| 4 | `preflight_quarantine_count_7` | PASS | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json` | 通过 |
| 5 | `preflight_no_source_missing` | PASS | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json` | 通过 |
| 6 | `preflight_no_target_preexisting` | PASS | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json` | 通过 |
| 7 | `copied_24_objects` | PASS | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json` | 通过 |
| 8 | `copy_errors_absent` | PASS | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json` | 通过 |
| 9 | `post_target_exists_24` | PASS | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json` | 通过 |
| 10 | `post_sha_match_24` | PASS | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json` | 通过 |
| 11 | `post_size_match_24` | PASS | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json` | 通过 |
| 12 | `post_row_count_match_24` | PASS | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json` | 通过 |
| 13 | `source_candidates_preserved_24` | PASS | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json` | 通过 |
| 14 | `active_catalog_hash_unchanged` | PASS | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json` | 通过 |
| 15 | `active_manifest_hash_unchanged` | PASS | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json` | 通过 |
| 16 | `legacy_canonical_paths_preserved` | PASS | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json` | 通过 |
| 17 | `no_move_delete_catalog_pointer_external` | PASS | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json` | 通过 |

## Object Verification

| Dataset | Role | Source Rows | Target Rows | Result | Target |
|---|---:|---:|---:|---|---|
| adj_factor | main | 17905960 | 17905960 | PASS | `canonical/adj_factor/1.0/run_id=cr139-w2-adj_factor-legacy_lake-20260629-canonical/part-adj-factor.parquet` |
| adj_factor | quarantine | 321815 | 321815 | PASS | `quality/cr139-w2/quarantine/adj_factor/part-adj-factor-superseded-metadata.parquet` |
| bse_code_mapping | main | 248 | 248 | PASS | `canonical/bse_code_mapping/1.0/run_id=cr139-w2-bse_code_mapping-legacy_lake-20260629-canonical/part-bse-code-mapping.parquet` |
| events | main | 350753 | 350753 | PASS | `canonical/events/1.0/run_id=cr139-w2-events-legacy_lake-20260629-canonical/part-events.parquet` |
| events | quarantine | 12134 | 12134 | PASS | `quality/cr139-w2/quarantine/events/part-events-conflict-and-superseded.parquet` |
| hs300_index | main | 11511 | 11511 | PASS | `canonical/hs300_index/1.0/run_id=cr139-w2-hs300_index-legacy_lake-20260629-canonical/part-hs300-index.parquet` |
| index_members | main | 283500 | 283500 | PASS | `canonical/index_members/1.0/run_id=cr139-w2-index_members-legacy_lake-20260629-canonical/part-index-members.parquet` |
| index_weights | main | 283800 | 283800 | PASS | `canonical/index_weights/1.0/run_id=cr139-w2-index_weights-legacy_lake-20260629-canonical/part-index-weights.parquet` |
| industry_classification | main | 11049 | 11049 | PASS | `canonical/industry_classification/1.0/run_id=cr139-w2-industry_classification-legacy_lake-20260629-canonical/part-industry-classification.parquet` |
| lifecycle_code_change | main | 250 | 250 | PASS | `canonical/lifecycle_code_change/1.0/run_id=cr139-w2-lifecycle_code_change-legacy_lake-20260629-canonical/part-lifecycle-code-change.parquet` |
| liquidity_capacity | main | 17093658 | 17093658 | PASS | `canonical/liquidity_capacity/1.0/run_id=cr139-w2-liquidity_capacity-legacy_lake-20260629-resolved/part-liquidity-capacity.parquet` |
| liquidity_capacity | quarantine | 1830 | 1830 | PASS | `quality/cr139-w2/quarantine/liquidity_capacity/part-liquidity-capacity-superseded-smoke.parquet` |
| market_cap | main | 17001327 | 17001327 | PASS | `canonical/market_cap/1.0/run_id=cr139-w2-market_cap-legacy_lake-20260629-resolved/part-market-cap.parquet` |
| market_cap | quarantine | 106776 | 106776 | PASS | `quality/cr139-w2/quarantine/market_cap/part-market-cap-superseded.parquet` |
| prices | main | 17013700 | 17013700 | PASS | `canonical/prices/1.0/run_id=cr139-w2-prices-legacy_lake-20260629-canonical/part-prices.parquet` |
| prices | quarantine | 401628 | 401628 | PASS | `quality/cr139-w2/quarantine/prices/part-prices-conflict-and-superseded.parquet` |
| prices_limit | main | 19210017 | 19210017 | PASS | `canonical/prices_limit/1.0/run_id=cr139-w2-prices_limit-legacy_lake-20260629-canonical/part-prices-limit.parquet` |
| prices_limit | quarantine | 11841929 | 11841929 | PASS | `quality/cr139-w2/quarantine/prices_limit/part-prices-limit-conflict-and-superseded.parquet` |
| prices_limit_code_change_fixes | main | 1129 | 1129 | PASS | `canonical/prices_limit_code_change_fixes/1.0/run_id=cr139-w2-prices_limit_code_change_fixes-legacy_lake-20260629-canonical/part-prices-limit-code-change-fixes.parquet` |
| prices_limit_coverage_exclusions | main | 91018 | 91018 | PASS | `canonical/prices_limit_coverage_exclusions/1.0/run_id=cr139-w2-prices_limit_coverage_exclusions-legacy_lake-20260629-canonical/part-prices-limit-coverage-exclusions.parquet` |
| stock_basic | main | 43558 | 43558 | PASS | `canonical/stock_basic/1.0/run_id=cr139-w2-stock_basic-legacy_lake-20260629-canonical/part-stock-basic.parquet` |
| trade_calendar | main | 10579 | 10579 | PASS | `canonical/trade_calendar/1.0/run_id=cr139-w2-trade_calendar-legacy_lake-20260629-canonical/part-trade-calendar.parquet` |
| trade_status | main | 13957403 | 13957403 | PASS | `canonical/trade_status/1.0/run_id=cr139-w2-trade_status-legacy_lake-20260629-canonical/part-trade-status.parquet` |
| trade_status | quarantine | 7612504 | 7612504 | PASS | `quality/cr139-w2/quarantine/trade_status/part-trade-status-conflict-and-superseded.parquet` |

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| 24/24 target copied and verified | PASS | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json` | SHA/size/row count 均需匹配 source candidate。 |
| Source candidate preserved | PASS | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json` | copy-only，未 move。 |
| Active metadata unchanged | PASS | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json` | 未写 active catalog/manifest。 |
| Legacy canonical preserved | PASS | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json` | 不删、不改 legacy canonical。 |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Execution evidence | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json` | PASS | Gate E-1 copy 执行证据。 |
| Evidence index | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION.index.json` | PASS | Gate E-1 摘要索引。 |
| Execution check | `process/checks/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.md` | PASS | 本文件。 |

## 结论

- 结论：`PASS`
- 复制对象：17 main + 7 quarantine = 24
- copied_object_count：24
- target_exists_count：24
- sha_match_count：24
- size_match_count：24
- row_count_match_count：24
- active_catalog_sha256：`3c9e937acb068a2d838d52e67f6990d4415306c17ba04ba4a8038adf7a7810f0`
- active_manifest_sha256：`57c5f86a7e170b99407005f54c6d66f3903fcb30a2fb0f573d8966e96eedae7a`
- 阻断项：无
- 下一步：Gate E-1 evidence review 后，进入 Gate C-2 active catalog refresh preview；仍不自动写 active catalog 或推进 pointer。
