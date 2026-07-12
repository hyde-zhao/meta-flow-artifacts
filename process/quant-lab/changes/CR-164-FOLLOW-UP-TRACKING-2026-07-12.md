---
source_cr: "CR-164"
status: "open"
created_at: "2026-07-12T21:51:00+08:00"
created_by: "host-orchestrator"
checkpoint_source: "CP8"
release_context_ref: "process/release/RELEASE-CONTEXT-CR164-MULTIPLE-TESTING-PBO-DSR-EVIDENCE.yaml"
---

# CR-164 Follow-up Tracking

## Purpose and status

本台账只登记 CP8 follow-up candidates。下列项目均为 `lifecycle_status=candidate`，未创建或启动正式 CR，不授权实现、真实数据、runtime、交易、部署或远端写入。

## Structured candidates

```yaml
follow_up_items:
  - id: FU-CR164-001
    title: Independent Agent/model verification
    lifecycle_status: candidate
    owner: quality owner
    activation_trigger: "the user lifts the no-subagent constraint, or an external audit requires independent reviewer isolation"
    required_before: "claiming independent Agent/model review for CR164"
    closure_condition: "independent CP7 rerun confirms the frozen evidence contract and affected regression"
    non_authorization: "does not authorize runtime, real data, deployment or remote write"
    source_risk: R-CR164-INLINE-INDEPENDENCE

  - id: FU-CR164-002
    title: Repository-wide workflow and artifact hygiene repair
    lifecycle_status: "closed"
    readiness_status: "ready_with_risk"
    gate_status: "closed"
    formal_cr_path: "process/changes/CR-165.md"
    related_active_cr: "CR-165"
    blocked_by: "closed_by=CR-165"
    closed_by: "CR-165"
    owner: meta-flow maintenance owner
    activation_trigger: "before requiring a repository-wide green suite as a release invariant"
    required_before: "claiming full repository test pass"
    closure_condition: "the 14 classified pre-existing failures are fixed or rebaselined through an approved CR and the full suite is green"
    non_authorization: "does not expand CR164 functional scope or authorize release"
    source_risk: R-REPO-GLOBAL-14

  - id: FU-CR164-003
    title: Automatic stationary-bootstrap block-length selector
    lifecycle_status: candidate
    owner: statistical methodology owner
    activation_trigger: "an approved requirement replaces explicit fixed window with an automatic selector"
    required_before: "using or claiming an automatic Politis-Romano-style selection method"
    closure_condition: "method, provenance, determinism and golden-vector evidence pass CP3/CP7"
    non_authorization: "does not authorize real-data characterization or runtime execution"
    source_boundary: "fixed-window bootstrap remains the CR164 contract"

  - id: FU-CR164-004
    title: Effective-trial estimation and C1 computation
    lifecycle_status: candidate
    owner: strategy-admission methodology owner
    activation_trigger: "an approved scope requires effective rather than raw trial count"
    required_before: "populating effective_trial_count or claiming C1 computable"
    closure_condition: "effective-count method, non-alias schema, validation and consumer projection pass CP3/CP7"
    non_authorization: "does not allow raw_trial_count to be relabeled as effective count"
    source_boundary: "CR164 effective fields remain typed_unavailable"
```

## Candidate summary

| ID | Status | Trigger | Non-authorization |
|---|---|---|---|
| FU-CR164-001 | candidate | independent audit required or no-subagent lifted | no runtime/data/remote-write authority |
| FU-CR164-002 | closed via CR-165 | repository-wide green suite becomes mandatory | no CR164 scope expansion |
| FU-CR164-003 | candidate | automatic selector requirement approved | no real-data/runtime authority |
| FU-CR164-004 | candidate | effective-trial/C1 scope approved | raw count cannot become effective count |

## Risk acceptance

The user accepted `R-CR164-INLINE-INDEPENDENCE` and `R-REPO-GLOBAL-14` with `approve CP8`. Acceptance permits local `READY_WITH_RISK` closure only; it is not a waiver for missing evidence and grants no true release authority.
