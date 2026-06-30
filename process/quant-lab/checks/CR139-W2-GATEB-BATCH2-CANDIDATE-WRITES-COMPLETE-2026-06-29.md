# CR139 W2 Gate B Batch 2 Candidate Writes Complete - 2026-06-29

## Decision

PASS_WITH_RISK_GATE_B_BATCH2_CANDIDATE_WRITES_COMPLETE

All seven Batch 2 datasets have additive-only CR139 candidate objects written and verified: `liquidity_capacity`, `market_cap`, `adj_factor`, `events`, `prices`, `prices_limit`, and `trade_status`.

## Dataset Summary

| Dataset | Main rows | Quarantine rows | Dropped exact duplicates | Main duplicate rows over unique | Evidence |
|---|---:|---:|---:|---:|---|
| `liquidity_capacity` | 17,093,658 | 1,830 | 0 | 0 | `process/evidence/CR139-W2-GATEB-BATCH2-PURE-METADATA-STRATEGY-UNIFICATION-EXECUTION.index.json` |
| `market_cap` | 17,001,327 | 106,776 | 0 | 0 | `process/evidence/CR139-W2-GATEB-BATCH2-PURE-METADATA-STRATEGY-UNIFICATION-EXECUTION.index.json` |
| `adj_factor` | 17,905,960 | 321,815 | 11,743,055 | 0 | `process/evidence/CR139-W2-GATEB-BATCH2-ADJ_FACTOR-WRITE-EXECUTION.index.json` |
| `events` | 350,753 | 12,134 | 0 | 0 | `process/evidence/CR139-W2-GATEB-BATCH2-EVENTS-WRITE-EXECUTION.index.json` |
| `prices` | 17,013,700 | 401,628 | 11,231,402 | 0 | `process/evidence/CR139-W2-GATEB-BATCH2-PRICES-WRITE-EXECUTION.index.json` |
| `prices_limit` | 19,210,017 | 11,841,929 | 9,910,579 | 0 | `process/evidence/CR139-W2-GATEB-BATCH2-PRICES_LIMIT-WRITE-EXECUTION.index.json` |
| `trade_status` | 13,957,403 | 7,612,504 | 0 | 0 | `process/evidence/CR139-W2-GATEB-BATCH2-TRADE_STATUS-WRITE-EXECUTION.index.json` |

## Boundary

Gate B candidate writes are complete. These are not published current truth. The following remain unauthorized and unexecuted:

- catalog write;
- provider catalog/provider-lake-catalog write;
- published pointer advance;
- physical partition migration;
- credential read;
- NAS operation;
- runtime/QMT/MiniQMT/gateway/trading operation;
- Git remote write;
- legacy deletion;
- rollback deletion.

## Next Gate

Proceed to integrated Gate B evidence review. Gate C catalog/manifest writes, Gate D published pointer advance, and Gate E physical migration require separate explicit authorization.
