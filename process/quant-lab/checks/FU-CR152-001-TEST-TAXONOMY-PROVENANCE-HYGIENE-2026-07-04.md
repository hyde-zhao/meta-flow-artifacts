---
check_id: "FU-CR152-001-TEST-TAXONOMY-PROVENANCE-HYGIENE"
type: "fast-lane-implementation-verification"
status: "PASS_WITH_RESIDUAL_UNRELATED_FAILURES"
owner: "host-orchestrator"
created_at: "2026-07-04T09:04:41+08:00"
source_follow_up: "process/changes/CR-152-ML-STRATEGY-E2E-FRAMEWORK-FOLLOW-UP-TRACKING-2026-07-02.md#FU-CR152-001"
authorization_source: "user-2026-07-04"
---

# FU-CR152-001 Test Taxonomy / Provenance Hygiene

## Scope

Compact fast-lane治理，目标是清理 CR152 CP7 暴露的测试 taxonomy / provenance hygiene 缺口。

## Implementation Summary

| 项 | 结果 |
|---|---|
| 根目录游离测试文件 | `tests/test_cr150_multifactor_framework_completion.py` moved to `tests/research/test_multifactor_framework_completion.py` |
| 根目录游离测试文件 | `tests/test_cr151_strategy_admission_statistical_gate.py` moved to `tests/research/test_strategy_admission_statistical_gate.py` |
| Forbidden `test_crNNN_` pattern | Removed from the two active test filenames |
| `tests/PROVENANCE.yaml` | Added 11 missing entries |
| Runtime / real data / trading / Git remote write | Not authorized and not executed |

## Provenance Entries Added

| Test file | Domain | Provenance |
|---|---|---|
| `tests/research/test_multifactor_framework_completion.py` | research | CR-150; old path `tests/test_cr150_multifactor_framework_completion.py` |
| `tests/research/test_strategy_admission_statistical_gate.py` | research | CR-151; old path `tests/test_cr151_strategy_admission_statistical_gate.py` |
| `tests/data_lake/test_current_truth_profile.py` | data_lake | CR-149 |
| `tests/data_lake/test_governed_lake_readiness_matrix.py` | data_lake | CR-149 |
| `tests/data_lake/test_physical_partition_migration_execute.py` | data_lake | CR-146 |
| `tests/data_lake/test_physical_partition_migration_plan.py` | data_lake | CR-146 |
| `tests/research/test_backtest_production_contracts.py` | research | CR-148 |
| `tests/research/test_cross_strategy_reliability_gates.py` | research | CR-154 |
| `tests/research/test_event_driven_strategy_e2e_contracts.py` | research | CR-153 |
| `tests/research/test_ml_strategy_e2e_contracts.py` | research | CR-152 |
| `tests/research/test_research_production_contracts.py` | research | CR-147 |

## Verification

| Check | Result |
|---|---|
| `uv run pytest tests/meta_flow/test_test_file_taxonomy.py -q` | PASS: `2 passed in 0.06s` |
| Root-level `tests/test_*.py` scan | PASS: no root-level test files remain |
| Provenance count equality | PASS: `actual_count=225`, `provenance_count=225`, `missing=[]`, `stale=[]` |
| Targeted affected test set | PASS: `111 passed in 1.20s` |
| `git diff --check` for changed test files | PASS |
| Full `uv run pytest -q` | `1603 passed, 4 failed in 43.66s`; failures are `tests/meta_flow/test_design_surface.py::{test_cr131_design_surface_current_docs_pass,test_cr131_design_surface_cli_json}` and `tests/meta_flow/test_process_artifact_hygiene.py::{test_cr132_process_artifact_hygiene_current_workspace_passes,test_cr132_process_artifact_hygiene_cli_json}` |

## Residual Risk

The full-suite failures are process/design-surface hygiene checks outside this FU-CR152 source/test taxonomy scope:

- `check_design_surface.py` reports pre-existing unexpected CR-named design root files under `process/docs/design`.
- `check_process_artifact_hygiene.py` reports dirty source/process artifact statuses, including this in-progress source diff and older process artifact drift.

These failures do not invalidate the taxonomy/provenance objective, which is proven by the targeted guardrail and provenance equality checks. Final packaging/commit remains separate from this verification and does not authorize Git remote write.
