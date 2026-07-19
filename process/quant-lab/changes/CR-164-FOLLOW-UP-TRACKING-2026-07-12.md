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
    title: Effective-trial offline estimator methodology
    lifecycle_status: closed
    readiness_status: ready_with_risk
    gate_status: closed
    formal_cr_path: "process/changes/CR-173.md"
    related_active_cr: ""
    blocked_by: "closed_by=CR-173"
    closed_by: "CR-173"
    activated_by: "CR-172 CP2 DQ-CR172-002=separate_methodology_cr; DQ-CR172-004=PATH-B; related_active_cr=CR-173"
    conflict_precheck_ref: "process/checks/CR173-CONFLICT-PRECHECK.json"
    owner: strategy-admission methodology owner
    activation_trigger: "an approved scope requires effective rather than raw trial count"
    required_before: "using a standalone spectral effective-dimensionality result or satisfying CR172's methodology prerequisite; it is necessary but not sufficient for populating public C1 or claiming C1 computable"
    closure_condition: "effective-dimensionality method, non-alias standalone schema, deterministic validation and public-projection boundary pass CP3/CP5/CP7/CP8"
    non_authorization: "does not allow raw_trial_count to be relabeled as effective count and does not authorize public C1 projection or a c1_computable claim"
    source_boundary: "CR164 effective fields and all current public C1/Gate1/admission consumers remain typed_unavailable"
```

## Candidate summary

| ID | Status | Trigger | Non-authorization |
|---|---|---|---|
| FU-CR164-001 | candidate | independent audit required or no-subagent lifted | no runtime/data/remote-write authority |
| FU-CR164-002 | closed via CR-165 | repository-wide green suite becomes mandatory | no CR164 scope expansion |
| FU-CR164-003 | candidate | automatic selector requirement approved | no real-data/runtime authority |
| FU-CR164-004 | closed via CR-173 | CR-172 CP2 PATH-B + separate methodology approved | offline methodology only；raw count cannot become effective count；no real-data/runtime authority |

## Risk acceptance

The user accepted `R-CR164-INLINE-INDEPENDENCE` and `R-REPO-GLOBAL-14` with `approve CP8`. Acceptance permits local `READY_WITH_RISK` closure only; it is not a waiver for missing evidence and grants no true release authority.

## Activation history

| 时间 | 项目 | 事件 | 证据 | 授权边界 |
|---|---|---|---|---|
| 2026-07-16 11:36 +08:00 | FU-CR164-004 | 由 CR-172 CP2 批准的 PATH-B / separate methodology 决策转为正式 `CR-173`；fresh conflict precheck PASS | `process/checks/CR173-CONFLICT-PRECHECK.json`、`process/checkpoints/CP2-CR172-REQUIREMENTS-BASELINE.md`、`process/changes/CR-173.md` | 仅离线 synthetic/fixture 方法、schema、golden vector 与 projection boundary；真实数据、strategy identity、public C1 write、runtime、trading、publish/deploy、Git remote write 均未授权 |
| 2026-07-16 14:30 +08:00 | FU-CR164-004 | CP3 `DO-CR173-CP3-002=PASS_BY_SPLIT`：执行 CP2 已批准的 DQ-007 切换条件，CR-173 收缩为 estimator-only | `docs/design/HLD-EFFECTIVE-TRIAL-OFFLINE-METHODOLOGY.md`、`process/checkpoints/CP3-CR173-HLD-REVIEW.md` | CR-173 只关闭 methodology prerequisite；public C1 projection / C1 computable 由 future C1-contract-owner CR 承接，不是 CR-172 重开 CP2 的前置 |
| 2026-07-17 12:39 +08:00 | FU-CR164-004 | CR-173 CP8 经用户批准，以 `closed/cp8_closed/READY_WITH_RISK` 完成 offline methodology prerequisite | `process/checkpoints/CP8-CR173-DELIVERY-READINESS.md`、`process/checks/CP8-CR173-DELIVERY-READINESS.result.json` | 最高 `offline_method_ready=true`；public C1、真实/empirical R、runtime、Stage 3、admission、publish/deploy 均未授权 |
