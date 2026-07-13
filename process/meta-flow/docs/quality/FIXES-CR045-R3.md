# CR-045 CP7 R3 Rework Input

Status: `required`

The three prior findings are independently closed. CP2 and CP5 remain recovery approvals after historical CP6.

## `CR045-F-004-R3`

Replace the one-to-one failure decision mapping with decision-compatible stop-reason sets. Required minimum behavior:

- `BLOCKED` accepts `blocked`, `authorization_required`, and `workflow_health_threshold`.
- `NEEDS_REWORK` accepts `needs_rework` and rejects unrelated failure causes.
- `NEEDS_DESIGN_CLARIFICATION` accepts `needs_design_clarification` and rejects unrelated failure causes.
- `FAIL` retains the intended generic failure mapping unless the project contract explicitly permits a more specific cause.
- Pass-like decisions continue rejecting stale failure reasons and accepting pending CP8 or explicitly legitimate interruptions.

Add isolated positive/negative fixtures, rerun state-transition, CP result consistency, focused tests, and full regression.
