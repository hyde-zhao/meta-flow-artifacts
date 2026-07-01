# CR146 Production Lake Readiness Matrix - 2026-07-01

## Summary

| Metric | Value |
|---|---:|
| dataset_count | 17 |
| production_ready_count | 0 |
| research_ready_count | 17 |
| quarantined_count | 0 |
| unsupported_count | 0 |
| current_truth_duplicate_key_total | 0 |
| current_truth_row_count_total | 103264532 |
| historical_root_business_conflict_group_count | 4272624 |
| n1_copy_first_candidate_count | 17 |
| physical_missing_count | 0 |

## Dataset Matrix

| Dataset | Status | Current Rows | Current Dup Keys | Historical Business Conflicts | PIT | N1 Action | Blockers | Warnings |
|---|---|---:|---:|---:|---|---|---|---|
| adj_factor | research_ready | 17905960 | 0 | 0 | pit_available_by_available_at_column | copy_first_to_current | - | n1_current_layout_migration_pending, historical_dedup_policy_gate_required |
| bse_code_mapping | research_ready | 248 | 0 | 0 | not_applicable_or_needs_classification | copy_first_to_current | - | n1_current_layout_migration_pending |
| events | research_ready | 350753 | 0 | 774 | pit_available_by_available_at_column | copy_first_to_current | - | n1_current_layout_migration_pending, historical_business_conflict_cleanup_gate_required, historical_dedup_policy_gate_required |
| hs300_index | research_ready | 11511 | 0 | 0 | pit_available_by_available_at_column | copy_first_to_current | - | n1_current_layout_migration_pending |
| index_members | research_ready | 283500 | 0 | 0 | pit_available | copy_first_to_current | - | n1_current_layout_migration_pending |
| index_weights | research_ready | 283500 | 0 | 0 | pit_available | copy_first_to_current | - | n1_current_layout_migration_pending |
| industry_classification | research_ready | 11049 | 0 | 0 | pit_available_by_available_at_column | copy_first_to_current | - | n1_current_layout_migration_pending |
| lifecycle_code_change | research_ready | 250 | 0 | 0 | not_applicable_or_needs_classification | copy_first_to_current | - | n1_current_layout_migration_pending |
| liquidity_capacity | research_ready | 17093658 | 0 | 0 | pit_available_by_available_at_column | copy_first_to_current | - | n1_current_layout_migration_pending |
| market_cap | research_ready | 17001327 | 0 | 0 | pit_available_by_available_at_column | copy_first_to_current | - | n1_current_layout_migration_pending |
| prices | research_ready | 17013700 | 0 | 79958 | pit_available_by_available_at_column | copy_first_to_current | - | n1_current_layout_migration_pending, historical_business_conflict_cleanup_gate_required, historical_dedup_policy_gate_required |
| prices_limit | research_ready | 19210017 | 0 | 499787 | pit_available_by_available_at_column | copy_first_to_current | - | n1_current_layout_migration_pending, historical_business_conflict_cleanup_gate_required, historical_dedup_policy_gate_required |
| prices_limit_code_change_fixes | research_ready | 1129 | 0 | 0 | not_applicable_or_needs_classification | copy_first_to_current | - | n1_current_layout_migration_pending |
| prices_limit_coverage_exclusions | research_ready | 91018 | 0 | 0 | not_applicable_or_needs_classification | copy_first_to_current | - | n1_current_layout_migration_pending |
| stock_basic | research_ready | 38930 | 0 | 0 | pit_available | copy_first_to_current | - | n1_current_layout_migration_pending |
| trade_calendar | research_ready | 10579 | 0 | 0 | pit_available_by_available_at_column | copy_first_to_current | - | n1_current_layout_migration_pending |
| trade_status | research_ready | 13957403 | 0 | 3692105 | pit_available_by_available_at_column | copy_first_to_current | - | n1_current_layout_migration_pending, historical_business_conflict_cleanup_gate_required, historical_dedup_policy_gate_required |

## Interpretation

- Catalog current truth has zero duplicate keys across 17 published datasets; historical-root duplicate/conflict evidence applies to cleanup/rewrite policy, not to the current catalog read path.
- The remaining production-grade blocker is structural: 17/17 datasets still use legacy `run_id=` physical layout and require copy-first N1 migration plus catalog pointer updates.
- N1 migration, catalog pointer updates, historical cleanup/rewrite, legacy delete/archive, and NAS sync require human gate before execution.
