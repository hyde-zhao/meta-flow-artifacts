# CR-045 CP7 R4 Test Report

Decision: `PASS`
Dispatch: `DISPATCH-CR045-CP7-REVERIFY-R4-QA`

CP2 and CP5 remain recovery approvals after historical CP6.

## Results

| Scope | Result |
|---|---|
| Exact decision × stop-reason matrix | 38/38 expected outcomes |
| State-transition suite | 11/11 passed |
| CR-045 focused suite | 125/125 passed |
| Full repository regression | 320/320 passed |
| Route/result/ledger/state/lifecycle/governance checks | PASS |
| Historical fallback and real-dispatch compatibility | PASS |
| Delivery guardrail | known non-CR cache hygiene failure |

## Finding status

- `CR045-F-001`: closed.
- `CR045-F-002`: closed.
- `CR045-F-003-R2`: closed.
- `CR045-F-004-R3`: closed.

No material CR-045 coverage gap remains. Cache cleanup is a separate CP8 prerequisite.
