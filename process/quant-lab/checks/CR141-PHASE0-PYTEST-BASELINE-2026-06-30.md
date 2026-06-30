# CR141 Phase 0 Pytest Baseline

## Command

```bash
uv run --python 3.11 pytest -q --tb=no -p no:cacheprovider
```

## Result

| Metric | Value |
|---|---:|
| Exit code | 1 |
| Failed | 46 |
| Passed | 1433 |
| Duration | 45.46s |

Failure set:

- `process/checks/CR141-PHASE0-BASELINE-FAILURES.txt`

## Comparison

This matches the CR140 post-close risk supplement count: `46 failed, 1433 passed`. The failure set is the expected inherited red baseline debt for CR141 triage.

## Authorization Boundary

This run executed local pytest only. No real lake readonly validation, real lake link/mount, NAS, provider fetch, lake write, catalog pointer write, QMT/MiniQMT/xtquant/gateway runtime, simulation/live/trading, or Git remote write was executed.

## Decision

Proceed to Phase 1 classification. No manual gate is required because this baseline confirms the known red state and does not introduce new runtime, data, trading, or remote-write risk.
