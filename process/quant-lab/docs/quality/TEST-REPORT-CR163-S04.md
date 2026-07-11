---
status: complete
version: "1.2"
story_id: "CR163-S04-existing-admission-projection"
decision: "PASS_WITH_RISK"
reverification_round: 2
updated_at: "2026-07-11T16:48:00+08:00"
---

# Test Report: CR163-S04 Final Reverification

## Summary

All affected automated and direct negative checks pass. Result: `PASS_WITH_RISK`, with only the intentional effective-trial/C1 claim ceiling retained.

## Coverage matrix

| Layer | Result | Evidence |
|---|---|---|
| full affected subset | PASS | 220 passed in 0.19s |
| CR151 trusted thresholds | PASS | PASS/FAIL/NEEDS_REVIEW preserved under trusted lineage |
| CR151 missing lineage | PASS | explicit BLOCKED with typed unavailable projection |
| CR151 manual reconciliation | PASS | absent/match/mismatch local; mismatch blocks; canonical unchanged |
| CR154 trusted raw lineage | PASS_WITH_RISK | raw present, effective unavailable/C1 non-computable keeps BLOCKED |
| CR154 missing lineage | PASS | typed unavailable and BLOCKED |
| untrusted/bare/malformed/effective injection | PASS | all fail closed across three consumers |
| validation binding mismatch | PASS | hash/count/status/manifest 4/4 blocked |
| canonical equality | PASS | exact full dict across 3/3 consumers |
| status/auth/no-new-gate | PASS | monotonic, flags unchanged, six exact gates |
| actual CR155 absent/blocked | PASS | true candidate becomes false/BLOCKED; derive false; backfill zero |
| syntax/whitespace | PASS | py_compile and diff check |

## Execution summary

- Full six-suite subset: `220 passed in 0.19s`.
- CR151 threshold/missing selector: `4 passed, 7 deselected in 0.05s`.
- CR154 Gate1 selector: `1 passed, 4 deselected in 0.03s`.
- S04 negative selector: `18 passed, 1 deselected in 0.06s`.
- Binding mismatch direct probe: `4/4 blocked`.
- Actual CR155 blocked-lineage probe: `BLOCKED / false / backfill 0`.
- `py_compile`: PASS; `git diff --check`: PASS.

## Coverage gaps and risk

Real runtime/data/statistical execution is intentionally N/A and forbidden. No effective trial method is implemented or tested; C1 remains non-computable. This limitation must remain visible in CP8 and is not waived.

## Stage decision

`PASS_WITH_RISK`; route to `verified-with-risk`.
