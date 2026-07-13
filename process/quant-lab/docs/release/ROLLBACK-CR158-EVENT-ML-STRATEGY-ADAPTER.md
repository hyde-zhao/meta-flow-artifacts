---
change_id: "CR-158"
status: "draft-pending-cp8-approval"
created_at: "2026-07-05T18:55:00+08:00"
---

# Rollback - CR158 Event + ML Strategy Adapter

## Rollback Scope

Rollback is source-level only because CR158 does not perform deployment, migration, data writes, runtime registration, publish, or remote Git operations.

## Source Rollback Candidates

| Path | Action if CP8 rejected or post-review issue found |
|---|---|
| `engine/strategy_type_adapters.py` | Remove or revise the adapter contract module in a follow-up patch. |
| `tests/research/test_*strategy_adapter*.py`, `tests/research/test_strategy_type_adapter_core.py` | Remove or revise tests that depend on the adapter contract. |
| `docs/quality/*CR158*.md` | Supersede with corrected verification evidence. |
| `process/checks/CP6-*CR158*.json`, `process/checks/CP7-*CR158*.json`, `process/checkpoints/CP8-*CR158*.md` | Preserve as audit history; add a new corrective checkpoint rather than deleting historical evidence. |

## Data and Runtime Rollback

N/A. No real data, lake, NAS, provider, credential, runtime, trading, broker, registry, publish, external framework, or Git remote operation was performed.
