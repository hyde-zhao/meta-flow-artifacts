---
story_id: "CR157-S02-research-evidence-index-traceability"
change_id: "CR-157"
title: "Research evidence index traceability"
status: "verified-with-risk"
owner: "meta-dev"
feature_design_refs:
  - "docs/features/research-evidence-traceability/DESIGN.md"
  - "docs/features/research-evidence-traceability/TEST-PLAN.md"
  - "docs/features/research-evidence-traceability/TASKS.md"
source_hld: "docs/design/HLD-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md"
lld_policy:
  required: true
  required_level: "full-lld"
  status: "approved"
  approved_at: "2026-07-05T13:31:00+08:00"
  evidence_ref: "process/stories/CR157-S02-research-evidence-index-traceability-LLD.md"
depends_on:
  - "CR157-S01-mature-admission-package-builder-contract"
implementation_allowed: true
authorization_boundary: "CP5 approved local source/test implementation only; refs-only evidence; no full report embedding, runtime, real data, publish or trading"
---

# CR157-S02 Research Evidence Index Traceability

## Goal

Define a refs-only research evidence index that links Stage 2 package entries to source evidence without copying full report bodies, reviews, diffs or transcripts.

## Scope

- Evidence id, source ref, artifact type, owner, source CR, hash/freshness, status and blocked claim linkage.
- Trace from mature admission package entries to evidence index entries.
- Negative cases for duplicate ids, missing refs, invalid hashes and embedded-body violations.

## Acceptance Criteria

- All mandatory Stage 2 exit refs can resolve through evidence ids.
- Evidence index rejects duplicate ids and missing source refs.
- Embedded full report body, full TEST-MATRIX, full REVIEW, full diff or transcript content is disallowed.
- Index remains local/static-only and does not access real data or runtime.

## File Ownership

| File | Intent |
|---|---|
| `engine/mature_multifactor_framework.py` | Future evidence index model or helper location, subject to CP5 approval. |
| `tests/research/test_mature_multifactor_framework.py` | Future evidence traceability fixture tests, subject to CP5 approval. |
| `process/evidence/CR157-*.index.json` | Future CP6/CP7 evidence index output path, subject to implementation gates. |

## Design Evidence Required For CP5

Full LLD covering schema, lookup contract, body exclusion policy, freshness/hash behavior, package integration and handoff integration.

## CP5 Design Evidence

- Evidence type: `full-lld`
- Evidence ref: `process/stories/CR157-S02-research-evidence-index-traceability-LLD.md`
- Review status: `ready-for-cp5-review`

## CP5 Approval

- Status: `approved`
- Approved at: `2026-07-05T13:31:00+08:00`
- Design evidence: `process/stories/CR157-S02-research-evidence-index-traceability-LLD.md`
- Implementation boundary: local source/test/docs only; no real lake/NAS/provider/credential/QMT/runtime/simulation/live/trading/broker/catalog/store/registry/publish/external framework/Git remote operation.
