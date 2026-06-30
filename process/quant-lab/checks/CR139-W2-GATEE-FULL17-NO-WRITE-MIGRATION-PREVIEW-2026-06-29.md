# CR139 W2 Gate E-0 Full 17 Dataset No-Write Migration Preview - 2026-06-29

## Decision

PASS_READY_FOR_GATE_E_COPY_AUTHORIZATION_REVIEW

Generated a no-write Gate E migration preview for all 17 datasets. No lake copy, move, delete, target directory creation, active catalog write, active manifest append, provider catalog/provider-lake-catalog write, published pointer advance, NAS/runtime/credential operation, or Git remote write was executed.

## Strategy

| Item | Decision |
|---|---|
| Scope | Full 17 dataset migration |
| Main target | `canonical/<dataset>/1.0/run_id=cr139-w2-.../<source-file-name>` |
| Quarantine target | `quality/cr139-w2/quarantine/<dataset>/<source-file-name>` |
| Mode | copy-only; no move; no delete |
| Candidate source | preserved |
| Legacy canonical | preserved |
| Active catalog refresh | later Gate C-2, must point to canonical CR139 targets, not candidate paths |

## Mechanical Summary

| Check | Result |
|---|---:|
| Datasets | 17 |
| Planned objects | 24 |
| Planned main objects | 17 |
| Planned quarantine objects | 7 |
| Target collisions | 0 |
| Source missing | 0 |
| Reader schema checks | 17/17 PASS |
| Planned copy bytes | 2,758,856,910 |
| Planned main copy bytes | 2,655,235,745 |
| Planned quarantine copy bytes | 103,621,165 |
| Active catalog hash unchanged | True |
| Active manifest hash unchanged | True |
| Write executed | False |

## Migration Mapping

| Dataset | Role | Source | Target | Bytes | Target exists |
|---|---|---|---|---:|---|
| `adj_factor` | main | `candidate/parquet/dataset=adj_factor/schema_version=1.0/run_id=cr139-w2-adj_factor-legacy_lake-20260629-canonical/part-adj-factor.parquet` | `canonical/adj_factor/1.0/run_id=cr139-w2-adj_factor-legacy_lake-20260629-canonical/part-adj-factor.parquet` | 77,729,543 | False |
| `adj_factor` | quarantine | `candidate/parquet/dataset=adj_factor/schema_version=1.0/run_id=cr139-w2-adj_factor-legacy_lake-20260629-canonical/quarantine/part-adj-factor-superseded-metadata.parquet` | `quality/cr139-w2/quarantine/adj_factor/part-adj-factor-superseded-metadata.parquet` | 784,970 | False |
| `bse_code_mapping` | main | `candidate/parquet/dataset=bse_code_mapping/schema_version=1.0/run_id=cr139-w2-bse_code_mapping-legacy_lake-20260629-canonical/part-bse-code-mapping.parquet` | `canonical/bse_code_mapping/1.0/run_id=cr139-w2-bse_code_mapping-legacy_lake-20260629-canonical/part-bse-code-mapping.parquet` | 16,094 | False |
| `events` | main | `candidate/parquet/dataset=events/schema_version=1.0/run_id=cr139-w2-events-legacy_lake-20260629-canonical/part-events.parquet` | `canonical/events/1.0/run_id=cr139-w2-events-legacy_lake-20260629-canonical/part-events.parquet` | 5,000,747 | False |
| `events` | quarantine | `candidate/parquet/dataset=events/schema_version=1.0/run_id=cr139-w2-events-legacy_lake-20260629-canonical/quarantine/part-events-conflict-and-superseded.parquet` | `quality/cr139-w2/quarantine/events/part-events-conflict-and-superseded.parquet` | 580,578 | False |
| `hs300_index` | main | `candidate/parquet/dataset=hs300_index/schema_version=1.0/run_id=cr139-w2-hs300_index-legacy_lake-20260629-canonical/part-hs300-index.parquet` | `canonical/hs300_index/1.0/run_id=cr139-w2-hs300_index-legacy_lake-20260629-canonical/part-hs300-index.parquet` | 837,860 | False |
| `index_members` | main | `candidate/parquet/dataset=index_members/schema_version=1.0/run_id=cr139-w2-index_members-legacy_lake-20260629-canonical/part-index-members.parquet` | `canonical/index_members/1.0/run_id=cr139-w2-index_members-legacy_lake-20260629-canonical/part-index-members.parquet` | 516,517 | False |
| `index_weights` | main | `candidate/parquet/dataset=index_weights/schema_version=1.0/run_id=cr139-w2-index_weights-legacy_lake-20260629-canonical/part-index-weights.parquet` | `canonical/index_weights/1.0/run_id=cr139-w2-index_weights-legacy_lake-20260629-canonical/part-index-weights.parquet` | 842,835 | False |
| `industry_classification` | main | `candidate/parquet/dataset=industry_classification/schema_version=1.0/run_id=cr139-w2-industry_classification-legacy_lake-20260629-canonical/part-industry-classification.parquet` | `canonical/industry_classification/1.0/run_id=cr139-w2-industry_classification-legacy_lake-20260629-canonical/part-industry-classification.parquet` | 96,102 | False |
| `lifecycle_code_change` | main | `candidate/parquet/dataset=lifecycle_code_change/schema_version=1.0/run_id=cr139-w2-lifecycle_code_change-legacy_lake-20260629-canonical/part-lifecycle-code-change.parquet` | `canonical/lifecycle_code_change/1.0/run_id=cr139-w2-lifecycle_code_change-legacy_lake-20260629-canonical/part-lifecycle-code-change.parquet` | 19,061 | False |
| `liquidity_capacity` | main | `candidate/parquet/dataset=liquidity_capacity/schema_version=1.0/run_id=cr139-w2-liquidity_capacity-legacy_lake-20260629-resolved/part-liquidity-capacity.parquet` | `canonical/liquidity_capacity/1.0/run_id=cr139-w2-liquidity_capacity-legacy_lake-20260629-resolved/part-liquidity-capacity.parquet` | 650,242,960 | False |
| `liquidity_capacity` | quarantine | `candidate/parquet/dataset=liquidity_capacity/schema_version=1.0/run_id=cr139-w2-liquidity_capacity-legacy_lake-20260629-resolved/quarantine/part-liquidity-capacity-superseded-smoke.parquet` | `quality/cr139-w2/quarantine/liquidity_capacity/part-liquidity-capacity-superseded-smoke.parquet` | 71,485 | False |
| `market_cap` | main | `candidate/parquet/dataset=market_cap/schema_version=1.0/run_id=cr139-w2-market_cap-legacy_lake-20260629-resolved/part-market-cap.parquet` | `canonical/market_cap/1.0/run_id=cr139-w2-market_cap-legacy_lake-20260629-resolved/part-market-cap.parquet` | 987,802,463 | False |
| `market_cap` | quarantine | `candidate/parquet/dataset=market_cap/schema_version=1.0/run_id=cr139-w2-market_cap-legacy_lake-20260629-resolved/quarantine/part-market-cap-superseded.parquet` | `quality/cr139-w2/quarantine/market_cap/part-market-cap-superseded.parquet` | 8,004,751 | False |
| `prices` | main | `candidate/parquet/dataset=prices/schema_version=1.0/run_id=cr139-w2-prices-legacy_lake-20260629-canonical/part-prices.parquet` | `canonical/prices/1.0/run_id=cr139-w2-prices-legacy_lake-20260629-canonical/part-prices.parquet` | 778,036,005 | False |
| `prices` | quarantine | `candidate/parquet/dataset=prices/schema_version=1.0/run_id=cr139-w2-prices-legacy_lake-20260629-canonical/quarantine/part-prices-conflict-and-superseded.parquet` | `quality/cr139-w2/quarantine/prices/part-prices-conflict-and-superseded.parquet` | 9,307,225 | False |
| `prices_limit` | main | `candidate/parquet/dataset=prices_limit/schema_version=1.0/run_id=cr139-w2-prices_limit-legacy_lake-20260629-canonical/part-prices-limit.parquet` | `canonical/prices_limit/1.0/run_id=cr139-w2-prices_limit-legacy_lake-20260629-canonical/part-prices-limit.parquet` | 117,729,976 | False |
| `prices_limit` | quarantine | `candidate/parquet/dataset=prices_limit/schema_version=1.0/run_id=cr139-w2-prices_limit-legacy_lake-20260629-canonical/quarantine/part-prices-limit-conflict-and-superseded.parquet` | `quality/cr139-w2/quarantine/prices_limit/part-prices-limit-conflict-and-superseded.parquet` | 67,212,938 | False |
| `prices_limit_code_change_fixes` | main | `candidate/parquet/dataset=prices_limit_code_change_fixes/schema_version=1.0/run_id=cr139-w2-prices_limit_code_change_fixes-legacy_lake-20260629-canonical/part-prices-limit-code-change-fixes.parquet` | `canonical/prices_limit_code_change_fixes/1.0/run_id=cr139-w2-prices_limit_code_change_fixes-legacy_lake-20260629-canonical/part-prices-limit-code-change-fixes.parquet` | 25,072 | False |
| `prices_limit_coverage_exclusions` | main | `candidate/parquet/dataset=prices_limit_coverage_exclusions/schema_version=1.0/run_id=cr139-w2-prices_limit_coverage_exclusions-legacy_lake-20260629-canonical/part-prices-limit-coverage-exclusions.parquet` | `canonical/prices_limit_coverage_exclusions/1.0/run_id=cr139-w2-prices_limit_coverage_exclusions-legacy_lake-20260629-canonical/part-prices-limit-coverage-exclusions.parquet` | 168,133 | False |
| `stock_basic` | main | `candidate/parquet/dataset=stock_basic/schema_version=1.0/run_id=cr139-w2-stock_basic-legacy_lake-20260629-canonical/part-stock-basic.parquet` | `canonical/stock_basic/1.0/run_id=cr139-w2-stock_basic-legacy_lake-20260629-canonical/part-stock-basic.parquet` | 514,235 | False |
| `trade_calendar` | main | `candidate/parquet/dataset=trade_calendar/schema_version=1.0/run_id=cr139-w2-trade_calendar-legacy_lake-20260629-canonical/part-trade-calendar.parquet` | `canonical/trade_calendar/1.0/run_id=cr139-w2-trade_calendar-legacy_lake-20260629-canonical/part-trade-calendar.parquet` | 167,982 | False |
| `trade_status` | main | `candidate/parquet/dataset=trade_status/schema_version=1.0/run_id=cr139-w2-trade_status-legacy_lake-20260629-canonical/part-trade-status.parquet` | `canonical/trade_status/1.0/run_id=cr139-w2-trade_status-legacy_lake-20260629-canonical/part-trade-status.parquet` | 35,490,160 | False |
| `trade_status` | quarantine | `candidate/parquet/dataset=trade_status/schema_version=1.0/run_id=cr139-w2-trade_status-legacy_lake-20260629-canonical/quarantine/part-trade-status-conflict-and-superseded.parquet` | `quality/cr139-w2/quarantine/trade_status/part-trade-status-conflict-and-superseded.parquet` | 17,659,218 | False |

## Reader Schema Compatibility

| Dataset | Status | Contract source | Required fields | Observed fields |
|---|---|---|---:|---:|
| `adj_factor` | pass | DATASET_SCHEMA_REGISTRY.columns | 11 | 11 |
| `bse_code_mapping` | pass | fallback:DATASET_KEY_COLUMNS_or_observed_fields | 10 | 10 |
| `events` | pass | DATASET_SCHEMA_REGISTRY.columns | 11 | 11 |
| `hs300_index` | pass | DATASET_SCHEMA_REGISTRY.columns | 18 | 18 |
| `index_members` | pass | DATASET_SCHEMA_REGISTRY.columns | 19 | 19 |
| `index_weights` | pass | DATASET_SCHEMA_REGISTRY.columns | 15 | 15 |
| `industry_classification` | pass | fallback:DATASET_KEY_COLUMNS_or_observed_fields | 15 | 15 |
| `lifecycle_code_change` | pass | fallback:DATASET_KEY_COLUMNS_or_observed_fields | 13 | 13 |
| `liquidity_capacity` | pass | fallback:DATASET_KEY_COLUMNS_or_observed_fields | 16 | 16 |
| `market_cap` | pass | fallback:DATASET_KEY_COLUMNS_or_observed_fields | 25 | 25 |
| `prices` | pass | DATASET_SCHEMA_REGISTRY.columns | 21 | 21 |
| `prices_limit` | pass | DATASET_SCHEMA_REGISTRY.columns | 11 | 11 |
| `prices_limit_code_change_fixes` | pass | fallback:DATASET_KEY_COLUMNS_or_observed_fields | 13 | 13 |
| `prices_limit_coverage_exclusions` | pass | fallback:DATASET_KEY_COLUMNS_or_observed_fields | 10 | 10 |
| `stock_basic` | pass | DATASET_SCHEMA_REGISTRY.columns | 17 | 17 |
| `trade_calendar` | pass | DATASET_SCHEMA_REGISTRY.columns | 11 | 11 |
| `trade_status` | pass | DATASET_SCHEMA_REGISTRY.columns | 13 | 13 |

## Post-Copy Verification Plan

- Verify 24/24 target files exist.
- Verify target SHA-256, file size, and parquet row count equal source candidate object manifest.
- Verify all source candidate files still exist.
- Verify legacy canonical roots still exist and are unchanged.
- Verify active `catalog/catalog.json` hash remains `3c9e937acb068a2d838d52e67f6990d4415306c17ba04ba4a8038adf7a7810f0`.
- Verify active `manifest/market_data_manifest.jsonl` hash remains `57c5f86a7e170b99407005f54c6d66f3903fcb30a2fb0f573d8966e96eedae7a`.
- Verify no provider catalog/provider-lake-catalog write and no published pointer advance occurred.

## Boundary

This preview does not authorize or execute physical migration. Gate E-1 requires separate explicit authorization. Candidate cleanup and legacy cleanup remain separate future gates and are not implied by migration copy.

Evidence index: `process/evidence/CR139-W2-GATEE-FULL17-NO-WRITE-MIGRATION-PREVIEW.index.json`
