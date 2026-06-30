---
change_id: "CR-139"
batch_id: "CR139-BACKLOG-A10"
stories: ["CR139-S36", "CR139-S37"]
stage: "CP6"
status: "complete"
implemented_at: "2026-06-30T17:00:00+08:00"
implemented_by: "host-orchestrator-inline"
validation_mode: "static-fixture"
---

# CR139 Backlog-A10 S36/S37 Implementation

## Scope

| Story | Implemented | Non-scope |
|---|---|---|
| S36 | CR139 fixture tests proving S13 DuckDB readonly façade preserves projections, partition filters, rendered SQL column pruning and equality predicate pushdown. | DuckDB package install, performance benchmark, real parquet scan, production reader refactor, catalog/manifest/pointer writes. |
| S37 | CR139 fixture e2e chain proving readonly request -> fallback result -> audit evidence -> side-effect check, with no `.duckdb` persistent file. | Real DuckDB runtime, real lake scan, provider fetch, NAS sync, runtime/trading operation, Git remote write. |

## Implementation Objects

| File | Change |
|---|---|
| `tests/test_cr139_duckdb_pushdown_e2e.py` | Added S36/S37 fixture/static contract tests. |
| `market_data/readers.py` | Reused existing S13 `read_dataset_via_duckdb_contract()` façade; no code change in this batch. |
| `market_data/duckdb_query.py` | Reused existing CR014/S13 read-only query contracts; no code change. |
| `market_data/audit.py` | Reused existing audit evidence and side-effect contracts; no code change. |
| `pyproject.toml` / `uv.lock` | Unchanged; no DuckDB dependency added. |

## Contract Mapping

| Contract | Implementation | Verification |
|---|---|---|
| Requested projections must be preserved through S13 façade. | `test_s36_projection_and_predicate_pushdown_are_preserved` asserts `request.projections` and SQL select list. | `tests/test_cr139_duckdb_pushdown_e2e.py` |
| Requested partition filters must be preserved and rendered deterministically. | Same test asserts `request.partition_filters` and `WHERE trade_date = '20260526'`. | `tests/test_cr139_duckdb_pushdown_e2e.py` |
| Forbidden write SQL must be blocked before execution. | `test_s36_forbidden_sql_template_is_blocked_before_execution` uses a `CREATE TABLE` template and expects `FORBIDDEN_SQL`. | `tests/test_cr139_duckdb_pushdown_e2e.py` |
| DuckDB adapter e2e must remain read-only and evidence-only. | `test_s37_readonly_e2e_audit_has_no_source_of_truth_side_effects` builds audit evidence and side-effect summary from a fallback readonly result. | `tests/test_cr139_duckdb_pushdown_e2e.py` |
| No persistent `.duckdb` file or dependency change. | All tests use `tmp_path`, no adapter, no dependency approval, and `.duckdb` sentinel checks. | `tests/test_cr139_duckdb_pushdown_e2e.py`; S13 regression tests |

## Verification

| Command | Result |
|---|---|
| `uv run --python 3.11 pytest -q tests/test_cr139_duckdb_pushdown_e2e.py tests/test_cr139_duckdb_readonly_adapter.py tests/test_cr014_duckdb_readonly_boundary.py` | PASS, 14 passed in 0.49s |

## Boundary Check

No DuckDB dependency install, provider fetch, credential read, real lake read/list/write, active catalog/manifest write, published pointer advance, `.duckdb` persistent file creation, physical migration, NAS operation, runtime/trading operation, or Git remote write was performed. Test writes are isolated under pytest `tmp_path`.
