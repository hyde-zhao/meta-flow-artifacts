# CR139 W2 Gate C Staged Catalog/Manifest Preview - 2026-06-29

## Decision

PASS_NO_WRITE_PREVIEW_GENERATED

Generated a Gate C staged catalog/manifest preview only. This is not current truth and does not authorize or execute active lake catalog writes, manifest appends, provider catalog/provider-lake-catalog writes, published pointer advance, physical migration, NAS/runtime/credential operations, Git remote writes, legacy deletion, or rollback deletion.

## Validation Summary

| Check | Result |
|---|---:|
| Candidate manifest records | 17 |
| Staged candidate catalog records | 17 |
| Candidate datasets | 17 |
| Candidate parquet objects | 24 |
| Main objects | 17 |
| Quarantine objects | 7 |
| Manifest record validations passed | 17/17 |
| Lineage checksum validations passed | 17/17 |
| Validation failures | 0 |
| Current truth visible | False |

## Active Metadata Before Snapshot

| File | SHA-256 | Count |
|---|---|---:|
| `catalog/catalog.json` | `3c9e937acb068a2d838d52e67f6990d4415306c17ba04ba4a8038adf7a7810f0` | 17 datasets |
| `manifest/market_data_manifest.jsonl` | `57c5f86a7e170b99407005f54c6d66f3903fcb30a2fb0f573d8966e96eedae7a` | 6367 lines |

## Staged Candidate Catalog Shape

The staged catalog uses schema `cr139.gatec.staged_candidate_catalog.v1`. It carries `candidate_main_path` and `candidate_quarantine_paths` separately and marks every record `current_truth_visible=false`. It intentionally does not overload active `CatalogEntry.canonical_path`.

| Dataset | Candidate run_id | Main rows | Quarantine rows | Lineage checksum |
|---|---|---:|---:|---|
| `adj_factor` | `cr139-w2-adj_factor-legacy_lake-20260629-canonical` | 17,905,960 | 321,815 | `sha256:82658c3dfe1b835ce2333c01ba1fbea5dd418dd53f6d5d02263d031e7bc26fbe` |
| `bse_code_mapping` | `cr139-w2-bse_code_mapping-legacy_lake-20260629-canonical` | 248 | 0 | `sha256:f2eb8941b91750afaf662ecf272c813bdad6278daaa1de175d2b82dd41473aff` |
| `events` | `cr139-w2-events-legacy_lake-20260629-canonical` | 350,753 | 12,134 | `sha256:b6ae0cea3de1fa1bc74fefd0aa6b3ccd51ff767bc9437f5a840b2cf5f7d0b1d9` |
| `hs300_index` | `cr139-w2-hs300_index-legacy_lake-20260629-canonical` | 11,511 | 0 | `sha256:f8df3b9e118030d6e26d31eb6073e1a8e4c1fbaef5e4bed9e9516762eb87e842` |
| `index_members` | `cr139-w2-index_members-legacy_lake-20260629-canonical` | 283,500 | 0 | `sha256:b36ca817df2ccc02a76eec29b5df581578f1d9588da3e5089157ffd288563bff` |
| `index_weights` | `cr139-w2-index_weights-legacy_lake-20260629-canonical` | 283,800 | 0 | `sha256:497aee5a274ac43799981345593de862efdb8781fbfe45ccc350f122118f27eb` |
| `industry_classification` | `cr139-w2-industry_classification-legacy_lake-20260629-canonical` | 11,049 | 0 | `sha256:42f617f3deccf944cc8eec8d62972604bb07bb01519f52d54fe8c906c9f5d5a3` |
| `lifecycle_code_change` | `cr139-w2-lifecycle_code_change-legacy_lake-20260629-canonical` | 250 | 0 | `sha256:457006a3d16fa8fadfc24312bd5b1254f5bf5b183e63a72cf290f94722193f59` |
| `liquidity_capacity` | `cr139-w2-liquidity_capacity-legacy_lake-20260629-resolved` | 17,093,658 | 1,830 | `sha256:e3825999f22a3c035be17cfabc3fd3551de44dc9abe94a89dbae584758c9249b` |
| `market_cap` | `cr139-w2-market_cap-legacy_lake-20260629-resolved` | 17,001,327 | 106,776 | `sha256:f8d28a73f37b22f6ecd8fa6622763e280652a7b1556ebacdcabcae6e7094a0aa` |
| `prices` | `cr139-w2-prices-legacy_lake-20260629-canonical` | 17,013,700 | 401,628 | `sha256:31c682195d13526c55d1e37a3232d15c799031719ba6fecc32f1bda1cae16fe7` |
| `prices_limit` | `cr139-w2-prices_limit-legacy_lake-20260629-canonical` | 19,210,017 | 11,841,929 | `sha256:33b61c080b759929f2431733a277670efed7109ad232aff8dfcb98a389555ff6` |
| `prices_limit_code_change_fixes` | `cr139-w2-prices_limit_code_change_fixes-legacy_lake-20260629-canonical` | 1,129 | 0 | `sha256:b19fced2b7a6b854dd930f63c73e558cb70e6932a5b50bb5c1600692d7acf9fc` |
| `prices_limit_coverage_exclusions` | `cr139-w2-prices_limit_coverage_exclusions-legacy_lake-20260629-canonical` | 91,018 | 0 | `sha256:347ab08055626793d1ec5ac90f71389dedac1d5eb15ab89a361ded8661fa5bfc` |
| `stock_basic` | `cr139-w2-stock_basic-legacy_lake-20260629-canonical` | 43,558 | 0 | `sha256:0e6bb57db5ec0c8d6a073de99f54559e52c965ba945138c91ddbc4706546974b` |
| `trade_calendar` | `cr139-w2-trade_calendar-legacy_lake-20260629-canonical` | 10,579 | 0 | `sha256:63e70c9477ecf17de7ef7bb98b119ed99347ae35db46d16874124a9fd857737e` |
| `trade_status` | `cr139-w2-trade_status-legacy_lake-20260629-canonical` | 13,957,403 | 7,612,504 | `sha256:1384e09cd9432c0b54a96e04fe3e14a64ce87bb888c2f70fcb3d98ad8bf7ebcf` |

## Active Catalog Virtual Diff

The exact virtual before/after diff is written to `process/evidence/CR139-W2-GATEC-STAGED-ACTIVE-CATALOG-VIRTUAL-DIFF-2026-06-29.diff`. It is a preview artifact only; it was not applied to `/home/hyde/data/quant-lab/data-lake/catalog/catalog.json`.

## Authorization Boundary For Future Gate C Write

A future authorization may allow writing staged metadata only. It must still forbid active `catalog/catalog.json` refresh, provider catalog/provider-lake-catalog writes, published pointer advance, physical migration, legacy deletion, rollback deletion, NAS/runtime/credential/trading operations, and Git remote writes unless separately approved.

### Required Staged Write Location

A future staged metadata write must use an isolated lake staged namespace and must not touch active metadata files.

| Item | Required value |
|---|---|
| Allowed staged root | `catalog/staged/cr139-w2/gatec/` |
| Allowed staged catalog file | `catalog/staged/cr139-w2/gatec/staged_candidate_catalog.v1.2026-06-29.json` |
| Allowed staged manifest file | `catalog/staged/cr139-w2/gatec/staged_candidate_manifest_records.v1.2026-06-29.jsonl` |
| Forbidden active catalog file | `catalog/catalog.json` |
| Forbidden active manifest file | `manifest/market_data_manifest.jsonl` |

Post-write validation must prove both active file hashes remain unchanged:

| Active file | Expected unchanged SHA-256 |
|---|---|
| `catalog/catalog.json` | `3c9e937acb068a2d838d52e67f6990d4415306c17ba04ba4a8038adf7a7810f0` |
| `manifest/market_data_manifest.jsonl` | `57c5f86a7e170b99407005f54c6d66f3903fcb30a2fb0f573d8966e96eedae7a` |

### Gate E Path Strategy Dependency

The virtual diff is a warning input for later gates, not an authorization target for this staged write. It shows that an active catalog refresh would replace 17 `canonical_path` values from legacy canonical paths with CR139 candidate paths. That path semantics change is blocked until D-GATEB-007 is decided: full 17-dataset migration versus publish-target-only plus explicit waiver. Active catalog refresh requires a separate Gate C-2 authorization after the Gate E path strategy and reader exposure review are complete.

## Artifacts

| Artifact | Path |
|---|---|
| Preview JSON | `process/evidence/CR139-W2-GATEC-STAGED-CATALOG-MANIFEST-PREVIEW-2026-06-29.json` |
| Evidence index | `process/evidence/CR139-W2-GATEC-STAGED-CATALOG-MANIFEST-PREVIEW.index.json` |
| Active catalog virtual diff | `process/evidence/CR139-W2-GATEC-STAGED-ACTIVE-CATALOG-VIRTUAL-DIFF-2026-06-29.diff` |

## Boundary Proof

Operation counts remain zero for lake catalog write, lake manifest append, provider catalog write, provider-lake-catalog write, published pointer advance, physical partition migration, and current truth changes.
