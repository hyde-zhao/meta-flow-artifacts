---
status: complete
version: "1.2"
story_id: "CR163-S05-integrity-recovery-permission-regression"
route_to: "verified-with-risk"
updated_at: "2026-07-11T17:56:00+08:00"
---

# Fix Status: CR163-S05 Final Reverification

| ID | Initial status | Final status | Evidence |
|---|---|---|---|
| QA-001 | BLOCKER | RESOLVED in round 2 | 13/13 installed; marker blocks before original; non-marker pass-through |
| QA-002 | HIGH | RESOLVED in round 1 | five schema/value negative classes |
| QA-003 | HIGH | RESOLVED in round 1 | 12/12 structured AST trace and bad-ref fail closed |

No further fix input remains. Retain the accepted future retry-loop integration risk and upstream claim/recovery ceilings. No workflow-health escalation.
