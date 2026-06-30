---
checkpoint_id: "CR139-W2-GATEC2-ACTIVE-CATALOG-MANIFEST-WRITE-EXECUTION-2026-06-29"
checkpoint_name: "CR139 W2 Gate C-2 Active Catalog and Manifest Write Execution"
type: "runtime_execution_check"
status: "PASS"
owner: "host-orchestrator"
created_at: "2026-06-29T23:18:44+08:00"
checked_at: "2026-06-29T23:18:44+08:00"
target:
  phase: "CR139-W2-DATA-CONTRACTS Gate C-2"
  story_id: null
  artifacts:
    - "process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-MANIFEST-WRITE-EXECUTION-2026-06-29.json"
    - "process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-MANIFEST-WRITE-EXECUTION.index.json"
---

# CR139 W2 Gate C-2 Active Catalog and Manifest Write Execution

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| Human authorization captured | PASS | `process/checkpoints/CR139-W2-GATEC2-ACTIVE-CATALOG-MANIFEST-WRITE-AUTHORIZATION-2026-06-29.md` | 用户已明确授权 active catalog + manifest write。 |
| Gate C-2 preview passed | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-PREVIEW-2026-06-29.json` | no-write preview failed_checks=0。 |
| Active metadata pre-write guard passed | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-MANIFEST-WRITE-EXECUTION-2026-06-29.json` | pre-write catalog/manifest hash 与 preview baseline 一致。 |

## Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | `active_catalog_json_parse` | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-MANIFEST-WRITE-EXECUTION-2026-06-29.json` | 通过 |
| 2 | `active_catalog_sha_equals_virtual_after` | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-MANIFEST-WRITE-EXECUTION-2026-06-29.json` | 通过 |
| 3 | `active_manifest_line_count_6384` | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-MANIFEST-WRITE-EXECUTION-2026-06-29.json` | 通过 |
| 4 | `active_manifest_appended_17_exact_preview_lines` | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-MANIFEST-WRITE-EXECUTION-2026-06-29.json` | 通过 |
| 5 | `active_catalog_dataset_count_17` | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-MANIFEST-WRITE-EXECUTION-2026-06-29.json` | 通过 |
| 6 | `active_catalog_cr139_canonical_paths_17` | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-MANIFEST-WRITE-EXECUTION-2026-06-29.json` | 通过 |
| 7 | `active_catalog_published_false_17` | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-MANIFEST-WRITE-EXECUTION-2026-06-29.json` | 通过 |
| 8 | `active_catalog_lineage_checksum_present_17` | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-MANIFEST-WRITE-EXECUTION-2026-06-29.json` | 通过 |
| 9 | `operation_counts_match_authorization` | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-MANIFEST-WRITE-EXECUTION-2026-06-29.json` | 通过 |

## Active Catalog Records After Write

| Dataset | canonical_path | Rows | latest_manifest_run_id | lineage_checksum | published |
|---|---|---:|---|---|---:|
| adj_factor | `canonical/adj_factor/1.0/run_id=cr139-w2-adj_factor-legacy_lake-20260629-canonical/part-adj-factor.parquet` | 17905960 | `cr139-w2-adj_factor-legacy_lake-20260629-canonical` | `sha256:a3255d68c1f3023b4165c4f64a2adb7610fad0e057b15dd4404b32c2edad7045` | False |
| bse_code_mapping | `canonical/bse_code_mapping/1.0/run_id=cr139-w2-bse_code_mapping-legacy_lake-20260629-canonical/part-bse-code-mapping.parquet` | 248 | `cr139-w2-bse_code_mapping-legacy_lake-20260629-canonical` | `sha256:68bbee8f95b3b0ecfd54fb65b5e0cb4950341e8fb9c7908303d62009d9ae3208` | False |
| events | `canonical/events/1.0/run_id=cr139-w2-events-legacy_lake-20260629-canonical/part-events.parquet` | 350753 | `cr139-w2-events-legacy_lake-20260629-canonical` | `sha256:a11a741f1e6a283a28d973cddce07971ae71506a9716b366fcb9636d38d5e57a` | False |
| hs300_index | `canonical/hs300_index/1.0/run_id=cr139-w2-hs300_index-legacy_lake-20260629-canonical/part-hs300-index.parquet` | 11511 | `cr139-w2-hs300_index-legacy_lake-20260629-canonical` | `sha256:d6f0c942276db655a17e7e0f83838634da4307b32fa41221cfd042000a72e1fd` | False |
| index_members | `canonical/index_members/1.0/run_id=cr139-w2-index_members-legacy_lake-20260629-canonical/part-index-members.parquet` | 283500 | `cr139-w2-index_members-legacy_lake-20260629-canonical` | `sha256:2b2a74d2e503dbeabe93295c5df2fba7609316362fa12c25c5a2f39dcc040027` | False |
| index_weights | `canonical/index_weights/1.0/run_id=cr139-w2-index_weights-legacy_lake-20260629-canonical/part-index-weights.parquet` | 283800 | `cr139-w2-index_weights-legacy_lake-20260629-canonical` | `sha256:9461a9db399540ad625dd00fed6795c3a316bde77ece97ebffabb2691c8fe99a` | False |
| industry_classification | `canonical/industry_classification/1.0/run_id=cr139-w2-industry_classification-legacy_lake-20260629-canonical/part-industry-classification.parquet` | 11049 | `cr139-w2-industry_classification-legacy_lake-20260629-canonical` | `sha256:8b9f628e4c979a7b45cafbc9528be1f5eb37eaca82e0cf7bf51894047dfed2f8` | False |
| lifecycle_code_change | `canonical/lifecycle_code_change/1.0/run_id=cr139-w2-lifecycle_code_change-legacy_lake-20260629-canonical/part-lifecycle-code-change.parquet` | 250 | `cr139-w2-lifecycle_code_change-legacy_lake-20260629-canonical` | `sha256:10b95f35fd9296751d4d919bdd25e606f7a8d9a339faf129cf2c8919ac7d934d` | False |
| liquidity_capacity | `canonical/liquidity_capacity/1.0/run_id=cr139-w2-liquidity_capacity-legacy_lake-20260629-resolved/part-liquidity-capacity.parquet` | 17093658 | `cr139-w2-liquidity_capacity-legacy_lake-20260629-resolved` | `sha256:6ff2d40f77f0f4bed8dec9eaba52fe676bd93910f40ef27c913bc7c767155af4` | False |
| market_cap | `canonical/market_cap/1.0/run_id=cr139-w2-market_cap-legacy_lake-20260629-resolved/part-market-cap.parquet` | 17001327 | `cr139-w2-market_cap-legacy_lake-20260629-resolved` | `sha256:e2b7f8d4dd8ad1e254194b703ca298a5760093abb95c2fe26762ce7b14db8d03` | False |
| prices | `canonical/prices/1.0/run_id=cr139-w2-prices-legacy_lake-20260629-canonical/part-prices.parquet` | 17013700 | `cr139-w2-prices-legacy_lake-20260629-canonical` | `sha256:305049fbb9437e0e584aecfda1b35afcd28e9912d13485642f011993a3f6cf82` | False |
| prices_limit | `canonical/prices_limit/1.0/run_id=cr139-w2-prices_limit-legacy_lake-20260629-canonical/part-prices-limit.parquet` | 19210017 | `cr139-w2-prices_limit-legacy_lake-20260629-canonical` | `sha256:fb0613e005a6281040f52649480bfb795e10704122b811e513ba2f3d9d669f2b` | False |
| prices_limit_code_change_fixes | `canonical/prices_limit_code_change_fixes/1.0/run_id=cr139-w2-prices_limit_code_change_fixes-legacy_lake-20260629-canonical/part-prices-limit-code-change-fixes.parquet` | 1129 | `cr139-w2-prices_limit_code_change_fixes-legacy_lake-20260629-canonical` | `sha256:77865aeb0a4653f7a342d388e9ff118ca9b70bdd9ab2595a52a5e9025ce69c4d` | False |
| prices_limit_coverage_exclusions | `canonical/prices_limit_coverage_exclusions/1.0/run_id=cr139-w2-prices_limit_coverage_exclusions-legacy_lake-20260629-canonical/part-prices-limit-coverage-exclusions.parquet` | 91018 | `cr139-w2-prices_limit_coverage_exclusions-legacy_lake-20260629-canonical` | `sha256:82cb028d6bf9af7ab306f2aa0e7f7f94bb1ad95a9bea29a42bda7e24ea152437` | False |
| stock_basic | `canonical/stock_basic/1.0/run_id=cr139-w2-stock_basic-legacy_lake-20260629-canonical/part-stock-basic.parquet` | 43558 | `cr139-w2-stock_basic-legacy_lake-20260629-canonical` | `sha256:3ea9fd522516a5dd356d2dd17dd91e70c4c820191dd802be4d1e4100dc668ec3` | False |
| trade_calendar | `canonical/trade_calendar/1.0/run_id=cr139-w2-trade_calendar-legacy_lake-20260629-canonical/part-trade-calendar.parquet` | 10579 | `cr139-w2-trade_calendar-legacy_lake-20260629-canonical` | `sha256:b6addf8b67d2f7029da0d974bd07aecc2feadd2ffacf3b6f0b47ee7fca83d5bc` | False |
| trade_status | `canonical/trade_status/1.0/run_id=cr139-w2-trade_status-legacy_lake-20260629-canonical/part-trade-status.parquet` | 13957403 | `cr139-w2-trade_status-legacy_lake-20260629-canonical` | `sha256:1b790318af6f62af2119708cc3c2420ae337ad7c9f0a14154810a797e617957f` | False |

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| Active catalog replaced | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-MANIFEST-WRITE-EXECUTION-2026-06-29.json` | active catalog hash equals virtual after. |
| Active manifest appended | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-MANIFEST-WRITE-EXECUTION-2026-06-29.json` | line count 6367 -> 6384。 |
| Pointer not advanced | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-MANIFEST-WRITE-EXECUTION-2026-06-29.json` | Gate D remains separate。 |
| Provider/runtime/external forbidden ops not touched | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-MANIFEST-WRITE-EXECUTION-2026-06-29.json` | provider/NAS/runtime/credential/Git all 0。 |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Execution evidence | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-MANIFEST-WRITE-EXECUTION-2026-06-29.json` | PASS | Gate C-2 active metadata write 证据。 |
| Evidence index | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-MANIFEST-WRITE-EXECUTION.index.json` | PASS | 摘要索引。 |
| Execution check | `process/checks/CR139-W2-GATEC2-ACTIVE-CATALOG-MANIFEST-WRITE-EXECUTION-2026-06-29.md` | PASS | 本文件。 |

## 结论

- 结论：`PASS`
- active_catalog_write_count：1
- active_manifest_append_count：17
- active_catalog_sha256_before：`3c9e937acb068a2d838d52e67f6990d4415306c17ba04ba4a8038adf7a7810f0`
- active_catalog_sha256_after：`87cadaa5048e68067863c6d151ce1671a3883b2e858fc334ea50305f029cebc3`
- active_manifest_sha256_before：`57c5f86a7e170b99407005f54c6d66f3903fcb30a2fb0f573d8966e96eedae7a`
- active_manifest_sha256_after：`3c0a67982b5234c544ee262df73e90d8729bf4caaf8803e300a2a2aaabce2439`
- active_manifest_line_count_before：6367
- active_manifest_line_count_after：6384
- cr139_canonical_path_count：17
- published_true_count：0
- lineage_checksum_present_count：17
- 阻断项：无
- 下一步：Gate D no-write pointer advance preview；不自动推进 pointer。
