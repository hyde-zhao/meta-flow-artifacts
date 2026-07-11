---
status: complete
version: "1.2"
story_id: "CR163-S02-recorder-seal-supersession"
feature_id: "FEAT-20"
validation_mode: "mixed"
verification_result: "PASS_WITH_RISK"
source_story: "process/stories/STORY-CR163-S02-recorder-seal-supersession.md"
source_implementation: "process/stories/STORY-CR163-S02-recorder-seal-supersession-IMPLEMENTATION.md"
created_by: "meta-qa-critical"
created_at: "2026-07-11T13:20:41+08:00"
updated_at: "2026-07-11T13:43:28+08:00"
---

# CR163-S02 Final CP7 Reverification Report

## 1. Decision

`PASS_WITH_RISK`. QA-CR163-S02-001 through -005 are all independently closed within the workflow-health threshold. The complete 196-test suite, prior-finding selectors, same-process two-handle matrix, synchronized cross-process ownership, interrupted-write latch, immutable graph/crash/no-resume set, syntax/whitespace checks and exact-10k characterization all pass. No new implementation defect, waiver, design clarification or design delta remains.

The risk qualifier carries approved first-slice limitations into CP8: malformed ledgers cannot resume in place; cross-process exclusion is a cooperative local advisory lock; exact-10k results are observations rather than capacity/readiness evidence.

## 2. Scope and authorization

- In scope: append/idempotency, QA-001..005, path containment, supersession request binding, same/cross-process writer ownership, short-write/OSError latch, prefix replay, immutable publish and graph consistency, crash/no-resume, exact-10k claim ceiling.
- Out of scope and not executed: source/test fixes; real lineage/data/lake/NAS/provider/credentials/runtime/statistical/backfill/release/publish operations.
- SUT: local code project; validation mode `mixed` (local fixture execution plus static/manual review).
- Dispatch/evidence: final CP7 reverify dispatch is present; CP6 rework round 2 return/evidence are valid; no design delta is required.

## 3. Verification object inventory

| Object | Method | Result |
|---|---|---|
| store implementation | source review, py_compile, full and targeted tests, direct two-handle probe | PASS |
| S02 fixture suite | negative coverage review and selectors | PASS — 31 tests contribute to 196 combined |
| S01 integration | combined contract suite | PASS |
| process-authoritative handle state | four native tests + direct identical/conflict/reopen probe | PASS |
| path/request/process/I/O rework | QA-001..004 selector and source review | PASS |
| immutable graph/crash/no-resume | targeted selector | PASS |
| exact-10k characterization | isolated test/JUnit properties and wording review | PASS_WITH_RISK — observation only |
| CP6 implementation/return/evidence | structural/semantic check | PASS |

## 4. Traceability matrix

| Scenario / requirement | Design contract | Verification | Status | Risk |
|---|---|---|---|---|
| SC-CR163-P03; REQ-004/006/007 | deterministic and target-bound immutable seal | full suite + graph/prefix fixtures | PASS | N/A |
| SC-CR163-R01/T01 | bound v1→v2 chain; tamper/break/cycle/fork fail closed | QA-002 + graph subset | PASS | N/A |
| SC-CR163-A01; REQ-008 | root containment and zero forbidden writes | QA-001 hostile family symlink | PASS | N/A |
| SC-CR163-N02; REQ-002/007 | duplicate identity idempotent or blocked before append | native and direct two-handle matrices | PASS | N/A |
| LLD §2.2 single writer | coherent same-process state; cross-process fail closed | QA-003/005 | PASS | cooperative local lock boundary |
| LLD §8 recovery | short/OSError latch; malformed evidence retained; no resume | QA-004 + partial/no-resume | PASS_WITH_RISK | restart under new family |
| LLD §§2.2/8/14 | exactly 10k, four observations, no threshold/claim | isolated characterization | PASS_WITH_RISK | no capacity inference |

## 5. Design contract verification and finding closure

| Finding / contract | Status | Independent evidence |
|---|---|---|
| QA-001 path mutation order | CLOSED | hostile family symlink rejected before outside mutation |
| QA-002 request/manifest consistency | CLOSED | ref/hash/reason mismatch matrix and resolver replay |
| QA-003 cross-process ownership | CLOSED | three synchronized spawn rounds; exactly one writer |
| QA-004 terminal append failure | CLOSED | short write and OSError propagate to later submit/seal |
| QA-005 same-process handle coherence | CLOSED | same replay idempotent; conflict blocked with one ledger fact; distinct events serialize; final close/reopen rebuilds |
| immutable graph/prefix validation | PASS | tamper, count mismatch, broken, cycle, fork, missing validation and preserved v1 prefix |
| no-repair/no-resume surface | PASS_WITH_RISK | malformed tail retained; old family blocked; new family required |

## 6. Layered execution evidence

| Check | Result |
|---|---|
| full S01+S02 suite | PASS — `196 passed in 28.26s` |
| QA-001..004 selector | PASS — `10 passed, 21 deselected in 1.65s` |
| QA-005 two-handle selector | PASS — `4 passed, 27 deselected in 0.05s` |
| QA-003 cross-process selector | PASS — `3 passed, 28 deselected in 1.53s` |
| partial/graph/immutable selector | PASS — `7 passed, 24 deselected in 10.49s` |
| exact-10k selector | PASS — `1 passed, 30 deselected in 16.69s`; xunit2 property warning only |
| direct same-process probe | PASS — original accepted, identical replay idempotent, conflicting replay blocked, two lines, shared state, clean reopen |
| py_compile / git diff check | PASS / PASS |

Exact-10k final rerun: Python 3.11.15; Linux WSL2 x86_64; fixture hash `fc520fc49f5b68f99a57a3da88fe42e2688316d9d8d6d43c7a0cd24fa9183661`; rebuild `1.3383567580003728s`; seal `4.819597302999682s`; peak tracemalloc `17580027` bytes; manifest `504` bytes. These are single-machine synthetic observations, not supported capacity, SLA, production-scale or long-search recoverability evidence.

## 7. Manual / semantic review

| Check | Result | Blocking | Note |
|---|---|---|---|
| Requirement consistency | PASS | no | all confirmed MUST behavior verified |
| Scenario coverage | PASS | no | positive, negative, recovery, path and concurrency paths covered |
| Documentation usability | PASS_WITH_RISK | no | no-resume and 10k ceiling are explicit and must remain visible downstream |
| Error actionability | PASS | no | deterministic conflict/failure reasons verified |
| Happy/negative balance | PASS | no | all five discovered negative classes have permanent regressions |
| Prompt/platform/install | N/A | no | code-only local store |

## 8. Remaining risks, workflow health and route

| Risk | Level | Acceptance / downstream action |
|---|---|---|
| malformed long-running ledger loses native continuation | HIGH product limitation | carry to CP8; disclose restart-from-declaration/new-family behavior before any long task enablement |
| advisory lock cannot stop a deliberately non-cooperating external writer | LOW boundary | supported writers must use this store; broader multi-writer need requires a new storage ADR/CR |
| exact-10k measurements may be misread as capacity | MEDIUM claim risk | retain observation-only language and zero numeric gate |

Workflow-health escalation: not required. This final within-threshold reverification found no new defect. Route Story to `verified-with-risk` and carry the three explicit limitations into CP8 Decision Brief/release-readiness review.
