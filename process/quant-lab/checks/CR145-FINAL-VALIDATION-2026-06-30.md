# CR145 Final Validation

## Entry Criteria

| Item | Status | Evidence |
|---|---|---|
| Rename implementation complete | PASS | 13 experiment entrypoints renamed; no top-level `tests/test_*.py` remains |
| Test merge implementation complete | PASS | 14 fragmented CR139 data lake files merged into 4 capability files |
| Indexes updated | PASS | `tests/PROVENANCE.yaml`, rename map, and merge plan updated |

## Checklist

| Check | Status | Evidence |
|---|---|---|
| Pytest collection | PASS | `uv run --python 3.11 pytest --collect-only -q`: 1490 tests collected |
| Naming guardrails and merged data lake tests | PASS | `uv run --python 3.11 pytest -q tests/meta_flow/test_test_file_taxonomy.py tests/meta_flow/test_experiment_entrypoint_taxonomy.py tests/data_lake/test_dedup_contracts.py tests/data_lake/test_lineage_and_run_id.py tests/data_lake/test_catalog_manifest_pointer.py tests/data_lake/test_schema_contracts.py`: 35 passed |
| Python compile check | PASS | `uv run --python 3.11 python -m py_compile experiments/*.py tests/*/test_*.py` |
| Old path/import scan | PASS | No matches for old `experiments/run_experiment_*`, `from experiments.run_experiment_*`, top-level `tests/test_*`, `tests/test_crNNN_*`, `tests/test_experiment_*`, `tests/test_stage*`, or `tests/test_story_*` outside legacy/provenance records |
| Pre-commit pytest excluding dirty-sensitive process hygiene | PASS | `uv run --python 3.11 pytest -q --ignore=tests/meta_flow/test_process_artifact_hygiene.py`: 1473 passed |
| Post-commit process hygiene and naming guardrails | PASS | `uv run --python 3.11 pytest -q tests/meta_flow/test_process_artifact_hygiene.py tests/meta_flow/test_test_file_taxonomy.py tests/meta_flow/test_experiment_entrypoint_taxonomy.py`: 20 passed |
| Post-commit workspace route health | PASS | `uv run --python 3.11 meta-flow workspace check`: `process_link_health: ok`, `artifact_git_dirty: clean` |
| Post-commit full pytest | PASS | `uv run --python 3.11 pytest -q`: 1490 passed |
| Self-check against two-layer test asset policy | PASS | P0 CR139 data lake merges were already complete; P1 runner control plane, QMT gateway contracts, and human gate contract merges were added after self-check |
| Expanded merge plan schema | PASS | `process/context/CR145-TEST-MERGE-PLAN.csv` now includes `merge_group`, `risk`, `requires_fixture_move`, `provenance_refs`, and explicit `move_only_do_not_merge` decisions |
| Post-P1 source hygiene and naming guardrails | PASS | `uv run --python 3.11 pytest -q tests/meta_flow/test_process_artifact_hygiene.py tests/meta_flow/test_test_file_taxonomy.py tests/meta_flow/test_experiment_entrypoint_taxonomy.py`: 20 passed |
| Post-P1 full pytest | PASS | `uv run --python 3.11 pytest -q`: 1490 passed |

## Exit Criteria

CR145 source/process changes are committed and validated. Dirty-worktree-sensitive process hygiene and full pytest passed after commits.
