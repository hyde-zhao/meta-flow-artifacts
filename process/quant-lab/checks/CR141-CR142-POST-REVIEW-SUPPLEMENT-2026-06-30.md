# CR141 / CR142 Post-Review Supplement

## Scope

This supplement records two post-review clarifications requested after independent verification of the experiment remediation work. It does not reopen CR141 or CR142 and does not change runtime behavior.

## Changes

| Area | Change | Evidence |
|---|---|---|
| CR141 scope declaration | Added an explicit scope revision noting that Phase 2 Wave 1 included `market_data` / `engine` data contract ownership migration, and that CR141 used 5 waves / 6 source commits. | `process/changes/CR-141-RED-BASELINE-DEBT-TRIAGE-2026-06-30.md` |
| CR142 API semantics | Added docstring text to `compute_equity_factor_matrices` clarifying that `additional_definitions` participates in direction parsing and can affect `directional_matrices`. | source commit `340a71b` |

## Verification

```bash
uv run --python 3.11 python -m py_compile engine/factor_calculators.py
```

Result: PASS.

```bash
uv run --python 3.11 pytest -q tests/test_factor_calculators.py
```

Result: `4 passed in 0.53s`.

## Authorization Boundary

No real lake readonly validation, real lake link/mount, NAS, provider fetch, lake write, catalog pointer write, QMT/MiniQMT/xtquant/gateway runtime, simulation/live/trading, credential read, or Git remote write was executed.

## Decision

The review findings are addressed as documentation and audit-chain clarifications. CR141 and CR142 remain `closed-current-delivery / ready`.
