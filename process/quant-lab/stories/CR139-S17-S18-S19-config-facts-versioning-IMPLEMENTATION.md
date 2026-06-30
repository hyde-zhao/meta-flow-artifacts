---
story_ids: ["CR139-S17", "CR139-S18", "CR139-S19"]
change_id: "CR-139"
stage: "CP6"
status: "implemented"
implemented_at: "2026-06-30T14:15:00+08:00"
implemented_by: "host-orchestrator-inline"
---

# CR139-S17/S18/S19 Config Facts Versioning Implementation

## Implementation Scope

This batch adds versioned config fact metadata to existing contracts:

| Story | Contract | Change |
|---|---|---|
| S17 | `BenchmarkDefinition` | Added `version`, `effective_from`, `release_id`, `risk_free_curve_ref`. |
| S18 | `CommissionSchedule` | Added `version`, `effective_from`, `release_id`, `slippage_bps`, `stamp_duty_rate`, `transfer_fee_rate`. |
| S19 | `PortfolioRiskPolicy` | Added `effective_from`, `release_id`, `universe_policy_ref`, `delisting_policy`, `st_policy`; validation now requires version metadata in mapping payloads. |

## Verification

| Command | Result |
|---|---|
| `uv run --python 3.11 pytest -q tests/test_cr139_config_facts_versioning.py tests/test_cr138_gateway_query_calendar_commission_pnl.py tests/test_stage2_mature_multifactor_framework.py tests/test_stage3_mature_multifactor_research.py` | PASS, 28 passed in 0.76s |

## Boundary Check

Fixture/static only. No real `config_facts/` write, lake/catalog/manifest/pointer/NAS/provider/runtime/trading/credential/Git operation was performed.
