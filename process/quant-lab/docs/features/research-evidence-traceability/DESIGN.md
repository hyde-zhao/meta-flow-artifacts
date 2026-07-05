---
feature_id: "FEAT-18"
feature_name: "Research Evidence Traceability"
change_id: "CR-157"
status: "cp4-story-planning"
version: "0.1"
created_at: "2026-07-05T13:18:00+08:00"
owner: "host-orchestrator"
source_hld: "docs/design/HLD-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md"
implementation_allowed: false
authorization_boundary: "refs-only/static evidence planning; no full report embedding and no runtime or real data access"
---

# Research Evidence Traceability Feature Design

## Purpose

Define a refs-only evidence index for Stage 2 research maturity. The feature provides stable evidence ids, source refs, hash/freshness metadata, owner feature/story, status and blocked-claim linkage for admission packages and handoff checks.

## Revision Record

| Version | Date | Author | Change |
|---|---|---|---|
| 0.1 | 2026-07-05 | host-orchestrator | Initial CR157 CP4 feature design for refs-only evidence traceability. |

## Feature Boundary

| Area | In Scope | Out of Scope |
|---|---|---|
| Evidence index | Evidence id, source ref, artifact type, hash, status, owner, source CR, freshness and blocked claim refs. | Embedding complete reports, long test logs, review bodies or transcripts. |
| Traceability | Admission package entry to evidence entry to source artifact path. | Rewriting existing source artifacts or normalizing historical reports. |
| Freshness | Hash/fingerprint and generated-at metadata sufficient for fixture validation. | Real file integrity attestation outside local workspace. |
| Audit slimming | Minimal refs for CP result / evidence index / handoff consumption. | Passing complete HLD, TEST-MATRIX, REVIEW, diff or transcript by default. |

## Integration Contract

| Caller / Neighbor | Direction | Contract |
|---|---|---|
| FEAT-17 mature package | Consumes evidence ids | Package entries refer to evidence ids and source refs. |
| FEAT-19 handoff guardrails | Consumes evidence index summary | Handoff checks require index completeness and blocked claim refs. |
| CP6 / CP7 evidence protocol | Compatible target | CR157 evidence output must be consumable as `process/evidence/*.index.json` later. |

## Failure Paths

| Failure | Required Behavior |
|---|---|
| Evidence id duplicate | Reject index as invalid. |
| Source ref missing | Mark evidence item `BLOCKED`; package cannot claim ready. |
| Full body embedded | Treat as contract violation; CP7 should fail privacy/context-budget review. |
| Unknown owner | Mark `NEEDS_REVIEW` or `BLOCKED` per CP5 policy, never silent PASS. |

## Not Authorized

No real data access, report regeneration, provider fetch, source mutation, store/catalog/registry write, publish or runtime execution is authorized by CP4.
