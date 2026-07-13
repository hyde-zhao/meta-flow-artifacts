# CR-045 CP7 R2 Fixes

Status: `required`

Both R1 HIGH findings are independently closed. CP2 and CP5 remain ex-post recovery approvals after historical CP6.

## CR045-F-003-R2

Update state-transition validation so a pass-like CP decision cannot satisfy its expected next route with a stale failure/rework stop reason. At minimum, CP7 `PASS`/`PASS_WITH_RISK` must reject `needs_rework`, `needs_design_clarification`, and `blocked` when the route requires pending CP8. Preserve legitimate behavior for the actual next required gate and explicitly justified authorization/workflow-health stops.

Add a decision × stop-reason negative matrix and rerun state-transition, CP result consistency, the focused suite, and full regression.

Host Orchestrator repository hygiene remains separate: remove or otherwise exclude ignored Python cache artifacts, then rerun the delivery guardrail.
