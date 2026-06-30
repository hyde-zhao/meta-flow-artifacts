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

## Exit Criteria

CR145 source/process changes are ready for source and process commits. Dirty-worktree-sensitive process hygiene and full pytest must be rerun after commits.
