---
change_id: "CR-163"
release_decision: "READY_WITH_RISK"
release_artifact_profile: "compact"
version_decision: "internal-release-candidate"
created_at: "2026-07-11T18:15:00+08:00"
updated_at: "2026-07-11T18:38:00+08:00"
---

# CR163 Trial Lineage Instrumentation — Release Notes

## Readiness

`READY_WITH_RISK` for human CP8 confirmation. No public version number or true release was created; treat this as an internal release candidate tied to CR163 because the repository has no authorized version/tag/publish action in scope.

## User-visible behavior

- Future instrumented candidate-producing runs can declare an experiment family before search, append auditable trial/attempt/selection facts and seal a deterministic immutable manifest.
- Raw trial count uses stable trial identity: retries do not inflate the count; different seeds remain distinct; failed/cancelled/excluded/never-started declared trials remain auditable.
- Sealed evidence is immutable. Corrections create a superseding version that retains prior ref/hash/reason.
- CR151, CR154 and the admission package consume one validation-bound projection and fail closed on absent, malformed, mismatched or tampered evidence.
- Existing uninstrumented paths remain unavailable/blocked; historical CR155 stays blocked and not paper-candidate.

## Compatibility and breaking behavior

- New contracts and optional lineage inputs are additive, but callers that omit lineage cannot receive positive admission truth; this is an intentional fail-closed behavior change.
- No installation path, environment variable, database, external interface or persisted production-data migration is introduced.
- Local JSON/JSONL lineage artifacts created by future instrumented runs are new evidence, not a migration of historical runs.

## Accepted risks

- A — no-resume availability loss: malformed/interrupted evidence cannot resume in place; a long-running run may require a full rerun. Before the first approved non-fixture/long-running instrumented run, implement resume or have that run's CR quantify and accept the loss.
- B — conditional future coverage: if a producer introduces a retry loop, that same CR must add the real producer same-session/same-trial ordinal E2E fixture before CP7/present eligibility.

## Design and scope boundaries — not risks

- Cooperative locking is the supported cooperating-writer model; seal/hash validation remains the correctness guard. Writer hardening activates only before a multi-process/multi-host/non-cooperative requirement.
- Exact-10k results are characterization only. Capacity/SLA work activates before any >10k support claim or production SLO/SLA.
- Effective trial/C1 remains intentionally unavailable under the existing `FU-CR161-002`; no duplicate CR163 follow-up is created.
- Both repositories are dirty. The user authorized paired source+artifact push; status is `authorized_pending_execution`, not executed.

## Evidence and rollback

See `process/release/RELEASE-CONTEXT-CR163-TRIAL-LINEAGE-INSTRUMENTATION.yaml`, the five Story CP7 evidence indexes, and `docs/release/ROLLBACK-CR163-TRIAL-LINEAGE-INSTRUMENTATION.md`.

This readiness note records authorization for the paired source+artifact repository push only; execution evidence is pending. It does not authorize deployment, package/version publish, live/runtime use, data access or unrelated Git remote writes.
