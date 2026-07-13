# CR-045 CP7 R3 Test Report

Decision: `NEEDS_REWORK`
Dispatch: `DISPATCH-CR045-CP7-REVERIFY-R3-QA`

CP2 and CP5 remain recovery approvals recorded after historical CP6.

## Results

| Scope | Result |
|---|---|
| State-transition unit suite | 11/11 passed |
| CR-045 focused suite | 125/125 passed |
| Full repository regression | 320/320 passed |
| Pass-like × stale-failure matrix | PASS |
| Pending CP8 positive path | PASS |
| Matching failure decisions | PASS |
| `BLOCKED × authorization_required` | FAIL: valid cause rejected |
| `BLOCKED × workflow_health_threshold` | FAIL: valid cause rejected |
| Historical route/result/dispatch/lifecycle compatibility | PASS |

## Finding closure and gap

- `CR045-F-001`: closed.
- `CR045-F-002`: closed.
- `CR045-F-003-R2`: closed.
- `CR045-F-004-R3`: open HIGH. Existing tests cover authorization/workflow-health only for PASS, not for the semantically natural `BLOCKED` decision.

## Guardrail

Delivery guardrail fails only on ignored test caches. It remains non-CR CP8 hygiene.
