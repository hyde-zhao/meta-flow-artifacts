# CR139 W3-B Publish Guard

## Result

- status: `pass_w3b_publish_guard`
- failed_checks: `[]`
- active catalog unchanged: `True`
- active manifest unchanged: `True`

## Checks

| check | result |
|---|---:|
| active_catalog_exists | True |
| active_catalog_unchanged | True |
| active_manifest_exists | True |
| active_manifest_unchanged | True |
| canonical_path_no_candidate_leak_17 | True |
| canonical_row_count_matches_17 | True |
| dataset_count_17 | True |
| no_provider_or_nas_implicit_operation | True |
| pit_purity_pass | True |
| pointer_contract_valid_17 | True |
| published_pointer_available_17 | True |
| published_true_17 | True |
| quality_paths_exist | True |
| reader_required_p0_supported | True |
| reader_support_matrix_pass | True |

## Reader Support Matrix

| dataset | contract_supported | required_reader_support | pointer_only_allowed | default_status | allow_warn_status | row_count | conditional_reader_support | reader_guard_passed |
|---|---:|---:|---:|---|---|---:|---:|---:|
| adj_factor | True | False | False | not_executed | not_executed | 17905960 | False | True |
| bse_code_mapping | False | False | True | not_executed | not_executed | None | False | True |
| events | True | False | False | available | available | 350753 | False | True |
| hs300_index | True | False | False | available | available | 11511 | False | True |
| index_members | True | False | False | available | available | 283500 | False | True |
| index_weights | True | False | False | available | available | 283500 | False | True |
| industry_classification | True | False | False | unavailable | available | 11049 | True | True |
| lifecycle_code_change | False | False | True | not_executed | not_executed | None | False | True |
| liquidity_capacity | True | True | False | not_executed | not_executed | 17093658 | False | True |
| market_cap | True | True | False | not_executed | not_executed | 17001327 | False | True |
| prices | True | False | False | not_executed | not_executed | 17013700 | False | True |
| prices_limit | True | False | False | not_executed | not_executed | 19210017 | False | True |
| prices_limit_code_change_fixes | True | False | False | available | available | 1129 | False | True |
| prices_limit_coverage_exclusions | True | False | False | available | available | 91018 | False | True |
| stock_basic | True | False | False | available | available | 38930 | False | True |
| trade_calendar | True | False | False | available | available | 10579 | False | True |
| trade_status | True | False | False | not_executed | not_executed | 13957403 | False | True |

## PIT Validation

| dataset | applicable | passed | distribution |
|---|---:|---:|---|
| adj_factor | False | True | {} |
| bse_code_mapping | False | True | {} |
| events | False | True | {} |
| hs300_index | False | True | {} |
| index_members | True | True | {'pit_available': 283500} |
| index_weights | True | True | {'pit_available': 283500} |
| industry_classification | False | True | {} |
| lifecycle_code_change | False | True | {} |
| liquidity_capacity | False | True | {} |
| market_cap | False | True | {} |
| prices | False | True | {} |
| prices_limit | False | True | {} |
| prices_limit_code_change_fixes | False | True | {} |
| prices_limit_coverage_exclusions | False | True | {} |
| stock_basic | True | True | {'pit_available': 38930} |
| trade_calendar | False | True | {} |
| trade_status | False | True | {} |

## Operation Counts

```json
{
  "active_catalog_write": 0,
  "active_manifest_append": 0,
  "credential_print_or_persist": 0,
  "git_remote": 0,
  "lake_data_write": 0,
  "lake_delete": 0,
  "nas_operation": 0,
  "physical_partition_migration": 0,
  "provider_catalog_write": 0,
  "provider_lake_catalog_write": 0,
  "published_pointer_advance": 0,
  "runtime_operation": 0
}
```
