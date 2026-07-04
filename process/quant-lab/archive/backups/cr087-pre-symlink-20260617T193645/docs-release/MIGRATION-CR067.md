# CR067 Migration Notes

CR067 performed a publication migration from local workspace contents to a clean GitHub baseline.

## Migrated

- Allowlisted source code and package/import compatibility files.
- User-facing README and USER-MANUAL.
- `.env.example` placeholder-only file.
- CR061 package/import smoke test.

## Not Migrated

- `data/**`
- `reports/**`
- `runs/**`
- `.env`, `.env.local`, `.envrc`
- `.venv/**`
- process state and full process history
- NAS / data lake / provider / catalog / runtime assets
- QMT / MiniQMT runtime state

## Compatibility

The public baseline preserves the current `quant_lab` compatibility namespace and legacy roots (`engine`, `market_data`, `strategies`, `trading`). No bulk import rewrite was performed.
