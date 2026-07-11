---
source_cr: "CR-163"
status: "open"
created_at: "2026-07-11T18:38:00+08:00"
created_by: "meta-qa-critical"
updated_at: "2026-07-11T18:38:00+08:00"
checkpoint_source: "CP8"
cr_index_path: "process/changes/CR-INDEX.json"
release_context_ref: "process/release/RELEASE-CONTEXT-CR163-TRIAL-LINEAGE-INSTRUMENTATION.yaml"
---

# CR-163 Follow-up Tracking

## Purpose and status

This ledger records CP8 candidates only. Every CR163 item below has `lifecycle_status=candidate`; no formal CR has been created or activated. Host Orchestrator owns any future conflict precheck, CR-INDEX/ledger synchronization and conversion to a formal CR.

Accepted risks for CR163 are limited to:

- A — no-resume availability/cost loss for long-running work; lineage correctness remains fail closed.
- B — conditional validation-coverage risk if a future producer adds a retry loop without a real producer E2E ordinal fixture; current implementation is not defective.

Cooperative locking, exact-10k characterization and effective/C1 are design/scope boundaries, not accepted risks. Effective/C1 remains tracked by existing `FU-CR161-002` and is not duplicated here.

## Structured candidates

```yaml
follow_up_items:
  - id: FU-CR163-001
    title: Resume support for long-running instrumented runs
    kind: implementation-gate
    lifecycle_status: candidate
    readiness_status: n/a
    gate_status: not_started
    owner: lineage-store owner
    activation_trigger: "before approval of the first non-fixture or long-running instrumented candidate-producing run"
    required_before: "that run is authorized to rely on partial-progress recovery"
    evidence_condition: "resume design and crash/interruption fixtures, or the run CR explicitly quantifies runtime/cost and accepts complete rerun loss"
    closure_condition: "resume passes independent CP7, or the triggering run CR records approved full-rerun-loss acceptance with bounded cost/time"
    non_authorization: "does not authorize the run, real data, credentials, runtime, backfill, deployment or publish"
    source_risk: "A — availability/full-rerun loss"
    formal_cr_path: ""

  - id: FU-CR163-002
    title: Writer model hardening beyond cooperative local writers
    kind: architecture-realignment
    lifecycle_status: candidate
    readiness_status: n/a
    gate_status: not_started
    owner: lineage-store architecture owner
    activation_trigger: "before the first multi-process, multi-host or non-cooperative writer requirement is approved"
    required_before: "claiming correctness under that expanded writer model"
    evidence_condition: "new writer-model ADR, ownership/fencing tests and independent concurrency verification"
    closure_condition: "expanded model passes architecture gate and CP7; absent the trigger, existing seal/hash correctness guard and cooperative scope are sufficient"
    non_authorization: "does not authorize distributed writers, external storage, NAS, service deployment or network access"
    source_boundary: "cooperative writer model — not a current risk"
    formal_cr_path: ""

  - id: FU-CR163-003
    title: Capacity threshold and production SLO/SLA evidence
    kind: implementation-gate
    lifecycle_status: candidate
    readiness_status: n/a
    gate_status: not_started
    owner: performance and release owner
    activation_trigger: "before the first claim of support for a family larger than 10,000 trials or any production SLO/SLA"
    required_before: "publishing the capacity/SLO/SLA claim"
    evidence_condition: "approved thresholds, representative environment, repeated capacity/recovery measurements and claim-bounded report"
    closure_condition: "independent validation supports the exact claimed threshold/SLO/SLA with rollback and observation criteria"
    non_authorization: "does not authorize production load, real research, infrastructure spend, deployment or public performance claims"
    source_boundary: "10k characterization only — not a current risk"
    formal_cr_path: ""

  - id: FU-CR163-004
    title: Real producer retry-loop end-to-end ordinal fixture
    kind: implementation-gate
    lifecycle_status: candidate
    readiness_status: n/a
    gate_status: not_started
    owner: producer retry-loop change owner
    activation_trigger: "any CR that introduces a retry loop in any instrumented producer"
    required_before: "that same CR reaches CP7 or the producer remains eligible for lineage present"
    evidence_condition: "real producer same-session/same-trial attempt ordinals 1..N, replay/idempotency and raw-count invariants"
    closure_condition: "fixture passes in the same change's regression set and independent CP7 confirms present eligibility"
    non_authorization: "does not authorize real research/runtime/data access, retries in production, trading, deployment or release"
    source_risk: "B — conditional future validation coverage"
    formal_cr_path: ""
```

## Candidate summary

| ID | Status | Owner | Trigger | Required before | Non-authorization |
|---|---|---|---|---|---|
| FU-CR163-001 | candidate | lineage-store owner | first approved non-fixture/long-running run | reliance on partial-progress recovery | no run/runtime/data authorization |
| FU-CR163-002 | candidate | store architecture owner | multi-process/multi-host/non-cooperative requirement | expanded writer correctness claim | no distributed/NAS/network authorization |
| FU-CR163-003 | candidate | performance/release owner | >10k or production SLO/SLA claim | publishing the claim | no production load/deploy claim authorization |
| FU-CR163-004 | candidate | retry-loop change owner | any producer retry-loop CR | same CR CP7/present eligibility | no real producer runtime authorization |

Existing reference: `FU-CR161-002` remains the sole candidate for effective-trial method/C1/statistical correction. It is intentionally not duplicated as `FU-CR163-*`.

## Five-dimensional impact analysis

| Dimension | Impact | Current decision |
|---|---|---|
| Requirements | none now; each candidate activates only on a future explicit trigger | no product-baseline change |
| Scenarios | future resume/writer/capacity/retry scenarios become mandatory only on activation | candidate tracking only |
| Planning | future owner must open/attach a formal CR after conflict precheck | no Story/LLD work now |
| Security | candidates grant no runtime, credential, data, NAS, distributed-writer or deployment authority | deny-default retained |
| Delivery | paired repository push is separately authorized pending execution; candidates do not block current READY_WITH_RISK | retain trigger/closure evidence |

## Document handling decisions

| Object | Decision | Baseline preservation |
|---|---|---|
| Current requirements/scenarios/design | unchanged | CR163 verified baseline remains authoritative |
| Release Context and release docs | updated | prior risk wording replaced by classified A/B plus boundaries |
| CR-INDEX / CR ledger / STATE / CP8 result/checkpoint | Host-owned synchronization | not modified by this lane |

## Risk acceptance and non-authorization

| ID | Classification | Acceptance wording | Switch condition |
|---|---|---|---|
| RA-CR163-A | accepted availability/cost risk | no-resume may require full rerun but cannot turn invalid lineage into valid evidence | FU-CR163-001 trigger |
| RA-CR163-B | accepted conditional coverage risk | current producers are verified; a future retry loop is ineligible for CP7/present until E2E ordinal coverage exists | FU-CR163-004 trigger |

The user authorized the paired source+artifact repository push for the approved CP8 delivery. Status remains `authorized_pending_execution`; this ledger records no commit or push evidence and authorizes no deployment, tag, package/version publish, runtime, data, credential, NAS, provider, broker, trading, historical reconstruction or statistical computation.

## Host synchronization pending

Per task boundary, this lane did not modify `CR-INDEX.json`, `CR-LEDGER.ndjson`, `STATE.current.json`, the CR document, or CP8 checkpoint/result. Host Orchestrator must synchronize the four candidate entries and record/validate `meta-flow check cr-tracking` after its authorized state update.
