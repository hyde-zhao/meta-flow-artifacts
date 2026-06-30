# CR141 Phase 1 Baseline Failure Classification

## Summary

CR141 Phase 0 froze 46 inherited pytest failures. Phase 1 classifies them by remediation owner and sequencing. The guiding rule is: fix low-risk static/guardrail/code debt in CR141, but defer failures that materially overlap remaining experiment engine convergence to `FU-CR140-002`.

## Bucket Summary

| Bucket | Count | Classification | CR141 action |
|---|---:|---|---|
| CR008 research input / auxiliary contract | 6 | Experiment-adjacent contract/static debt | Inspect and fix if local/static; no real lake/provider. |
| CR011 experiment report / boundary claims | 6 | Experiment-adjacent report and boundary debt | Inspect and fix if local/static. |
| CR013 backfill / claim boundary docs and reports | 13 | Data-governance documentation/static debt | Fix only text/fixture/static contract drift; no real operation. |
| CR015/CR016 sensitive raw output runbook debt | 3 | Security/redaction documentation debt | Fix static text/output contract drift. |
| CR019/CR020/CR045 runtime no-authorization docs | 9 | Runtime boundary documentation debt | Fix static no-authorization drift only; no runtime. |
| CR131 design surface | 2 | Process/design surface drift | Fix static process artifact drift only. |
| Remaining experiment suites 17_21 / 23_29 | 2 | E3 overlap, remaining experiment engine convergence | Defer to `FU-CR140-002` unless a tiny fixture-only fix is proven independent. |
| market_data imports engine family | 1 | Architecture boundary code debt | Inspect import graph; fix only if isolated and no runtime. |
| market_data tushare comparison docs | 1 | Static data-source boundary docs debt | Fix static docs/fixture drift only. |
| script entrypoint naming | 1 | CR139 root script naming debt | Fix stable-entrypoint registration if isolated. |
| STORY-004/013 loader/logging | 2 | Legacy offline loader / logging quality debt | Inspect and fix local fixture/code drift if isolated. |

Total: 46.

## Detailed Classification

| # | Failure | Bucket | Initial action |
|---:|---|---|---|
| 1 | `tests/test_cr008_factor_auxiliary_data_contract.py::test_t08_reader_helper_returns_typed_missing_and_never_auto_executes` | CR008 research input / auxiliary contract | Inspect static contract drift. |
| 2 | `tests/test_cr008_factor_auxiliary_data_contract.py::test_t11_experiment_15_schema_report_and_summary_include_auxiliary_contract` | CR008 research input / auxiliary contract | Inspect report/schema text drift. |
| 3 | `tests/test_cr008_quality_adjustment_label_gates.py::test_adjustment_policy_mixed_missing_or_mismatch_fails[missing-adjustment_policy_missing]` | CR008 research input / auxiliary contract | Inspect label gate contract. |
| 4 | `tests/test_cr008_quality_adjustment_label_gates.py::test_adjustment_policy_mixed_missing_or_mismatch_fails[mismatch-adjustment_policy_mismatch]` | CR008 research input / auxiliary contract | Inspect label gate contract. |
| 5 | `tests/test_cr008_research_input_metadata.py::test_experiment_cli_data_dir_must_be_explicit` | CR008 research input / auxiliary contract | Inspect CLI metadata expectation. |
| 6 | `tests/test_cr008_research_input_metadata.py::test_no_connector_runtime_storage_imports_or_credentials_in_target_files` | CR008 research input / auxiliary contract | Inspect forbidden import/static scan. |
| 7 | `tests/test_cr011_adjustment_audit.py::test_s05_forbidden_boundaries_are_static_and_no_secret_leakage` | CR011 report / boundary claims | Inspect static secret-leakage scan. |
| 8 | `tests/test_cr011_benchmark_policy_consumption.py::test_experiment_production_strict_missing_benchmark_has_blocked_claims_without_proxy` | CR011 report / boundary claims | Inspect report claim boundary. |
| 9 | `tests/test_cr011_benchmark_policy_consumption.py::test_old_experiment_report_path_is_rejected_before_writes` | CR011 report / boundary claims | Inspect old path fail-fast behavior. |
| 10 | `tests/test_cr011_capacity_cost_sensitivity.py::test_experiment_metadata_uses_s07_contract_without_old_report_overwrite` | CR011 report / boundary claims | Inspect metadata contract drift. |
| 11 | `tests/test_cr011_exposure_claims.py::test_s06_forbidden_boundaries_are_static_and_no_secret_leakage` | CR011 report / boundary claims | Inspect static secret-leakage scan. |
| 12 | `tests/test_cr011_factor_panel_robust_validation.py::test_s08_forbidden_boundaries_are_static_and_legacy_report_fails_fast` | CR011 report / boundary claims | Inspect legacy report fail-fast behavior. |
| 13 | `tests/test_cr013_backfill_roadmap_boundaries.py::test_backfill_roadmap_lists_full_history_and_vwap_release_criteria` | CR013 backfill / claim boundary | Fix static roadmap drift if local. |
| 14 | `tests/test_cr013_backfill_roadmap_boundaries.py::test_backfill_roadmap_current_real_operations_are_not_authorized` | CR013 backfill / claim boundary | Fix no-authorization wording if local. |
| 15 | `tests/test_cr013_backfill_roadmap_boundaries.py::test_backfill_roadmap_contains_no_direct_execution_commands_or_secret_prompts` | CR013 backfill / claim boundary | Fix static command/secret wording if local. |
| 16 | `tests/test_cr013_backfill_roadmap_boundaries.py::test_backfill_roadmap_forbidden_operation_counters_are_zero` | CR013 backfill / claim boundary | Fix static counter drift if local. |
| 17 | `tests/test_cr013_execution_vwap_claim_boundary.py::test_execution_audit_resolves_real_vwap_and_fill_as_blocked` | CR013 backfill / claim boundary | Fix blocked-claim evidence if local. |
| 18 | `tests/test_cr013_execution_vwap_claim_boundary.py::test_amount_volume_derived_vwap_attempt_is_not_allowed` | CR013 backfill / claim boundary | Fix claim boundary if local. |
| 19 | `tests/test_cr013_execution_vwap_claim_boundary.py::test_execution_claim_metadata_and_report_preserve_boundaries` | CR013 backfill / claim boundary | Fix static metadata/report drift. |
| 20 | `tests/test_cr013_full_history_gap_register.py::test_full_history_gap_register_has_exact_ten_formal_datasets` | CR013 backfill / claim boundary | Fix register contents if local. |
| 21 | `tests/test_cr013_full_history_gap_register.py::test_full_history_summary_blocks_2020_2024_claim_and_preserves_old_evidence` | CR013 backfill / claim boundary | Fix summary/evidence wording if local. |
| 22 | `tests/test_cr013_full_history_gap_register.py::test_full_history_forbidden_operation_counters_are_zero` | CR013 backfill / claim boundary | Fix static counter drift if local. |
| 23 | `tests/test_cr013_unsupported_register_claim_boundary.py::test_unsupported_register_exact_rows_and_excluded_denominator` | CR013 backfill / claim boundary | Fix unsupported register if local. |
| 24 | `tests/test_cr013_unsupported_register_claim_boundary.py::test_claim_boundary_summary_merges_s01_s02_and_excludes_denominator` | CR013 backfill / claim boundary | Fix summary merge if local. |
| 25 | `tests/test_cr013_unsupported_register_claim_boundary.py::test_docs_and_report_show_supported_research_only_unsupported_blocked_boundaries` | CR013 backfill / claim boundary | Fix docs/report boundary if local. |
| 26 | `tests/test_cr015_foundation_runbook_boundary.py::test_sensitive_value_output_count_is_zero` | CR015/CR016 sensitive output | Fix static sensitive output wording. |
| 27 | `tests/test_cr016_docs_incident_playbooks.py::test_sensitive_raw_value_output_count_is_zero` | CR015/CR016 sensitive output | Fix static sensitive output wording. |
| 28 | `tests/test_cr016_runbook_approval_gates.py::test_sensitive_raw_value_output_count_is_zero` | CR015/CR016 sensitive output | Fix static sensitive output wording. |
| 29 | `tests/test_cr019_deferred_capabilities.py::test_readme_declares_later_gated_non_p0_boundary_without_current_enablement` | CR019/CR020/CR045 runtime boundary docs | Fix static README boundary drift. |
| 30 | `tests/test_cr019_deferred_capabilities.py::test_dependency_and_scope_expansion_counters_stay_zero_in_contract` | CR019/CR020/CR045 runtime boundary docs | Fix static counter drift. |
| 31 | `tests/test_cr019_qmt_gateway_fallback.py::test_incident_playbook_cr019_section_freezes_manual_only_boundary` | CR019/CR020/CR045 runtime boundary docs | Fix static playbook wording. |
| 32 | `tests/test_cr019_qmt_gateway_lifecycle.py::test_install_doc_uses_placeholders_and_contains_no_sensitive_literals` | CR019/CR020/CR045 runtime boundary docs | Fix placeholder wording. |
| 33 | `tests/test_cr020_docs_runbook_no_authorization.py::test_cr020_manual_install_sections_and_runtime_commands_are_present` | CR019/CR020/CR045 runtime boundary docs | Fix static runbook drift. |
| 34 | `tests/test_cr020_docs_runbook_no_authorization.py::test_cr020_docs_no_authorization_table_blocks_trading_and_writes` | CR019/CR020/CR045 runtime boundary docs | Fix no-authorization table. |
| 35 | `tests/test_cr020_docs_runbook_no_authorization.py::test_cr020_cp7_evidence_schema_excludes_raw_positions_and_secret_output` | CR019/CR020/CR045 runtime boundary docs | Fix CP7 evidence schema/static docs. |
| 36 | `tests/test_cr020_server_qmt_login_session.py::test_env_example_is_placeholder_only_and_contains_no_real_values` | CR019/CR020/CR045 runtime boundary docs | Fix `.env.example` placeholder contract if in scope. |
| 37 | `tests/test_cr045_goldminer_no_operation_static.py::test_runbook_does_not_contain_positive_runtime_authorization_claims` | CR019/CR020/CR045 runtime boundary docs | Fix static authorization wording. |
| 38 | `tests/test_cr131_design_surface.py::test_cr131_design_surface_current_docs_pass` | CR131 design surface | Fix process design surface drift if local. |
| 39 | `tests/test_cr131_design_surface.py::test_cr131_design_surface_cli_json` | CR131 design surface | Fix CLI JSON/static docs drift if local. |
| 40 | `tests/test_experiment_17_21_factor_suite.py::test_experiment_17_21_generates_factor_stability_and_strategy_outputs` | Remaining experiment suites 17_21 / 23_29 | Defer to `FU-CR140-002` unless independent fixture-only fix. |
| 41 | `tests/test_experiment_23_29_ml_factor_suite.py::test_stage4_suite_generates_expected_reports_with_bounded_concurrency` | Remaining experiment suites 17_21 / 23_29 | Defer to `FU-CR140-002` unless independent fixture-only fix. |
| 42 | `tests/test_market_data_contracts.py::test_market_data_does_not_import_engine_family` | market_data imports engine family | Inspect architecture import graph. |
| 43 | `tests/test_market_data_tushare_comparison.py::test_docs_record_tushare_backfill_required_missing_proxy_and_backtrader_boundaries` | market_data tushare comparison docs | Fix static docs drift if local. |
| 44 | `tests/test_script_entrypoint_naming.py::test_script_entrypoint_naming_guardrail_passes_current_repo` | CR139 root script naming debt | Inspect stable-entrypoint registration. |
| 45 | `tests/test_story_004_013.py::test_story_004_loader_reads_offline_parquet_and_quality_gate` | STORY-004/013 loader/logging | Inspect local fixture/code drift. |
| 46 | `tests/test_story_004_013.py::test_t_logging_minimal_01_cli_diagnostics` | STORY-004/013 loader/logging | Inspect local fixture/code drift. |

## Phase 2 Proposed Order

1. Fix isolated guardrail/static failures first: script entrypoint naming, CR131 design surface, market_data import boundary if a small dependency inversion is sufficient.
2. Fix experiment-adjacent CR008/CR011 static/report contract failures where they do not require true engine convergence.
3. Defer the two remaining experiment suite failures to `FU-CR140-002` unless inspection proves they are trivial fixture drift.
4. Leave any real lake/runtime/trading/remote-write need blocked and convert it to a follow-up authorization gate.
