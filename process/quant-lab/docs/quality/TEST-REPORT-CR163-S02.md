---
status: complete
version: "1.2"
scope: "CR163-S02 final CP7 reverify"
created_at: "2026-07-11T13:43:28+08:00"
validation_mode: "mixed"
verification_result: "PASS_WITH_RISK"
---

# Test Report: CR163-S02 Final Reverification

## Outcome

`PASS_WITH_RISK`. All 196 tests pass; QA-001..005 are closed; no blocking test gap or new defect remains. Risk is limited to the approved no-resume/advisory-lock/characterization boundaries.

## Verification inventory and trace

| Scope | Scenario / contract | Evidence | Result |
|---|---|---|---|
| full store/core regression | S02 + unchanged S01 contract | 196 tests | PASS |
| prior four findings | path, request binding, process ownership, terminal I/O latch | 10 targeted tests | PASS |
| same-process handle coherence | duplicate receipt and single-writer semantics | 4 targeted tests + direct probe | PASS |
| cross-process ownership | exactly one supported writer | 3 synchronized rounds | PASS |
| graph/crash/no-resume | partial, tamper, broken, cycle, fork, immutable | 7 targeted tests | PASS_WITH_RISK |
| exact-10k claim ceiling | exact count, fixed hash, four metrics, no threshold | 1 isolated test | PASS_WITH_RISK |
| syntax/whitespace | module compile / diff check | direct commands | PASS |

## Commands

| Command | Result |
|---|---|
| full store + contract pytest | `196 passed in 28.26s` |
| QA-001..004 selector | `10 passed, 21 deselected in 1.65s` |
| QA-005 selector | `4 passed, 27 deselected in 0.05s` |
| QA-003 process selector | `3 passed, 28 deselected in 1.53s` |
| partial/graph/immutable selector | `7 passed, 24 deselected in 10.49s` |
| exact-10k selector | `1 passed, 30 deselected in 16.69s`; xunit2 warning only |
| independent two-handle probe | PASS: accepted/idempotent/conflict-blocked; two lines; clean rebuild |
| py_compile / diff check | PASS / PASS |

## Exact-10k evidence

Fixture hash `fc520fc49f5b68f99a57a3da88fe42e2688316d9d8d6d43c7a0cd24fa9183661`; rebuild `1.3383567580003728s`; seal `4.819597302999682s`; peak tracemalloc `17580027`; manifest `504` bytes. No threshold or capacity/readiness inference was applied.

## Findings and coverage gaps

| Finding | Final status | Evidence |
|---|---|---|
| QA-001 | CLOSED | hostile family symlink mutation-order fixture |
| QA-002 | CLOSED | request ref/hash/reason and resolver binding fixtures |
| QA-003 | CLOSED | synchronized independent-process ownership fixtures |
| QA-004 | CLOSED | short/OSError terminal latch fixtures |
| QA-005 | CLOSED | exhaustive same-process two-handle matrix + direct probe |

Coverage gaps requiring rework: none. Prompt/Skill, platform, installer, real runtime/data and external operations are N/A for this local code Story. Remaining limitations are recorded for CP8 rather than waived or hidden.
