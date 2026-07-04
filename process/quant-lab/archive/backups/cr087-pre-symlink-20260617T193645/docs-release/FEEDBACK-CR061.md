# FEEDBACK-CR061

## Observation Signals

| Signal | Threshold | Action |
|---|---|---|
| `import quant_lab` fails | Any failure | Open CR061 rework or follow-up fix. |
| `import quant_lab.<legacy_root>` returns duplicate module state | Any confirmed duplicate-state bug | Rework alias bridge. |
| `uv lock` introduces dependency drift beyond project package rename | Any unexpected dependency diff | Stop follow-up release and review lock. |
| Existing legacy import fails | Any failure | Treat as regression. |
| User command docs become inconsistent | Any reported mismatch | Update README / USER-MANUAL under a doc CR or CP8 fix. |

## Feedback Routing

| Feedback Type | Route |
|---|---|
| Import compatibility bug | CR061 fix / rework |
| Need GitHub remote publication | CR062 |
| Need physical relayout | Future relayout CR |
| Need NAS/data lake migration | CR063 / CR064 |
| Need runtime or trading validation | Explicit runtime authorization / CR046 recovery only |

## Current Known Risks

- Full pytest was not run in CR061 Batch B.
- Bulk physical relayout is still deferred.
- Git remote / push remains not authorized.
