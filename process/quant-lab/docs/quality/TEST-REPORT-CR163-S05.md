---
status: complete
version: "1.2"
story_id: "CR163-S05-integrity-recovery-permission-regression"
decision: "PASS_WITH_RISK"
reverification_round: 2
updated_at: "2026-07-11T17:56:00+08:00"
---

# Test Report: CR163-S05 Final Reverification

## Result

`PASS_WITH_RISK`. All required focused, adversarial and full-packet checks pass.

| Layer | Result | Evidence |
|---|---|---|
| installation manifest | PASS | exact 13 categories, target+predicate for each |
| simultaneous guarded path | PASS | 13/13 observed zero |
| marker invocation | PASS | 13/13 via actual patched API; original spy 0 |
| non-marker invocation | PASS | 13/13 pass-through; safe originals 13; counters zero |
| schema negatives | PASS | missing/unknown/bool/negative/nonzero 5/5 |
| trace | PASS | 12/12 AST-resolvable; bad ref blocks |
| integrity/recovery | PASS | 10→1, five negatives, v1/v2/broken/cyclic |
| CR155 and claim ceiling | PASS | 2/2 blocked; no reconstruction/effective/C1/runtime-ready claims |
| focused S05 | PASS | 33 passed in 0.93s |
| full packet | PASS | 262 passed in 29.59s |
| process/static | PASS | return/evidence OK; diff clean |

Real systems remain N/A and forbidden. Residual risks are future retry-loop integration and upstream claim/recovery limitations, not current failures.
