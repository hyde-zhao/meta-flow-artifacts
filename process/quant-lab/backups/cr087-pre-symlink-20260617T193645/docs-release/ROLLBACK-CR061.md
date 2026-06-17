# ROLLBACK-CR061

## Rollback Scope

Rollback is limited to CR061 Batch B repo-local diff.

| Item | Rollback Action |
|---|---|
| `pyproject.toml` | Restore `[project].name = "local-backtest"`. |
| `uv.lock` | Restore the previous lockfile virtual package block for `local-backtest`. |
| `quant_lab/**` | Remove Batch B added compatibility namespace files. |
| `tests/test_cr061_package_import_layout.py` | Remove Batch B smoke test. |
| CR061 evidence docs | Preserve by default for audit; add superseding rollback evidence if rollback occurs. |

## Rollback Verification

| Command / Check | Expected |
|---|---|
| `uv lock` | Lock resolves with restored project metadata. |
| `uv run --python 3.11 pytest -q tests/test_cr061_package_import_layout.py` | N/A after removing test, or adjust expectation before running. |
| Legacy import smoke | `engine`, `market_data`, `strategies`, `trading` still import. |
| `git diff --check` | PASS. |

## Not A Rollback Mechanism

- Git history rewrite.
- Force push.
- NAS / data lake deletion.
- Credential read.
- Runtime state mutation.

## Rollback Decision

Rollback is not recommended because CP6 and CP7 passed within the approved scope. Use rollback only if downstream tooling cannot tolerate `quant-lab` project metadata or the compatibility namespace.
