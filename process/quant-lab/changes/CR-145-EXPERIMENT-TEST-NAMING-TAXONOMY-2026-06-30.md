---
id: "CR-145"
title: "Experiment and Test Naming Taxonomy Convergence"
status: "validated"
kind: "refactor"
lifecycle_status: "validated"
readiness_status: "complete"
gate_profile: "standard"
created_at: "2026-06-30T23:50:00+08:00"
created_by: "host-orchestrator"
product_baseline_refresh_required: false
impact_level: "high-medium"
rollback_to: "pre-CR145 clean worktree"
not_authorized:
  - real_lake_write
  - provider_fetch
  - catalog_pointer_publish
  - qmt_live_runtime
  - simulation_or_trading
---

# CR-145 Experiment and Test Naming Taxonomy Convergence

## Objective

Converge `experiments/` run entrypoint names and `tests/` test asset taxonomy around durable domain/capability names, demoting CR/story/chapter/experiment numbers to machine-tracked provenance.

## Scope

- Rename 13 experiment entrypoints to domain-first names.
- Move 228 top-level test files into domain directories.
- Merge selected fragmented CR139 data lake tests into capability files.
- Update imports, hardcoded path allowlists, docs, and indexes.
- Add `tests/PROVENANCE.yaml` as the machine-readable old-path/provenance index.

## Alternatives Considered

| Alternative | Decision | Reason |
|---|---|---|
| Flat remove `crNNN_` prefix plus markers | Rejected | Leaves 228 tests flat and keeps meta-flow/product tests mixed. |
| `tests/cr139/` style CR directories | Rejected | Moves the temporary CR axis from filename to directory. |
| Domain directories plus provenance registry | Accepted | Improves findability while retaining audit traceability. |

## Risks

- `run_*` files still act as both CLI entrypoints and importable libraries; a later `experiments/_lib/` extraction should be considered if future renames again touch many imports.
- Hardcoded test paths in hygiene and guardrail tests must be kept synchronized.

## Evidence

- Rename map: `process/context/CR145-EXPERIMENT-TEST-RENAME-MAP.csv`
- Merge plan: `process/context/CR145-TEST-MERGE-PLAN.csv`
- Provenance registry: `tests/PROVENANCE.yaml`
- Final validation: `process/checks/CR145-FINAL-VALIDATION-2026-06-30.md`

## Implementation Summary

- `experiments/`: renamed all 13 `run_experiment_*` entrypoints to domain-first `run_*_expNN` names, with `run_low_turnover_double_sort.py` kept unnumbered because it is outside the original experiment sequence.
- `tests/`: moved every top-level `tests/test_*.py` file into domain directories: `backtest`, `data_lake`, `docs_quality`, `experiments`, `market_data`, `meta_flow`, `research`, `runner`, `safety`, `scripts`, and `trading`.
- Fragmented CR139 data lake tests were merged into four durable capability files: `test_dedup_contracts.py`, `test_lineage_and_run_id.py`, `test_catalog_manifest_pointer.py`, and `test_schema_contracts.py`.
- P1 small-fragment candidates were merged after self-check into three durable capability files: `tests/runner/test_control_plane_contracts.py`, `tests/trading/test_qmt_gateway_contracts.py`, and `tests/meta_flow/test_human_gate_contracts.py`.
- Cross-file imports, static guard allowlists, docs, research metadata, hygiene scripts, and hardcoded test paths were updated to the new paths.
- Added taxonomy guardrails for `experiments/` entrypoints and `tests/` domain/provenance coverage.
