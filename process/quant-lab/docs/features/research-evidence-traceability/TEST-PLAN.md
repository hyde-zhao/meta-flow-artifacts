---
feature_id: "FEAT-18"
change_id: "CR-157"
status: "cp4-story-planning"
version: "0.1"
created_at: "2026-07-05T13:18:00+08:00"
owner: "host-orchestrator"
implementation_allowed: false
---

# Research Evidence Traceability Test Plan

## Purpose

Define validation expectations for refs-only Stage 2 evidence index behavior.

## Test Levels

| Level | Scope | Required Coverage |
|---|---|---|
| Schema tests | Evidence id, source ref, artifact type, owner, source CR, status, hash/freshness fields. | Valid index, duplicate id, missing source ref, unsupported artifact type. |
| Traceability tests | Package entry to evidence entry to source artifact. | All mandatory package refs resolve to evidence ids. |
| Slimming tests | Context budget and body exclusion. | Reject embedded full report body, full transcript, full diff or long review content. |
| Negative tests | Corruption and stale refs. | Invalid hash, stale generated-at, unknown owner, missing blocked claim reason. |

## Fixture Matrix

| Fixture Group | Story Owner | Required Cases |
|---|---|---|
| Valid refs-only index | S02 | Minimal complete index with all mandatory Stage 2 exit ref families. |
| Duplicate / missing refs | S02 | Duplicate evidence id, missing source path, missing owner, missing source CR. |
| Embedded body rejection | S02 / S05 | Report body, TEST-MATRIX copy, REVIEW copy, transcript copy. |
| Handoff consumption | S02 / S03 | Handoff accepts complete index and blocks incomplete index. |

## Forbidden Validation

No fixture may access real lake/NAS/provider/QMT/runtime/trading/broker/credential/store/catalog/registry/publish resources.
