# FEAT-27 TEST-PLAN

| 组合 | 预期 |
|---|---|
| all mandatory PASS | PASS |
| BH PASS + PBO FAIL | FAIL |
| PBO PASS + DSR unavailable | TYPED_UNAVAILABLE |
| any BLOCKED / orphan ref | BLOCKED |
| consumer already worse | remains worse |

Projection coverage=3/3；new gate count=0；overclaims=0；CR155 blocked=1/1。

