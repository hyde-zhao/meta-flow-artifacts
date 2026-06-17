# RELEASE-NOTES-CR061

## Release Decision

`READY_WITH_RISK`

CR061 Batch B completes the first package/import identity convergence slice for quant-lab. It is ready for user delivery acceptance, with known risks listed below.

## User-Visible Changes

| Area | Change |
|---|---|
| Project metadata | `pyproject.toml` project name is now `quant-lab`. |
| Lock metadata | `uv.lock` now records the virtual project package as `quant-lab v0.1.0`. |
| Python imports | New `quant_lab` compatibility namespace is available. |
| Legacy compatibility | Existing `engine`, `market_data`, `strategies`, and `trading` import roots remain available. |

## Behavior Changes

- `import quant_lab` succeeds.
- `import quant_lab.engine`, `import quant_lab.market_data`, `import quant_lab.strategies`, and `import quant_lab.trading` alias the existing legacy roots.
- Existing user commands such as `python -m market_data.cli ...` are not changed by this CR.

## Breaking Changes

No intended breaking change.

## Known Risks

| Risk ID | Severity | Description | Handling |
|---|---|---|---|
| R-CR061B-V01 | MEDIUM | Full pytest was not run. | CP8 risk acceptance required. |
| R-CR061B-V02 | LOW | `trading` is a namespace package and only package-level alias was verified. | Revisit before bulk relayout. |
| R-CR061B-V03 | LOW | `quant_lab.market_data` alias inherits legacy package initialization. | Smoke passed; monitor future failures. |

## Not Authorized By This Release

- Git remote add / push / tag / branch rename / history rewrite.
- NAS copy / move / delete / archive promote.
- Data lake root replacement, provider fetch, lake write, catalog publish.
- Credential read.
- QMT / MiniQMT runtime, account query, order submit/cancel, simulation/live.
- CR046 recovery.

## Evidence

- Implementation: `docs/features/cr061-package-import-layout/IMPLEMENTATION.md`
- Verification: `docs/features/cr061-package-import-layout/VERIFICATION.md`
- CP6: `process/checks/CP6-CR061-BATCH-B-PACKAGE-IMPORT-LAYOUT-CODING-DONE.md`
- CP7: `process/checks/CP7-CR061-BATCH-B-PACKAGE-IMPORT-LAYOUT-VERIFICATION-DONE.md`
