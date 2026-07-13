---
change_id: "CR-163"
profile: "compact"
created_at: "2026-07-11T18:15:00+08:00"
updated_at: "2026-07-11T18:38:00+08:00"
---

# CR163 Feedback and Observation Plan

| Signal | Trigger | Route |
|---|---|---|
| lineage present without valid seal/binding | any occurrence | BLOCKER incident; disable consumption and open defect CR |
| orphan/count mismatch/tamper not blocked | any occurrence | BLOCKER incident and rollback |
| raw count changes on retry/replay | any occurrence | HIGH defect; regression and lineage audit |
| sealed v1 bytes/hash change | any occurrence | BLOCKER integrity incident |
| broken/cyclic supersession accepted | any occurrence | BLOCKER integrity incident |
| forbidden-operation category nonzero | any occurrence | authorization incident; stop execution |
| CR155 candidate true or reconstruction >0 | any occurrence | BLOCKER overclaim incident |
| effective/C1 represented as available | any occurrence before separate CR | BLOCKER claim violation |
| first non-fixture/long-running run proposed | before approval | activate FU-CR163-001 or explicitly quantify/accept full rerun loss in that run CR |
| multi-process/multi-host/non-cooperative writer required | before requirement approval | activate FU-CR163-002; otherwise cooperative scope is sufficient |
| >10k support or production SLO/SLA claim proposed | before claim | activate FU-CR163-003 |
| any producer retry loop introduced | same CR, before CP7/present eligibility | complete FU-CR163-004 E2E ordinal fixture |
| effective/C1 computation proposed | before scope expansion | use existing FU-CR161-002 |

Feedback classification: defect → issue/repair CR; new statistical method/runtime/backfill → requirement CR; storage scalability/locking → technical follow-up; documentation misunderstanding → docs fix. This file is an input source and does not itself start a follow-up CR.

No post-release monitoring is active because no real release or runtime operation occurred.

Candidate tracking: `process/changes/CR-163-FOLLOW-UP-TRACKING-2026-07-11.md`. These candidates are not active formal CRs.
