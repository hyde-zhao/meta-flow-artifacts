# CR142 Phase 0 Engine Contract Audit

## Scope

CR142 starts from `FU-CR140-002` to converge the remaining experiment evaluation stack. This Phase 0 audit is source/static only and does not execute real lake, NAS, provider, catalog, runtime, simulation, live, trading, credential, or Git remote write operations.

## Findings

| Finding | Evidence | Decision |
|---|---|---|
| Engine statistics already provides reusable forward return, single sort, double sort, long-short, Newey-West, and Fama-MacBeth utilities. | `engine/factor_statistics.py` | Reuse via experiment adapters where output contract allows. |
| Factor library supports `additional_definitions`. | `engine/factor_library.py::build_equity_factor_library` | Use for runner-local factor definitions instead of core promotion. |
| Factor calculators supports `calculator_registry`, but direction mapping reads only core definitions. | `engine/factor_calculators.py::compute_equity_factor_matrices` | Phase 1 fix: add explicit additional definitions / direction contract. |
| Factor registry supports `extra_entries`, and stage3 catalog entries already exist for several runner-local factors. | `engine/factor_registry.py` | Keep runner-local registry explicit; do not claim production core factor status. |
| Experiment 17_21 still owns reporting, strategy construction, liquidity/cost checks, and fixture orchestration. | `experiments/run_experiment_17_21_factor_suite.py` | Only migrate reusable evaluation statistics, not whole experiment script. |
| Experiment 23_29 consumes 17_21 helpers and adds ML suite orchestration. | `experiments/run_experiment_23_29_ml_factor_suite.py` | Reuse the same adapter; preserve optional dependency fallback. |

## Phase 1 Test Slice

Add/extend `tests/test_factor_calculators.py` so a custom negative factor supplied through additional definitions and calculator registry is direction-adjusted correctly. This proves the engine extension contract before touching experiments.

## Authorization Boundary

No real lake readonly validation, real lake link/mount, NAS, provider fetch, lake write, catalog pointer write, QMT/MiniQMT/xtquant/gateway runtime, simulation/live/trading, credential read, or Git remote write is authorized.

## Decision

Proceed to Phase 1 with a small engine extension contract fix. If the implementation needs real data semantics or runtime verification, stop and request a separate human gate.
