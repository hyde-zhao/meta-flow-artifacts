# CR155 Release Notes

## Decision

Release decision: `READY_WITH_RISK`.

This release adds a daily multifactor baseline strategy artifact and now includes real local governed lake readonly validation evidence. The two reruns are deterministic, but the strategy is not admitted as a paper candidate: `paper_candidate=false` because `economic_significance` and `out_of_sample_validation` are blocked.

## Changes

| Area | Summary |
|---|---|
| Strategy artifact | Added `DailyMultifactorBaselineArtifact` and deterministic JSON-safe serialization. |
| Readonly provenance | Added forbidden operation counters and fixture/static fallback semantics. |
| Validation | Added historical backtest refs, walk-forward split manifest, required metrics and real lake validation evidence. |
| Admission | Added statistical/reliability gate composition and `paper_candidate` derivation; real artifact package is `BLOCKED`. |
| Rerun evidence | Added required metric comparison, strict default tolerance and release wording guardrail; two real lake reruns match exactly. |

## Quality

| Evidence | Result |
|---|---|
| CR155 targeted tests | 11 passed |
| Related CR151/CR154/package regression | 24 passed |
| CP6 result | PASS |
| CP7 result | PASS_WITH_RISK |
| Real lake validation | executed |
| Rerun consistency | PASS |
| Admission package | BLOCKED |
| Paper candidate | false |

## Known Risk

| ID | Severity | Status |
|---|---|---|
| I-CR155-REAL-LAKE-ADMISSION-BLOCKED | HIGH | Requires CP8 close decision. |

## Not Authorized

CP8 approval does not authorize real lake write, catalog pointer mutation, NAS/provider/credential access, runtime, paper/live/trading/broker operation, external framework execution, store/registry write, publish or production deployment.
