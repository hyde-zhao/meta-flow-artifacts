---
change_id: "CR-164"
release_decision: "READY_WITH_RISK"
release_artifact_profile: "compact"
created_at: "2026-07-12T21:48:00+08:00"
---

# CR-164 Multiple-testing / PBO / DSR Evidence — Release Notes

## Readiness

`READY_WITH_RISK` for CP8 human review. This is an internal release candidate, not `RELEASED`; no version, tag, publish, deployment, runtime/data access or Git remote write has been executed or authorized.

## Delivered behavior

- Immutable statistical evidence input/method/summary contracts with deterministic canonical hash and fail-closed validation.
- Stable BH and seeded fixed-window stationary-bootstrap WRC/SPA calculators with explicit provenance.
- Stable CSCV/PBO and DSR with `dsr_input_method=raw_trial_count`; effective-trial fields remain unavailable.
- Conservative mandatory-method aggregation: `BLOCKED > FAIL > TYPED_UNAVAILABLE > PASS`.
- Trusted projection into three existing admission consumers without creating a parallel gate or improving a worse upstream state.
- CR155 remains blocked and not paper-candidate when required native lineage is absent.

## Evidence and risks

Affected regression is 109/109 PASS; provenance is 7/7; forbidden operation count is 0. Risks are the Host-inline QA independence limitation and 14 pre-existing full-suite workflow/artifact failures. See `docs/quality/VERIFICATION-REPORT-CR164.md`.
