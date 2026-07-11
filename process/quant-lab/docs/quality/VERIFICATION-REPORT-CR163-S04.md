---
status: complete
version: "1.2"
story_id: "CR163-S04-existing-admission-projection"
feature_id: "FEAT-22"
validation_mode: "static-only"
verification_result: "PASS_WITH_RISK"
reverification_round: 2
source_story: "process/stories/STORY-CR163-S04-existing-admission-projection.md"
source_implementation: "process/stories/STORY-CR163-S04-existing-admission-projection-IMPLEMENTATION.md"
created_by: "meta-qa-critical"
created_at: "2026-07-11T16:24:00+08:00"
updated_at: "2026-07-11T16:48:00+08:00"
---

# CR163-S04 Final CP7 Reverification Report

## 1. Decision

`PASS_WITH_RISK`. QA-CR163-S04-001..004 are independently closed within the workflow-health threshold. The complete affected 220-test subset, CR151 trusted-lineage threshold matrix, missing-lineage gates, CR154 trusted-raw/effective-C1 ceiling, 18-case negative selector, four-case binding mismatch probe, actual CR155 absent/blocked paths, syntax and whitespace checks all pass. No new defect, waiver, design clarification or design delta remains.

The risk qualifier preserves the approved claim ceiling: effective trial count is unavailable, its ref/method are empty, and CR154 C1/deflated-performance claims remain non-computable and BLOCKED until a separately approved later CR supplies a method. This is intended safe behavior, not an implementation defect.

No source/test fix, Host state update, external operation, historical backfill, statistical calculation or release action was performed by QA.

## 2. Scope and authorization

- In scope: QA-001..004, three existing consumers, S01 validation binding, CR151 thresholds/manual reconciliation, CR154 raw/effective boundary, package status/auth, actual CR155 candidate derivation, malformed/mismatch/tamper matrices, gate inventory and affected regression baseline.
- Out of scope: real research/data/lake/NAS/provider/credentials/QMT/broker/trading/statistical/backfill/publication/release operations.
- SUT: local code project; static source review plus deterministic in-memory and native unit/fixture execution.
- CP6 rework round 2 return/evidence is consistent with observed behavior: production remains fail closed; the migration updates the two affected native test modules.

## 3. Verification object inventory

| Object | Method | Result |
|---|---|---|
| validation-bound adapter | source review, S01 suite, untrusted/bare/bound-mismatch probes | PASS |
| CR151 statistical consumer | trusted PASS/FAIL/NEEDS_REVIEW, missing BLOCKED, manual matrix | PASS |
| CR154 Gate1 consumer | trusted raw lineage and missing-lineage cases | PASS_WITH_RISK — correctly BLOCKED by effective/C1 ceiling or missing lineage |
| admission package | canonical equality, status/auth matrix, forged inputs | PASS |
| actual CR155 package | real compose/derive with absent and blocked lineage | PASS — BLOCKED, candidate false, backfill 0 |
| affected native regression baseline | six-suite execution | PASS — 220/220 |
| CP6 return/evidence and implementation report | structural/semantic review | PASS |

## 4. Traceability and finding closure

| Finding / contract | Result | Independent evidence |
|---|---|---|
| QA-001 untrusted Mapping/bare DTO cannot mint present | CLOSED | parameterized three-consumer cases, bare DTO case, 4/4 bound mismatch probe |
| positive only through manifest+validation | PASS | trusted wrapper delegates to S01 projector; target/count/status/manifest mismatch fail closed |
| QA-002 exact canonical equality | CLOSED | CR151 == CR154 == package full projection dict |
| manual count local-only | PASS | absent/match/mismatch outside canonical projection; mismatch gate BLOCKED |
| effective unavailable / C1 non-computable | PASS_WITH_RISK | count None, ref/method empty, CR154 BLOCKED |
| status only worsens / auth unchanged | PASS | four-status/four-flag matrix |
| no new gate | PASS | six existing gate IDs, no lineage gate ID |
| QA-003 actual CR155 candidate closes | CLOSED | real PASS/true becomes BLOCKED/false for absent and blocked lineage; derive false; backfill 0 |
| QA-004 native test-contract migration | CLOSED | affected subset 220/220; fail-closed negative matrices remain green |

## 5. Layered execution evidence

| Command / probe | Result |
|---|---|
| full affected six-suite subset | PASS — 220 passed in 0.19s |
| CR151 trusted PASS/FAIL/NEEDS_REVIEW + missing BLOCKED selector | PASS — 4 passed, 7 deselected in 0.05s |
| CR154 Gate1 selector | PASS — 1 passed, 4 deselected in 0.03s |
| S04 malformed/mismatch/manual/status/CR155 selector | PASS — 18 passed, 1 deselected in 0.06s |
| direct binding mismatch matrix | PASS — hash/count/non-PASS/missing-manifest 4/4 BLOCKED, raw absent |
| direct actual CR155 blocked-lineage probe | PASS — BLOCKED, candidate false, derive false, backfill 0 |
| `py_compile` three consumers | PASS |
| `git diff --check` | PASS |

Final production hashes: CR151 `3a00ef6903d364135c9b11586633437c624474004491ff00041d1c24478c8803`; CR154 `1480ab6556ebfca2c0c0c96a94c08863ecb2a3f54bfc02dca56af825f16aefc5`; package `db8df0d2d1262defbcda425d15e7574894397932eb0f21db6cddab6ca51413a5`.

## 6. Manual / semantic review

| Dimension | Result | Blocking | Note |
|---|---|---|---|
| Requirement consistency | PASS | no | confirmed validation-bound/fail-closed contract implemented |
| Scenario coverage | PASS | no | positive, missing, malformed, mismatch, tamper, effective, status and CR155 covered |
| Error actionability | PASS | no | stable untrusted/malformed/target/manual reasons |
| Happy/negative balance | PASS | no | threshold outcomes coexist with missing-lineage BLOCKED tests |
| Claim ceiling | PASS_WITH_RISK | no | raw lineage is not effective/statistical proof |
| Permission boundary | PASS | no | all verification local; forbidden operations remain zero |

## 7. Remaining risk, workflow health and route

| Risk | Level | Acceptance / downstream action |
|---|---|---|
| effective trial count and C1 remain unavailable | accepted HIGH claim limitation | carry to CP8; do not enable deflated-performance/admission-readiness wording without separately approved method/evidence |

Workflow-health escalation is not required: final rework round 2 closed QA-004 without a new defect and stayed within threshold. Route Story to `verified-with-risk` and carry only the effective/C1 limitation downstream.
