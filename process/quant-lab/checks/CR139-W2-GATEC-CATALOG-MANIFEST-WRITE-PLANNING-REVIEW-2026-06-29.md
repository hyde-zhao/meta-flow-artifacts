# CR139 W2 Gate C Catalog/Manifest Write Planning Review - 2026-06-29

## Decision

BLOCKED_FOR_ACTIVE_CATALOG_JSON_WRITE / READY_FOR_STAGED_METADATA_PREVIEW

Gate B candidate objects are complete enough to prepare Gate C metadata previews, but this review does not support direct writes to the active lake catalog file `catalog/catalog.json`. No lake catalog write, manifest append, provider catalog/provider-lake-catalog write, published pointer advance, physical partition migration, NAS/runtime/credential operation, Git remote write, legacy deletion, or rollback deletion was executed.

## Mechanical Summary

| Check | Result |
|---|---|
| Candidate datasets discovered | 17 |
| Active catalog datasets | 17 |
| Candidate dataset set matches active catalog dataset set | True |
| Candidate parquet objects | 24 (17 main, 7 quarantine) |
| Active catalog published=true entries | 0 |
| Active catalog entries lacking `lineage_checksum` | 17 |
| Active manifest line count | 6367 |

## Findings

| Severity | ID | Finding | Recommendation |
|---|---|---|---|
| BLOCKING | GATEC-001 | Direct active catalog.json write can expose unpublished candidate paths through legacy readers | Do not overwrite catalog/catalog.json with candidate paths in this gate. Use a staged candidate catalog namespace or wait until final path/publish gate is ready. |
| BLOCKING | GATEC-002 | Gate C precondition for final physical paths is not satisfied or explicitly waived | Decide Gate E scope before any active catalog/path refresh, or write only staged metadata that is clearly not current truth. |
| HIGH | GATEC-003 | build_catalog_manifest_pair is a current-pointer shape and fails unpublished candidate payloads | For candidate metadata, use manifest-only preview plus staged catalog records, or add a candidate catalog/manifest contract before lake catalog writes. |
| MEDIUM | GATEC-004 | CatalogEntry has canonical_path but no candidate_path field, so active catalog cannot represent old canonical and new candidate paths at the same time | Do not overload canonical_path with candidate paths in active catalog.json. Store candidate_path in a staged Gate C plan/manifest record until publish path is chosen. |

## Candidate Metadata Preview

| Dataset | Candidate run_id | Main rows | Quarantine rows | Main objects | Quarantine objects | Candidate lineage checksum |
|---|---|---:|---:|---:|---:|---|
| `adj_factor` | `cr139-w2-adj_factor-legacy_lake-20260629-canonical` | 17,905,960 | 321,815 | 1 | 1 | `sha256:d0383f29c8c34045932132ded7f05d0c14582dec9bb6ef15a3540c9a7eaae59e` |
| `bse_code_mapping` | `cr139-w2-bse_code_mapping-legacy_lake-20260629-canonical` | 248 | 0 | 1 | 0 | `sha256:90f2ed108bb0f186acf4d5b974163b56a02359f5150891c89858819031a4beb2` |
| `events` | `cr139-w2-events-legacy_lake-20260629-canonical` | 350,753 | 12,134 | 1 | 1 | `sha256:a77349b7e68602c2c974f7f3877eec1fc1c1bdd1d052a17eff5abe8295d2e93d` |
| `hs300_index` | `cr139-w2-hs300_index-legacy_lake-20260629-canonical` | 11,511 | 0 | 1 | 0 | `sha256:5af0d1350e22494c550e02a9451dda1d099bb1b7acdf4eb79e71b4ed1af71afd` |
| `index_members` | `cr139-w2-index_members-legacy_lake-20260629-canonical` | 283,500 | 0 | 1 | 0 | `sha256:2bcdf10a4598425f6c3d235ccfcd08deb712cfbafd5529794be1e262342508f0` |
| `index_weights` | `cr139-w2-index_weights-legacy_lake-20260629-canonical` | 283,800 | 0 | 1 | 0 | `sha256:011de5203330baf7c381414389c0418623dd8b7c5d07b0b01faf167a2f736e9a` |
| `industry_classification` | `cr139-w2-industry_classification-legacy_lake-20260629-canonical` | 11,049 | 0 | 1 | 0 | `sha256:fa19684ef5a76e3fd516b4927b8b4c331b56e12b16eb4e0dc635ac376f9aa277` |
| `lifecycle_code_change` | `cr139-w2-lifecycle_code_change-legacy_lake-20260629-canonical` | 250 | 0 | 1 | 0 | `sha256:b724ed07a428b853d4759d529a574de53ee14c352efd7384b1dcec800656d269` |
| `liquidity_capacity` | `cr139-w2-liquidity_capacity-legacy_lake-20260629-resolved` | 17,093,658 | 1,830 | 1 | 1 | `sha256:a409573d5cb5264638df52c136249ce14ca33777ff13b44e163cdfc8e069d576` |
| `market_cap` | `cr139-w2-market_cap-legacy_lake-20260629-resolved` | 17,001,327 | 106,776 | 1 | 1 | `sha256:bebc76b3ce94e48929a9f1adb8bdf4db7faab9742182d620db7a713af95f8ca5` |
| `prices` | `cr139-w2-prices-legacy_lake-20260629-canonical` | 17,013,700 | 401,628 | 1 | 1 | `sha256:ab4f643cb535926d6c7301fe4f8e32671d819a52163f3acd7c04cf940b0bfa21` |
| `prices_limit` | `cr139-w2-prices_limit-legacy_lake-20260629-canonical` | 19,210,017 | 11,841,929 | 1 | 1 | `sha256:2380985a64f25d92dd6414a3077d8249ff530e83c8eadb491aa8408ec332f73b` |
| `prices_limit_code_change_fixes` | `cr139-w2-prices_limit_code_change_fixes-legacy_lake-20260629-canonical` | 1,129 | 0 | 1 | 0 | `sha256:9a0423aa9a9eadcac030150f8c03106262f7fe582ba5357ea9cedbc81ce9a6c5` |
| `prices_limit_coverage_exclusions` | `cr139-w2-prices_limit_coverage_exclusions-legacy_lake-20260629-canonical` | 91,018 | 0 | 1 | 0 | `sha256:d2ada0400e1f0ac9d7f1ef2fdae003e6d710aa77946f8246cd91eda810c90ba7` |
| `stock_basic` | `cr139-w2-stock_basic-legacy_lake-20260629-canonical` | 43,558 | 0 | 1 | 0 | `sha256:520afacf69a1b9ac234dd0a471286ab3e93a4c31b60fcd9690b48d5bc3d13e77` |
| `trade_calendar` | `cr139-w2-trade_calendar-legacy_lake-20260629-canonical` | 10,579 | 0 | 1 | 0 | `sha256:d735b3c7de695a06dde25a606641e746ac5066f8bf94bac224a6918659d11f40` |
| `trade_status` | `cr139-w2-trade_status-legacy_lake-20260629-canonical` | 13,957,403 | 7,612,504 | 1 | 1 | `sha256:34660ab427deed2b17610f8d4fb58c72a377e63676e520a047e544f058003fe2` |

## Review Notes

- Gate C plan preconditions require reviewed source-of-truth diff, verified lineage checksum, and catalog/manifest records pointing to final physical paths. The current objects are under `candidate/parquet/...`, while Gate E final path scope is not yet complete or explicitly waived.
- `CatalogStore.upsert` writes the active file `catalog/catalog.json`. Several readers still consume `CatalogStore.get()` or raw `canonical_path` rather than only `get_published_current_pointer()`, so replacing active `canonical_path` with candidate paths would be observable before Gate D.
- `build_catalog_manifest_pair` builds a `catalog_pointer` plus a manifest from one payload. It is suitable for published/current pointer shaped records; it fails unpublished candidate payloads that intentionally have no `published_at`, `universe_scope`, or `as_of_trade_date`.
- `derive_manifest_from_catalog` can produce manifest-shaped candidate metadata if a `candidate_path` is supplied, but the active catalog dataclass does not preserve a separate `candidate_path` field. This is why staged metadata is the safe next step.

## Recommended Next Step

Generate a no-write Gate C staged catalog/manifest preview under `process/evidence` first: 17 candidate manifest records, a staged candidate catalog shape, lineage checksum validation, and exact active-catalog diff. After that, choose one of these authorization routes:

1. Staged Gate C write only: write append-only/staged metadata that is not consumed as current truth; keep `catalog/catalog.json` unchanged.
2. Active catalog refresh later: only after Gate E final path decision is complete or waived, and after reader exposure is reviewed.
3. If speed is prioritized, explicitly waive Gate E path migration for the publish target and accept that Gate D would publish paths under the reviewed candidate/staged location. This requires a separate risk decision.

## Boundary

Still not authorized and not executed: active catalog write, manifest append to lake, provider catalog/provider-lake-catalog write, published pointer advance, physical migration, NAS, credential read, runtime/QMT/gateway/trading, Git remote write, legacy deletion, rollback deletion.

Machine index: `process/evidence/CR139-W2-GATEC-CATALOG-MANIFEST-WRITE-PLANNING-REVIEW.index.json`
