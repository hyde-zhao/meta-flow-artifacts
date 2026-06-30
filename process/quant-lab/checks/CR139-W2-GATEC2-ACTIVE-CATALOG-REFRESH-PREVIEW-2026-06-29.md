---
checkpoint_id: "CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-PREVIEW-2026-06-29"
checkpoint_name: "CR139 W2 Gate C-2 Active Catalog Refresh Preview"
type: "auto_precheck"
status: "PASS"
owner: "host-orchestrator"
created_at: "2026-06-29T23:08:56+08:00"
checked_at: "2026-06-29T23:08:56+08:00"
target:
  phase: "CR139-W2-DATA-CONTRACTS Gate C-2"
  story_id: null
  artifacts:
    - "process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-PREVIEW-2026-06-29.json"
    - "process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-PREVIEW.index.json"
    - "process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-VIRTUAL-DIFF-2026-06-29.diff"
    - "process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-AFTER-VIRTUAL-2026-06-29.json"
    - "process/evidence/CR139-W2-GATEC2-ACTIVE-MANIFEST-APPEND-PREVIEW-2026-06-29.jsonl"
---

# CR139 W2 Gate C-2 Active Catalog Refresh Preview

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| Gate C staged metadata exists | PASS | `process/evidence/CR139-W2-GATEC-STAGED-CATALOG-MANIFEST-PREVIEW-2026-06-29.json` | staged catalog/manifest 已作为 lineage 输入。 |
| Gate E-1 copy verified | PASS | `process/evidence/CR139-W2-GATEE-FULL17-COPY-MIGRATION-EXECUTION-2026-06-29.json` | 24/24 canonical/quality target 已落位并验证。 |
| Active catalog readable | PASS | `/home/hyde/data/quant-lab/data-lake/catalog/catalog.json` | 本轮只读。 |
| Active manifest readable | PASS | `/home/hyde/data/quant-lab/data-lake/manifest/market_data_manifest.jsonl` | 本轮只读。 |

## Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | `gate_e1_copy_verified` | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-PREVIEW-2026-06-29.json` | 通过 |
| 2 | `dataset_count_17` | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-PREVIEW-2026-06-29.json` | 通过 |
| 3 | `canonical_targets_exist_17` | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-PREVIEW-2026-06-29.json` | 通过 |
| 4 | `canonical_targets_use_cr139_run_id_17` | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-PREVIEW-2026-06-29.json` | 通过 |
| 5 | `canonical_targets_not_candidate_17` | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-PREVIEW-2026-06-29.json` | 通过 |
| 6 | `published_remains_false_17` | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-PREVIEW-2026-06-29.json` | 通过 |
| 7 | `lineage_checksum_valid_17` | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-PREVIEW-2026-06-29.json` | 通过 |
| 8 | `catalog_manifest_consistency_17` | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-PREVIEW-2026-06-29.json` | 通过 |
| 9 | `manifest_append_preview_17` | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-PREVIEW-2026-06-29.json` | 通过 |
| 10 | `active_catalog_file_unchanged` | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-PREVIEW-2026-06-29.json` | 通过 |
| 11 | `active_manifest_file_unchanged` | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-PREVIEW-2026-06-29.json` | 通过 |
| 12 | `no_write_operation_counts` | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-PREVIEW-2026-06-29.json` | 通过 |

## Refresh Preview

| Dataset | Before canonical_path | After canonical_path | Before rows | After rows | Lineage checksum |
|---|---|---|---:|---:|---|
| adj_factor | `canonical/adj_factor/1.0/run_id=run-stage3-data-update-20260530-20260626-prices-20260627` | `canonical/adj_factor/1.0/run_id=cr139-w2-adj_factor-legacy_lake-20260629-canonical/part-adj-factor.parquet` | 105041 | 17905960 | `sha256:a3255d68c1f3023b4165c4f64a2adb7610fad0e057b15dd4404b32c2edad7045` |
| bse_code_mapping | `canonical/bse_code_mapping/1.0/run_id=run-cr018-price-limit-lifecycle-cleanup-20150101-20260528-20260529` | `canonical/bse_code_mapping/1.0/run_id=cr139-w2-bse_code_mapping-legacy_lake-20260629-canonical/part-bse-code-mapping.parquet` | 248 | 248 | `sha256:68bbee8f95b3b0ecfd54fb65b5e0cb4950341e8fb9c7908303d62009d9ae3208` |
| events | `canonical/events/1.0/run_id=run-stage3-data-update-20260530-20260626-trade-status-20260627/part-events-stock-st.parquet` | `canonical/events/1.0/run_id=cr139-w2-events-legacy_lake-20260629-canonical/part-events.parquet` | 4407 | 350753 | `sha256:a11a741f1e6a283a28d973cddce07971ae71506a9716b366fcb9636d38d5e57a` |
| hs300_index | `canonical/hs300_index/1.0/run_id=run-stage3-data-update-20260530-20260626-aux-20260627/part-benchmark-index-daily.parquet` | `canonical/hs300_index/1.0/run_id=cr139-w2-hs300_index-legacy_lake-20260629-canonical/part-hs300-index.parquet` | 76 | 11511 | `sha256:d6f0c942276db655a17e7e0f83838634da4307b32fa41221cfd042000a72e1fd` |
| index_members | `canonical/index_members/1.0/run_id=run-stage3-data-update-20260530-20260626-aux-20260627/part-benchmark-index-members.parquet` | `canonical/index_members/1.0/run_id=cr139-w2-index_members-legacy_lake-20260629-canonical/part-index-members.parquet` | 300 | 283500 | `sha256:2b2a74d2e503dbeabe93295c5df2fba7609316362fa12c25c5a2f39dcc040027` |
| index_weights | `canonical/index_weights/1.0/run_id=run-stage3-data-update-20260530-20260626-aux-20260627/part-benchmark-index-weights.parquet` | `canonical/index_weights/1.0/run_id=cr139-w2-index_weights-legacy_lake-20260629-canonical/part-index-weights.parquet` | 300 | 283800 | `sha256:9461a9db399540ad625dd00fed6795c3a316bde77ece97ebffabb2691c8fe99a` |
| industry_classification | `canonical/industry_classification/1.0/run_id=run-stage3-data-update-20260530-20260626-aux-20260627/part-industry-classification.parquet` | `canonical/industry_classification/1.0/run_id=cr139-w2-industry_classification-legacy_lake-20260629-canonical/part-industry-classification.parquet` | 5525 | 11049 | `sha256:8b9f628e4c979a7b45cafbc9528be1f5eb37eaca82e0cf7bf51894047dfed2f8` |
| lifecycle_code_change | `canonical/lifecycle_code_change/1.0/run_id=run-cr018-price-limit-lifecycle-cleanup-20150101-20260528-20260529` | `canonical/lifecycle_code_change/1.0/run_id=cr139-w2-lifecycle_code_change-legacy_lake-20260629-canonical/part-lifecycle-code-change.parquet` | 250 | 250 | `sha256:10b95f35fd9296751d4d919bdd25e606f7a8d9a339faf129cf2c8919ac7d934d` |
| liquidity_capacity | `canonical/liquidity_capacity/1.0/run_id=run-stage3-data-update-20260530-20260626-prices-20260627` | `canonical/liquidity_capacity/1.0/run_id=cr139-w2-liquidity_capacity-legacy_lake-20260629-resolved/part-liquidity-capacity.parquet` | 104712 | 17093658 | `sha256:6ff2d40f77f0f4bed8dec9eaba52fe676bd93910f40ef27c913bc7c767155af4` |
| market_cap | `canonical/market_cap/1.0/run_id=run-stage3-data-update-20260530-20260626-aux-20260627/part-market-cap-2026.parquet` | `canonical/market_cap/1.0/run_id=cr139-w2-market_cap-legacy_lake-20260629-resolved/part-market-cap.parquet` | 104946 | 17001327 | `sha256:e2b7f8d4dd8ad1e254194b703ca298a5760093abb95c2fe26762ce7b14db8d03` |
| prices | `canonical/prices/1.0/run_id=run-stage3-data-update-20260530-20260626-prices-20260627` | `canonical/prices/1.0/run_id=cr139-w2-prices-legacy_lake-20260629-canonical/part-prices.parquet` | 104712 | 17013700 | `sha256:305049fbb9437e0e584aecfda1b35afcd28e9912d13485642f011993a3f6cf82` |
| prices_limit | `canonical/prices_limit/1.0/run_id=run-stage3-data-update-20260530-20260626-aux-20260627/part-prices-limit-2026.parquet` | `canonical/prices_limit/1.0/run_id=cr139-w2-prices_limit-legacy_lake-20260629-canonical/part-prices-limit.parquet` | 145346 | 19210017 | `sha256:fb0613e005a6281040f52649480bfb795e10704122b811e513ba2f3d9d669f2b` |
| prices_limit_code_change_fixes | `canonical/prices_limit_code_change_fixes/1.0/run_id=run-cr018-price-limit-lifecycle-cleanup-20150101-20260528-20260529` | `canonical/prices_limit_code_change_fixes/1.0/run_id=cr139-w2-prices_limit_code_change_fixes-legacy_lake-20260629-canonical/part-prices-limit-code-change-fixes.parquet` | 1129 | 1129 | `sha256:77865aeb0a4653f7a342d388e9ff118ca9b70bdd9ab2595a52a5e9025ce69c4d` |
| prices_limit_coverage_exclusions | `canonical/prices_limit_coverage_exclusions/1.0/run_id=run-cr018-price-limit-lifecycle-cleanup-20150101-20260528-20260529` | `canonical/prices_limit_coverage_exclusions/1.0/run_id=cr139-w2-prices_limit_coverage_exclusions-legacy_lake-20260629-canonical/part-prices-limit-coverage-exclusions.parquet` | 91018 | 91018 | `sha256:82cb028d6bf9af7ab306f2aa0e7f7f94bb1ad95a9bea29a42bda7e24ea152437` |
| stock_basic | `canonical/stock_basic/1.0/run_id=run-stage3-data-update-20260530-20260626-aux-20260627/part-stock-basic-lifecycle.parquet` | `canonical/stock_basic/1.0/run_id=cr139-w2-stock_basic-legacy_lake-20260629-canonical/part-stock-basic.parquet` | 5857 | 43558 | `sha256:3ea9fd522516a5dd356d2dd17dd91e70c4c820191dd802be4d1e4100dc668ec3` |
| trade_calendar | `canonical/trade_calendar/1.0/run_id=run-stage3-data-update-20260530-20260626-prices-20260627` | `canonical/trade_calendar/1.0/run_id=cr139-w2-trade_calendar-legacy_lake-20260629-canonical/part-trade-calendar.parquet` | 28 | 10579 | `sha256:b6addf8b67d2f7029da0d974bd07aecc2feadd2ffacf3b6f0b47ee7fca83d5bc` |
| trade_status | `canonical/trade_status/1.0/run_id=run-stage3-data-update-20260530-20260626-trade-status-20260627/part-trade-status-2026.parquet` | `canonical/trade_status/1.0/run_id=cr139-w2-trade_status-legacy_lake-20260629-canonical/part-trade-status.parquet` | 104712 | 13957403 | `sha256:1b790318af6f62af2119708cc3c2420ae337ad7c9f0a14154810a797e617957f` |

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| No-write preview generated | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-PREVIEW-2026-06-29.json` | 只生成虚拟 after catalog、diff 和 manifest append preview。 |
| Active catalog unchanged | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-PREVIEW-2026-06-29.json` | 当前 active catalog hash 未变。 |
| Active manifest unchanged | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-PREVIEW-2026-06-29.json` | 当前 active manifest hash 未变。 |
| Current truth remains invisible | PASS | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-PREVIEW-2026-06-29.json` | 虚拟 after 中 17/17 published=false；Gate D pointer advance 未授权。 |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Preview JSON | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-PREVIEW-2026-06-29.json` | PASS | Gate C-2 机器证据。 |
| Evidence index | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-PREVIEW.index.json` | PASS | 预览摘要索引。 |
| Virtual active catalog after | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-AFTER-VIRTUAL-2026-06-29.json` | PASS | 不写入 lake，仅供审查。 |
| Virtual diff | `process/evidence/CR139-W2-GATEC2-ACTIVE-CATALOG-REFRESH-VIRTUAL-DIFF-2026-06-29.diff` | PASS | active catalog before/after 差异。 |
| Manifest append preview | `process/evidence/CR139-W2-GATEC2-ACTIVE-MANIFEST-APPEND-PREVIEW-2026-06-29.jsonl` | PASS | 17 条 manifest 预览，不 append active manifest。 |

## 结论

- 结论：`PASS`
- catalog_record_refresh_count：17
- manifest_append_preview_count：17
- canonical_target_exists_count：17
- lineage_checksum_validation_pass_count：17
- catalog_manifest_consistency_pass_count：17
- active_catalog_sha256_before：`3c9e937acb068a2d838d52e67f6990d4415306c17ba04ba4a8038adf7a7810f0`
- active_catalog_sha256_after_preview：`3c9e937acb068a2d838d52e67f6990d4415306c17ba04ba4a8038adf7a7810f0`
- active_manifest_sha256_before：`57c5f86a7e170b99407005f54c6d66f3903fcb30a2fb0f573d8966e96eedae7a`
- active_manifest_sha256_after_preview：`57c5f86a7e170b99407005f54c6d66f3903fcb30a2fb0f573d8966e96eedae7a`
- active_catalog_after_virtual_sha256：`87cadaa5048e68067863c6d151ce1671a3883b2e858fc334ea50305f029cebc3`
- 阻断项：无
- 下一步：发起 Gate C-2 active catalog refresh write 授权；仍不自动推进 pointer 或写 provider catalog。
