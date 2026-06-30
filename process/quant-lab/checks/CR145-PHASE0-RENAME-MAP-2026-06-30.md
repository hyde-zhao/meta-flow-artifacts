# CR145 Phase 0 Rename and Merge Map

## Entry Criteria

| Item | Status | Evidence |
|---|---|---|
| Clean source worktree before start | PASS | `git status --short` empty |
| Clean process worktree before start | PASS | `git -C process status --short` empty |
| Process route health | PASS | `uv run --python 3.11 meta-flow workspace check` |

## Checklist

| Item | Status | Evidence |
|---|---|---|
| Experiment rename map generated | PASS | `process/context/CR145-EXPERIMENT-TEST-RENAME-MAP.csv` |
| Test merge plan generated | PASS | `process/context/CR145-TEST-MERGE-PLAN.csv` |
| Provenance registry generated | PASS | `tests/PROVENANCE.yaml` |

## Exit Criteria

Phase 0 map exists and CR145 implementation can proceed.
