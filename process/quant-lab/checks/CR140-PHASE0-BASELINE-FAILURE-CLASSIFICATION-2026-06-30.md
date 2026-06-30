# CR140 Phase 0 Baseline Failure Classification

## Entry Criteria

- Phase 0 full pytest baseline 已记录：46 failed, 1428 passed。
- Failure set 已固化在 `process/checks/CR140-PHASE0-BASELINE-FAILURES.txt`。
- 进入 Phase 1 前需要区分基线失败的来源与 CR140 爆炸半径。

## Classification Legend

| 分类 | 含义 | 后续处理 |
|---|---|---|
| `CR139-debt` | 已知 CR139 / gate script / process routing 相关债务，不属于 CR140 目标。 | 不作为 CR140 修复目标；若修复，记录为基线失败减少。 |
| `redesign-debt` | redesign / 历史文档 / 旧 CR 守护测试债务。 | 不主动修复；仅防止新增失败。 |
| `CR140-hit` | 与 CR140 Phase 1/2/3 迁移、reporting、实验、engine 边界或 entrypoint 直接相关。 | Phase 后必须重点复核；若失败变化，说明是 CR140 影响。 |
| `unrelated` | 与本轮 experiments 重构无直接关系。 | 只作为全量回归背景噪声。 |

## Failure Matrix

| Node id | 分类 | 理由 | Phase 后判断 |
|---|---|---|---|
| `tests/test_cr008_factor_auxiliary_data_contract.py::test_t08_reader_helper_returns_typed_missing_and_never_auto_executes` | `CR140-hit` | 涉及 experiment/lake reader helper 和研究输入边界。 | 失败减少可接受；新增同类失败阻断。 |
| `tests/test_cr008_factor_auxiliary_data_contract.py::test_t11_experiment_15_schema_report_and_summary_include_auxiliary_contract` | `CR140-hit` | 涉及 experiment 15 report/schema，Phase 2 helper 收敛可能触达。 | 重点复核输出字段。 |
| `tests/test_cr008_quality_adjustment_label_gates.py::test_adjustment_policy_mixed_missing_or_mismatch_fails[missing-adjustment_policy_missing]` | `CR140-hit` | 研究输入质量 gate 与实验输出边界相关。 | 不得新增质量 gate 类失败。 |
| `tests/test_cr008_quality_adjustment_label_gates.py::test_adjustment_policy_mixed_missing_or_mismatch_fails[mismatch-adjustment_policy_mismatch]` | `CR140-hit` | 同上。 | 不得新增质量 gate 类失败。 |
| `tests/test_cr008_research_input_metadata.py::test_experiment_cli_data_dir_must_be_explicit` | `CR140-hit` | Phase 1/3 涉及 experiment 输入路径、fixture 和 lake 边界。 | 重点复核 CLI/metadata。 |
| `tests/test_cr008_research_input_metadata.py::test_no_connector_runtime_storage_imports_or_credentials_in_target_files` | `CR140-hit` | Phase 1 移动文件会改变 target file 扫描面。 | 新增 forbidden import 阻断。 |
| `tests/test_cr011_adjustment_audit.py::test_s05_forbidden_boundaries_are_static_and_no_secret_leakage` | `CR140-hit` | reporting / old report boundary 可能受 Phase 1 影响。 | 需对照 import/path scan。 |
| `tests/test_cr011_benchmark_policy_consumption.py::test_experiment_production_strict_missing_benchmark_has_blocked_claims_without_proxy` | `CR140-hit` | experiment production_strict / reporting 边界。 | 输出 claim 变化需解释。 |
| `tests/test_cr011_benchmark_policy_consumption.py::test_old_experiment_report_path_is_rejected_before_writes` | `CR140-hit` | old report guard 是 CR140 不变量。 | 不得放松 guard。 |
| `tests/test_cr011_capacity_cost_sensitivity.py::test_experiment_metadata_uses_s07_contract_without_old_report_overwrite` | `CR140-hit` | experiment metadata/report schema 可能受 helper/reporting 迁移影响。 | 重点复核 metadata。 |
| `tests/test_cr011_exposure_claims.py::test_s06_forbidden_boundaries_are_static_and_no_secret_leakage` | `CR140-hit` | reporting/secret leakage 静态边界。 | 不得新增泄漏。 |
| `tests/test_cr011_factor_panel_robust_validation.py::test_s08_forbidden_boundaries_are_static_and_legacy_report_fails_fast` | `CR140-hit` | old report guard 永久保留。 | 不得变弱。 |
| `tests/test_cr013_backfill_roadmap_boundaries.py::test_backfill_roadmap_lists_full_history_and_vwap_release_criteria` | `redesign-debt` | CR013 文档/roadmap 守护，非 CR140 直接目标。 | 只监控新增。 |
| `tests/test_cr013_backfill_roadmap_boundaries.py::test_backfill_roadmap_current_real_operations_are_not_authorized` | `redesign-debt` | 同上。 | 只监控新增。 |
| `tests/test_cr013_backfill_roadmap_boundaries.py::test_backfill_roadmap_contains_no_direct_execution_commands_or_secret_prompts` | `redesign-debt` | 同上。 | 只监控新增。 |
| `tests/test_cr013_backfill_roadmap_boundaries.py::test_backfill_roadmap_forbidden_operation_counters_are_zero` | `redesign-debt` | 同上。 | 只监控新增。 |
| `tests/test_cr013_execution_vwap_claim_boundary.py::test_execution_audit_resolves_real_vwap_and_fill_as_blocked` | `CR140-hit` | readiness/reporting boundary 可能受 Phase 1.3 迁移影响。 | 重点复核 data_lake audit 文档。 |
| `tests/test_cr013_execution_vwap_claim_boundary.py::test_amount_volume_derived_vwap_attempt_is_not_allowed` | `CR140-hit` | 同上。 | 不得放松 blocked claim。 |
| `tests/test_cr013_execution_vwap_claim_boundary.py::test_execution_claim_metadata_and_report_preserve_boundaries` | `CR140-hit` | 同上。 | 重点复核 report metadata。 |
| `tests/test_cr013_full_history_gap_register.py::test_full_history_gap_register_has_exact_ten_formal_datasets` | `CR140-hit` | readiness audit / dataset register 边界可能受 Phase 1.3 影响。 | 新增 dataset/register 偏差阻断。 |
| `tests/test_cr013_full_history_gap_register.py::test_full_history_summary_blocks_2020_2024_claim_and_preserves_old_evidence` | `CR140-hit` | 同上。 | 不得把 audit 误表述为 readiness 通过。 |
| `tests/test_cr013_full_history_gap_register.py::test_full_history_forbidden_operation_counters_are_zero` | `CR140-hit` | 同上。 | forbidden counter 必须保持 0。 |
| `tests/test_cr013_unsupported_register_claim_boundary.py::test_unsupported_register_exact_rows_and_excluded_denominator` | `CR140-hit` | Phase 1 reporting/readiness audit import path 会触达该扫描面。 | 重点复核 target file/path 字符串。 |
| `tests/test_cr013_unsupported_register_claim_boundary.py::test_claim_boundary_summary_merges_s01_s02_and_excludes_denominator` | `CR140-hit` | 同上。 | 重点复核 summary。 |
| `tests/test_cr013_unsupported_register_claim_boundary.py::test_docs_and_report_show_supported_research_only_unsupported_blocked_boundaries` | `CR140-hit` | 同上。 | 不得改变 unsupported boundary。 |
| `tests/test_cr015_foundation_runbook_boundary.py::test_sensitive_value_output_count_is_zero` | `redesign-debt` | foundation runbook 文档债，非 CR140 直接目标。 | 只监控新增。 |
| `tests/test_cr016_docs_incident_playbooks.py::test_sensitive_raw_value_output_count_is_zero` | `redesign-debt` | incident playbook 文档债。 | 只监控新增。 |
| `tests/test_cr016_runbook_approval_gates.py::test_sensitive_raw_value_output_count_is_zero` | `redesign-debt` | approval gate 文档债。 | 只监控新增。 |
| `tests/test_cr019_deferred_capabilities.py::test_readme_declares_later_gated_non_p0_boundary_without_current_enablement` | `redesign-debt` | README/deferred capability 文档债。 | 只监控新增。 |
| `tests/test_cr019_deferred_capabilities.py::test_dependency_and_scope_expansion_counters_stay_zero_in_contract` | `redesign-debt` | 同上。 | 只监控新增。 |
| `tests/test_cr019_qmt_gateway_fallback.py::test_incident_playbook_cr019_section_freezes_manual_only_boundary` | `redesign-debt` | QMT runbook 文档债；本轮不碰 runtime。 | 只监控新增。 |
| `tests/test_cr019_qmt_gateway_lifecycle.py::test_install_doc_uses_placeholders_and_contains_no_sensitive_literals` | `redesign-debt` | install doc 文档债。 | 只监控新增。 |
| `tests/test_cr020_docs_runbook_no_authorization.py::test_cr020_manual_install_sections_and_runtime_commands_are_present` | `redesign-debt` | CR020 doc/runbook 债。 | 只监控新增。 |
| `tests/test_cr020_docs_runbook_no_authorization.py::test_cr020_docs_no_authorization_table_blocks_trading_and_writes` | `redesign-debt` | 同上。 | 只监控新增。 |
| `tests/test_cr020_docs_runbook_no_authorization.py::test_cr020_cp7_evidence_schema_excludes_raw_positions_and_secret_output` | `redesign-debt` | 同上。 | 只监控新增。 |
| `tests/test_cr020_server_qmt_login_session.py::test_env_example_is_placeholder_only_and_contains_no_real_values` | `redesign-debt` | env example / QMT placeholder 文档债。 | 只监控新增。 |
| `tests/test_cr045_goldminer_no_operation_static.py::test_runbook_does_not_contain_positive_runtime_authorization_claims` | `redesign-debt` | Goldminer runbook 文档债。 | 只监控新增。 |
| `tests/test_cr131_design_surface.py::test_cr131_design_surface_current_docs_pass` | `redesign-debt` | design surface 文档路由债。 | 只监控新增。 |
| `tests/test_cr131_design_surface.py::test_cr131_design_surface_cli_json` | `redesign-debt` | 同上。 | 只监控新增。 |
| `tests/test_experiment_17_21_factor_suite.py::test_experiment_17_21_generates_factor_stability_and_strategy_outputs` | `CR140-hit` | Phase 2 helper/report formatting 和 Phase 4 排除边界相关。 | 不得改变旧 baseline guard。 |
| `tests/test_experiment_23_29_ml_factor_suite.py::test_stage4_suite_generates_expected_reports_with_bounded_concurrency` | `CR140-hit` | experiments helper/reporting 迁移可能影响 ML/stage4 reports。 | 重点复核输出 schema。 |
| `tests/test_market_data_contracts.py::test_market_data_does_not_import_engine_family` | `CR140-hit` | Phase 1.3 数据湖审计迁移需防止 market_data -> engine 反向依赖。 | 新增 engine-family import 阻断。 |
| `tests/test_market_data_tushare_comparison.py::test_docs_record_tushare_backfill_required_missing_proxy_and_backtrader_boundaries` | `redesign-debt` | Tushare comparison 文档债，非 CR140 直接目标。 | 只监控新增。 |
| `tests/test_script_entrypoint_naming.py::test_script_entrypoint_naming_guardrail_passes_current_repo` | `CR139-debt` | 当前失败根因是 CR139 gate scripts 未注册稳定入口；非 CR140 目标。 | 若 Phase 1.3 新增入口，不能扩大失败。 |
| `tests/test_story_004_013.py::test_story_004_loader_reads_offline_parquet_and_quality_gate` | `redesign-debt` | 旧 Story loader/fixture 债。 | 只监控新增。 |
| `tests/test_story_004_013.py::test_t_logging_minimal_01_cli_diagnostics` | `redesign-debt` | 旧 logging CLI 债。 | 只监控新增。 |

## Summary

| 分类 | 数量 |
|---|---:|
| `CR140-hit` | 25 |
| `redesign-debt` | 20 |
| `CR139-debt` | 1 |
| `unrelated` | 0 |
| Total | 46 |

## Phase Comparison Rule

每个 Phase 后必须输出：

- 新增失败：`current_failures - phase0_failures`
- 修复的基线失败：`phase0_failures - current_failures`
- 保持失败：`current_failures ∩ phase0_failures`

阻断条件：

- `新增失败` 非空。
- `CR140-hit` 分类中出现新的同族失败或输出 schema 变化且无设计说明。
- `CR139-debt` 或 `redesign-debt` 被顺手修复可以接受，但不得作为 CR140 主要交付目标。
