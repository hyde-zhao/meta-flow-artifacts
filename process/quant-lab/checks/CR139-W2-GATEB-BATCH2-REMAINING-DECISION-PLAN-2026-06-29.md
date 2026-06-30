# CR139 W2 Gate B Batch 2 Remaining Decision Plan - 2026-06-29

## Decision

PASS_WITH_RISK_REMAINING_DECISION_PLAN_READY_WRITE_STILL_REQUIRES_EXPLICIT_DATASET_AUTHORIZATION

The post-`adj_factor` planning gate completed for `events`, `prices`, `prices_limit`, and `trade_status`. This is read-only planning. No lake write, candidate delete, catalog write, pointer advance, physical migration, credential read, NAS/runtime/trading operation, Git remote write, legacy deletion, or rollback was executed.

## Dataset Decisions

| Dataset | Business-conflict groups | Planned main rows | Planned quarantine rows | Readiness |
|---|---:|---:|---:|---|
| `events` | 774 | 350,753 | 12,134 | `ready_for_explicit_events_write_authorization_after_review` |
| `prices` | 79,958 | 17,013,700 | 401,628 | `blocked_until_schema_normalization_decision_and_single_dataset_authorization` |
| `prices_limit` | 499,787 | 19,210,017 | 11,841,929 | `blocked_until_single_dataset_authorization_after_events_or_prices_schema_review` |
| `trade_status` | 3,692,105 | 13,957,403 | 7,612,504 | `blocked_until_single_dataset_authorization; recommend after events because conflict scale is high` |

## Recommended Sequence

1. `events`: next explicit write candidate; small table, quarantine 774 business-conflict groups, resolve 10,583 metadata-only groups.
2. `prices`: approve union schema with null fill before write; business-conflict groups quarantine.
3. `prices_limit`: single-dataset split write only; do not batch with other large tables.
4. `trade_status`: single-dataset split write only; largest conflict bucket, keep separate.

## Prices Schema Policy Recommendation

Use union schema with null fill for missing columns. Observed variants: 3. Missing columns are primarily `volume`, `amount`, and `available_at_rule` in older/smoke sources. This policy preserves all known columns and avoids silently dropping liquidity-relevant price fields.

## Evidence

- Detail: `process/evidence/CR139-W2-GATEB-BATCH2-REMAINING-DECISION-PLAN-2026-06-29.json`
- Index: `process/evidence/CR139-W2-GATEB-BATCH2-REMAINING-DECISION-PLAN.index.json`
- Source split planning: `process/evidence/CR139-W2-GATEB-BATCH2-REMAINING-SPLIT-PLANNING.index.json`

## Next

Recommended next explicit authorization: `events` split candidate write only. `prices`, `prices_limit`, and `trade_status` remain blocked until their named dataset authorization is reviewed.
