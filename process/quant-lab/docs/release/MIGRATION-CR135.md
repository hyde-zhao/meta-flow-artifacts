---
title: "CR135 Migration"
source_cr: "CR-135"
status: "ready_with_risk"
profile: "compact"
created_at: "2026-06-23"
---

# CR135 Migration

## Data / State Migration

N/A. CR135 does not migrate persisted project state, market data, lake data, catalog pointers or external runtime data.

## Interface Compatibility

| Surface | Change | Compatibility |
|---|---|---|
| `RunSpec` | Adds optional `bundle_output_path` | Compatible |
| CLI | Adds `--bundle-output` and `--inspect-bundle` | Compatible |
| Bundle schema | Adds `cr135-run-artifact-bundle-v1` manifest | New optional artifact |

## Operator Action

No migration action is required. Existing runner calls continue to work without `bundle_output_path`.
