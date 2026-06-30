---
checkpoint_id: "GATEB-CR139-W2-BATCH2-DEEP-PROFILE"
checkpoint_name: "CR139 W2 Gate B Batch 2 Consolidated Read-Only Deep Profile"
type: "runtime_gate_auto_check"
status: "PASS_WITH_RISK"
owner: "host-orchestrator"
created_at: "2026-06-29T17:30:02+08:00"
checked_at: "2026-06-29T17:35:10+08:00"
target:
  phase: "story-execution"
  gate: "Gate B"
  datasets: ["liquidity_capacity", "market_cap", "events", "trade_status", "adj_factor", "prices", "prices_limit"]
manual_checkpoint: "user authorized Batch 2 consolidated read-only deep profile only; no lake write"
---

# CR139 W2 Gate B Batch 2 Consolidated Read-Only Deep Profile Check

## Entry Criteria

| Item | Status | Evidence | Notes |
|---|---|---|---|
| Explicit read-only authorization exists | PASS | User chat authorization | No lake/catalog/pointer/migration write authorized. |
| Process route health checked | PASS | `meta-flow workspace check` | process symlink health OK. |
| Write plan covers all 7 Batch 2 datasets | PASS | `process/plans/CR139-W2-GATEB-PER-DATASET-WRITE-PLAN.json` | All target datasets are Batch 2. |
| Candidate target roots absent before profile | PASS | `process/evidence/CR139-W2-GATEB-BATCH2-DEEP-PROFILE-2026-06-29T173002+0800.json` | Profile did not create target roots. |

## Checklist

| Check | Status |
|---|---|
| `authorized_scope_is_read_only` | PASS |
| `dataset_count_7` | PASS |
| `no_profile_failures` | PASS |
| `all_row_counts_match_write_plan` | PASS |
| `all_targets_absent_before_write` | PASS |
| `catalog_hash_unchanged` | PASS |
| `published_tree_hash_unchanged` | PASS |
| `canonical_tree_hash_unchanged` | PASS |
| `candidate_tree_hash_unchanged` | PASS |
| `operation_counts_zero` | PASS |
| `all_batch2_remain_one_dataset_authorization` | PASS |

## Dataset Profile Summary

| Dataset | Rows | Batch0 Duplicate Rows Over Unique | Full-Row Fingerprint Duplicate Rows Over Unique | Quarantine Required | Recommended Action | Readiness Verdict |
|---|---:|---:|---:|---|---|---|
| `liquidity_capacity` | 17095488 | 1830 | 0 | true | `quarantine_metadata_conflict_key_duplicates_before_candidate_write` | `blocked_requires_one_dataset_quarantine_authorization` |
| `market_cap` | 17108103 | 106776 | 0 | true | `quarantine_metadata_conflict_key_duplicates_before_candidate_write` | `blocked_requires_one_dataset_quarantine_authorization` |
| `events` | 362887 | 11360 | 0 | true | `quarantine_duplicate_primary_keys_before_candidate_write` | `blocked_requires_events_quarantine_write_authorization` |
| `trade_status` | 21569907 | 3920399 | 79870 | true | `quarantine_metadata_conflict_key_duplicates_before_candidate_write` | `blocked_requires_one_dataset_quarantine_authorization` |
| `adj_factor` | 29970830 | 12064870 | 11907842 | true | `quarantine_metadata_conflict_key_duplicates_before_candidate_write` | `blocked_requires_one_dataset_quarantine_authorization` |
| `prices` | 28646730 | 11553072 | 11470750 | true | `quarantine_metadata_conflict_key_duplicates_before_candidate_write` | `blocked_requires_one_dataset_quarantine_authorization` |
| `prices_limit` | 40962525 | 21252721 | 13728564 | true | `quarantine_metadata_conflict_key_duplicates_before_candidate_write` | `blocked_requires_one_dataset_quarantine_authorization` |

## Totals

| Metric | Value |
|---|---:|
| row_count_total | 155716470 |
| batch0_duplicate_key_over_unique_total | 48911028 |
| full_row_fingerprint_duplicate_over_unique_total | 37187026 |
| quarantine_required_count | 7 |
| exact_dedup_only_candidate_count | 0 |
| copy_candidate_count | 0 |

## Machine Consumer Notes

- `conflict_share_of_duplicate_key_excess` is now populated as a machine-readable alias for `estimated_conflict_share_of_duplicate_key_excess` on all seven Batch 2 datasets.
- `trade_status` now uses required exact-bucket wording; its exact-copy bucket requires an explicit decision, with exact-dedup recommended.

## Method Note

The full-row scan uses per-row pandas 64-bit fingerprints with NumPy sort to make the 155M-row read-only profile feasible in one pass. `prices` required a union-column normalization repair because its canonical root contains schema variants; missing columns were treated as NA for the planning profile. Future write gates must revalidate the selected dataset's exact candidate content immediately before writing.

## Operation Counts

| Operation | Count |
|---|---:|
| provider_fetch | 0 |
| lake_write | 0 |
| catalog_write | 0 |
| current_pointer_publish | 0 |
| physical_partition_migration | 0 |
| credential_read | 0 |
| nas_operation | 0 |
| runtime_operation | 0 |
| git_remote_write | 0 |

## Refined Risk Buckets

| Dataset | Dup-key over unique | Full-row dup over unique | Estimated non-identical same-key conflicts | Conflict share | Risk class | Refined action |
|---|---:|---:|---:|---:|---|---|
| `liquidity_capacity` | 1,830 | 0 | 1,830 | 100.0% | `pure_metadata_conflict` | `quarantine_metadata_conflict_key_groups_only` |
| `market_cap` | 106,776 | 0 | 106,776 | 100.0% | `pure_metadata_conflict` | `quarantine_metadata_conflict_key_groups_only` |
| `events` | 11,360 | 0 | 11,360 | 100.0% | `primary_key_conflict_quarantine` | `quarantine_duplicate_primary_key_groups_only` |
| `trade_status` | 3,920,399 | 79,870 | 3,840,529 | 98.0% | `mostly_metadata_conflict_with_required_exact_bucket_decision` | `quarantine_metadata_conflict_key_groups_and_exact_dedup_proven_exact_copy_bucket` |
| `adj_factor` | 12,064,870 | 11,907,842 | 157,028 | 1.3% | `mostly_exact_duplicates_with_small_conflict_bucket` | `exact_dedup_main_bucket_plus_quarantine_conflict_bucket` |
| `prices` | 11,553,072 | 11,470,750 | 82,322 | 0.7% | `mostly_exact_duplicates_with_small_conflict_bucket` | `exact_dedup_main_bucket_plus_quarantine_conflict_bucket` |
| `prices_limit` | 21,252,721 | 13,728,564 | 7,524,157 | 35.4% | `mixed_exact_duplicates_and_metadata_conflicts` | `exact_dedup_bucket_plus_quarantine_conflict_bucket` |

- `adj_factor` and `prices` should not be treated as full-table quarantine candidates; their dominant duplicate bucket is exact-copy dedup, with a smaller conflict bucket requiring quarantine.
- `prices_limit` is genuinely mixed and needs split exact-dedup plus quarantine bucket materialization.
- `liquidity_capacity`, `market_cap`, and `events` are pure/primary-key conflict routing candidates with zero full-row duplicate fingerprints.
- `prices` additionally requires an explicit schema-normalization decision before write authorization.

## Decision Implications

- This gate is profile-only. It does not authorize any Batch 2 write.
- Batch 2 remains one-dataset-per-authorization. Future authorization must name exactly one dataset and the quarantine/dedup/copy mode.
- Quarantine physical shape is planned under each dataset's CR139 candidate path `quarantine/`; legacy canonical paths remain read-only.
- `prices` also needs an explicit schema-normalization decision because source files have schema variants.

## Deliverables

| Deliverable | Path | Status |
|---|---|---|
| Detail evidence | `process/evidence/CR139-W2-GATEB-BATCH2-DEEP-PROFILE-2026-06-29T173002+0800.json` | PASS |
| Prices union-column repair evidence | `process/evidence/CR139-W2-GATEB-BATCH2-DEEP-PROFILE-PRICES-UNION-REPAIR-2026-06-29T173510+0800.json` | PASS |
| Evidence index | `process/evidence/CR139-W2-GATEB-BATCH2-DEEP-PROFILE.index.json` | PASS |

## Conclusion

- Conclusion: `PASS_WITH_RISK_BATCH2_DEEP_PROFILE_COMPLETE_DECISIONS_REQUIRED`
- Still not authorized: Batch 2 lake write, catalog/provider catalog/provider-lake-catalog write, published pointer advance, physical migration, credential read, NAS/runtime/QMT/MiniQMT/gateway/trading/live, Git remote, rollback deletion.
- Recommended next step: review the profile and choose exactly one Batch 2 dataset for explicit candidate write authorization, with quarantine/dedup/schema-normalization mode named in the authorization.
