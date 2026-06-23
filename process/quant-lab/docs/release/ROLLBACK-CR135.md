---
title: "CR135 Rollback"
source_cr: "CR-135"
status: "ready_with_risk"
profile: "compact"
created_at: "2026-06-23"
---

# CR135 Rollback

## Rollback Target

Return Strategy Runner Core to the pre-CR135 state where CR134 evidence index output exists but artifact bundle output / inspect is absent.

## Files To Revert If Needed

- `trading/strategy_runner/artifact_bundle.py`
- `tests/test_cr135_runner_artifact_bundle.py`
- CR135 changes in `trading/strategy_runner/result.py`
- CR135 changes in `trading/strategy_runner/run_spec.py`
- CR135 changes in `trading/strategy_runner/runner.py`
- CR135 changes in `trading/strategy_runner/cli.py`
- CR135 changes in `trading/strategy_runner/__init__.py`
- CR135 process evidence and feature docs v1.3

## Verification After Rollback

Run CR128 / CR133 / CR134 regression and hygiene checks. CR135 bundle tests should be removed or expected to fail after rollback.

## Irreversible Actions

None. CR135 performs no external publish, runtime action, NAS action, data migration or trading operation.
