# MIGRATION-CR061

## Migration Summary

CR061 Batch B performs a repo-local package identity migration slice:

| Surface | Before | After |
|---|---|---|
| Distribution metadata | `local-backtest` | `quant-lab` |
| Lock virtual package | `local-backtest v0.1.0` | `quant-lab v0.1.0` |
| New namespace | N/A | `quant_lab` |
| Legacy roots | `engine`, `market_data`, `strategies`, `trading` | unchanged |

## Compatibility

- Backward compatibility is preserved for existing imports.
- No data schema migration is performed.
- No environment variable migration is performed.
- No command-line migration is performed.
- No NAS or data lake migration is performed.

## Deferred Migration Items

| Item | Reason | Follow-Up |
|---|---|---|
| Physical relayout to `src/quant_lab/**` | Requires bulk move / import rewrite gate. | Future CR. |
| Bulk import rewrite | High blast radius. | Future CR with full dry-run. |
| GitHub remote publication | Explicitly not authorized in CR061. | CR062 recommended. |
| NAS / data lake migration | External state and data safety. | Future CR063 / CR064. |
| Runtime / trading validation | CR046 remains paused. | Explicit runtime authorization only. |

## Operator Notes

Use existing commands unless a future CR changes them. In particular, CR061 does not replace `python -m market_data.cli`.
