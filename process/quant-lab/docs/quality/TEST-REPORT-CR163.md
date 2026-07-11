---
status: complete
version: "1.0"
change_id: "CR-163"
decision: "PASS_WITH_RISK"
created_at: "2026-07-11T18:15:00+08:00"
updated_at: "2026-07-11T18:38:00+08:00"
---

# CR163 Aggregate Test Report

| Verification layer | Result | Evidence |
|---|---|---|
| Full lineage packet | PASS | 262 passed in 29.59s |
| Admission affected subset | PASS | 220 passed |
| Requirements/scenarios | PASS | 8/8, 12/12 structured and AST-resolvable |
| Producer inventory | PASS | 2/2 chains, 4/4 mappings |
| Deterministic seals | PASS | 10 logical seals → 1 hash |
| Negative integrity classes | PASS | 5/5 blocked |
| Authorization boundaries | PASS | 13/13 installed; guarded path zero; marker blocks; non-marker passes |
| CR155 regression | PASS | actual/synthetic blocked; paper false; reconstruction/backfill 0 |
| Effective/C1 ceiling | PASS | explicit scope boundary; intentionally unavailable/non-computable |
| Five CP7 results | PASS_WITH_RISK aggregate | 1 PASS, 4 PASS_WITH_RISK, 0 blockers |
| Double-repository delivery state | AUTHORIZED_PENDING_EXECUTION | both repos dirty; paired push authorized, not executed |

No runtime, real-data, credential, provider, NAS, broker, simulation, paper/live, statistical-correction, historical reconstruction, publish or release execution test was authorized. Those are excluded capabilities, not untested positive claims.

Conclusion: test evidence supports `READY_WITH_RISK` based on accepted A/B, not design boundaries or dirty-repo process state; this is not `RELEASED`.
