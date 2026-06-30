# CR140 Phase 3 Baseline Comparison

## 结论

PASS。Phase 3 source 与 artifact 均提交后，从 clean source/artifact 状态运行全量 pytest，失败集合未扩大。

## 执行前状态

- source repo：`git status --short` 为空。
- artifact repo：`git -C /home/hyde/workspace/meta-flow-artifacts status --short` 为空。
- Phase 3 source commit：`0e65d90 refactor(experiments): route turnover evaluation through engine`
- Phase 3 artifact commit：`fc90832 record CR140 phase3 synthetic turnover parity`

## 验证命令

```bash
uv run --python 3.11 pytest -q --tb=no -p no:cacheprovider
```

结果：

```text
46 failed, 1431 passed in 43.53s
```

## 与 Phase 0 对比

Phase 0 基线：

```text
46 failed, 1428 passed in 49.07s
```

对比结论：

- 新增失败：0
- 修复的基线失败：0
- 保持失败：46
- 新增通过：3，来自 `tests/test_cr140_turnover_engine_adapter.py` 的 CR140 synthetic parity 测试。

当前失败清单与 `process/checks/CR140-PHASE0-BASELINE-FAILURES.txt` 逐项一致：

- `tests/test_cr008_factor_auxiliary_data_contract.py::test_t08_reader_helper_returns_typed_missing_and_never_auto_executes`
- `tests/test_cr008_factor_auxiliary_data_contract.py::test_t11_experiment_15_schema_report_and_summary_include_auxiliary_contract`
- `tests/test_cr008_quality_adjustment_label_gates.py::test_adjustment_policy_mixed_missing_or_mismatch_fails[missing-adjustment_policy_missing]`
- `tests/test_cr008_quality_adjustment_label_gates.py::test_adjustment_policy_mixed_missing_or_mismatch_fails[mismatch-adjustment_policy_mismatch]`
- `tests/test_cr008_research_input_metadata.py::test_experiment_cli_data_dir_must_be_explicit`
- `tests/test_cr008_research_input_metadata.py::test_no_connector_runtime_storage_imports_or_credentials_in_target_files`
- `tests/test_cr011_adjustment_audit.py::test_s05_forbidden_boundaries_are_static_and_no_secret_leakage`
- `tests/test_cr011_benchmark_policy_consumption.py::test_experiment_production_strict_missing_benchmark_has_blocked_claims_without_proxy`
- `tests/test_cr011_benchmark_policy_consumption.py::test_old_experiment_report_path_is_rejected_before_writes`
- `tests/test_cr011_capacity_cost_sensitivity.py::test_experiment_metadata_uses_s07_contract_without_old_report_overwrite`
- `tests/test_cr011_exposure_claims.py::test_s06_forbidden_boundaries_are_static_and_no_secret_leakage`
- `tests/test_cr011_factor_panel_robust_validation.py::test_s08_forbidden_boundaries_are_static_and_legacy_report_fails_fast`
- `tests/test_cr013_backfill_roadmap_boundaries.py::test_backfill_roadmap_lists_full_history_and_vwap_release_criteria`
- `tests/test_cr013_backfill_roadmap_boundaries.py::test_backfill_roadmap_current_real_operations_are_not_authorized`
- `tests/test_cr013_backfill_roadmap_boundaries.py::test_backfill_roadmap_contains_no_direct_execution_commands_or_secret_prompts`
- `tests/test_cr013_backfill_roadmap_boundaries.py::test_backfill_roadmap_forbidden_operation_counters_are_zero`
- `tests/test_cr013_execution_vwap_claim_boundary.py::test_execution_audit_resolves_real_vwap_and_fill_as_blocked`
- `tests/test_cr013_execution_vwap_claim_boundary.py::test_amount_volume_derived_vwap_attempt_is_not_allowed`
- `tests/test_cr013_execution_vwap_claim_boundary.py::test_execution_claim_metadata_and_report_preserve_boundaries`
- `tests/test_cr013_full_history_gap_register.py::test_full_history_gap_register_has_exact_ten_formal_datasets`
- `tests/test_cr013_full_history_gap_register.py::test_full_history_summary_blocks_2020_2024_claim_and_preserves_old_evidence`
- `tests/test_cr013_full_history_gap_register.py::test_full_history_forbidden_operation_counters_are_zero`
- `tests/test_cr013_unsupported_register_claim_boundary.py::test_unsupported_register_exact_rows_and_excluded_denominator`
- `tests/test_cr013_unsupported_register_claim_boundary.py::test_claim_boundary_summary_merges_s01_s02_and_excludes_denominator`
- `tests/test_cr013_unsupported_register_claim_boundary.py::test_docs_and_report_show_supported_research_only_unsupported_blocked_boundaries`
- `tests/test_cr015_foundation_runbook_boundary.py::test_sensitive_value_output_count_is_zero`
- `tests/test_cr016_docs_incident_playbooks.py::test_sensitive_raw_value_output_count_is_zero`
- `tests/test_cr016_runbook_approval_gates.py::test_sensitive_raw_value_output_count_is_zero`
- `tests/test_cr019_deferred_capabilities.py::test_readme_declares_later_gated_non_p0_boundary_without_current_enablement`
- `tests/test_cr019_deferred_capabilities.py::test_dependency_and_scope_expansion_counters_stay_zero_in_contract`
- `tests/test_cr019_qmt_gateway_fallback.py::test_incident_playbook_cr019_section_freezes_manual_only_boundary`
- `tests/test_cr019_qmt_gateway_lifecycle.py::test_install_doc_uses_placeholders_and_contains_no_sensitive_literals`
- `tests/test_cr020_docs_runbook_no_authorization.py::test_cr020_manual_install_sections_and_runtime_commands_are_present`
- `tests/test_cr020_docs_runbook_no_authorization.py::test_cr020_docs_no_authorization_table_blocks_trading_and_writes`
- `tests/test_cr020_docs_runbook_no_authorization.py::test_cr020_cp7_evidence_schema_excludes_raw_positions_and_secret_output`
- `tests/test_cr020_server_qmt_login_session.py::test_env_example_is_placeholder_only_and_contains_no_real_values`
- `tests/test_cr045_goldminer_no_operation_static.py::test_runbook_does_not_contain_positive_runtime_authorization_claims`
- `tests/test_cr131_design_surface.py::test_cr131_design_surface_current_docs_pass`
- `tests/test_cr131_design_surface.py::test_cr131_design_surface_cli_json`
- `tests/test_experiment_17_21_factor_suite.py::test_experiment_17_21_generates_factor_stability_and_strategy_outputs`
- `tests/test_experiment_23_29_ml_factor_suite.py::test_stage4_suite_generates_expected_reports_with_bounded_concurrency`
- `tests/test_market_data_contracts.py::test_market_data_does_not_import_engine_family`
- `tests/test_market_data_tushare_comparison.py::test_docs_record_tushare_backfill_required_missing_proxy_and_backtrader_boundaries`
- `tests/test_script_entrypoint_naming.py::test_script_entrypoint_naming_guardrail_passes_current_repo`
- `tests/test_story_004_013.py::test_story_004_loader_reads_offline_parquet_and_quality_gate`
- `tests/test_story_004_013.py::test_t_logging_minimal_01_cli_diagnostics`

## 决策

Phase 3 可以关闭，进入 Phase 5 cleanup。

仍需保持 CR140 声明边界：

- Phase 3 只证明 synthetic fixture 上的新旧重构等价。
- 不声明真实数据湖研究语义已验证。
- 不声明 simulation/live/QMT 准入。
- 未执行真实 lake、NAS、provider、catalog pointer、QMT/live/runtime 操作。
