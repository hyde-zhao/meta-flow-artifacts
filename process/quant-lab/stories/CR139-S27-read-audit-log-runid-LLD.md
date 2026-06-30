---
story_id: "CR139-S27"
title: "L3 read audit log + run-id propagation"
story_slug: "read-audit-log-runid"
lld_version: "1.0"
tier: "M"
status: "ready-for-review"
confirmed: false
created_by: "host-orchestrator"
created_at: "2026-06-30T11:45:00+08:00"
feature_design_refs:
  - "docs/design/FEATURE-DESIGN-MATRIX.md v1.13 REQ-233"
  - "process/docs/design/HLD-STRATEGY-DATA-FOUNDATION.md v0.2"
  - "process/docs/design/ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md v0.2"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "d1 纯新建对象"
  rationale: "L3 is a new read-audit contract that crosses FEAT-02 read side and FEAT-11 RunEvidenceIndex consumption."
open_items: 0
---

# LLD: CR139-S27 — Read Audit Log + Run ID Propagation

> This file is the CR139 Backlog-A3 CP5 full-lld design evidence for S27.
> It does not authorize implementation, real lake writes, active catalog/manifest writes, pointer advance, NAS/provider/runtime/trading/credential/Git operations.

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-30 | host-orchestrator | 首版 S27 full-lld；定义 read audit record、reader hook、RunEvidenceIndex run-id handoff and fixture/static tests. |

## 0. Upstream Basis

| Source | Path / ID | S27 usage |
|---|---|---|
| Story card | `process/stories/CR139-S27-read-audit-log-runid.md` | REQ-233, full-lld, output files, cross-boundary owner/consumer. |
| A0 gap analysis | `process/checks/CR139-BACKLOG-A0-GAP-ANALYSIS-2026-06-30.md` | Confirms S27 still needs full Story execution; S22/Gate C/D lineage is producer evidence only. |
| S22 implementation | `process/stories/CR139-S22-runid-lineage-penetration-IMPLEMENTATION.md` | Existing `RunEvidenceIndex` lineage fields are consumer-side target. |
| Reader code | `market_data/readers.py` | Existing `ReaderResult`, catalog entry, issues, and `source_run_id` output columns. |
| HLD/ADR/MATRIX | companion docs | FEAT-02 read side owns read audit; FEAT-11 consumes run evidence and must not own reader hook. |

## 1. Goal

Create a read-side audit contract so every successful or blocked reader call can produce a deterministic audit record containing dataset, status, issue codes, catalog/published run identifiers, and row lineage run identifiers. The audit record must be consumable by `RunEvidenceIndex` without reversing dependency direction.

## 2. Requirements

| ID | Requirement | Design response |
|---|---|---|
| REQ-233-A | Reader read completion emits read audit evidence. | Add `market_data/read_audit.py` with pure audit record construction and JSON-safe serialization. |
| REQ-233-B | Audit log and `RunEvidenceIndex` use the same run-id lineage. | Extract `catalog_entry.latest_manifest_run_id`, frame `source_run_id` values, and optional caller `strategy_run_id` into an audit payload that FEAT-11 can read. |
| REQ-233-C | Blocked reads are auditable. | Audit record includes `status`, `issue_codes`, and remediation action when `ReaderResult.status != available`. |
| Boundary | No default persistent audit write in CP6. | CP6 fixture/static scope uses in-memory audit sink/list; real audit persistence remains out of scope. |

## 3. Modules And Responsibilities

| Module | Responsibility | Ownership |
|---|---|---|
| `market_data/read_audit.py` | New dataclass/serializer and helper to build read audit records from `ReaderResult`. | S27 exclusive. |
| `market_data/readers.py` | Optional hook point for audit record construction after `read_dataset`/`read_panel_as_of` result. | FEAT-02 read side, merge-order after S25 and before S28. |
| `trading/strategy_runner/evidence_index.py` | Consumer only; should not import reader code in S27 unless a narrow adapter is required. | FEAT-11 consumer, no reverse write ownership. |
| `tests/test_cr139_read_audit_runid.py` | Fixture/static tests for available and blocked reads. | S27 exclusive. |

## 4. File Impact

| Action | File | Scope |
|---|---|---|
| Create | `market_data/read_audit.py` | `ReadAuditRecord`, `build_read_audit_record()`, JSON-safe `to_dict()`. |
| Modify | `market_data/readers.py` | Optional hook or helper call only; no catalog/manifest/pointer writes. |
| Create | `tests/test_cr139_read_audit_runid.py` | Fixture/static tests. |
| No change by default | `trading/**` | Consumer validation can use dict payload; no trading runtime or QMT. |

## 5. Data Model

| Field | Type | Required | Notes |
|---|---|---|---|
| `audit_id` | string | yes | Deterministic hash over dataset, status, requested as_of, catalog run, source run IDs, and issue codes. |
| `dataset` | string | yes | Reader dataset. |
| `reader_name` | string | yes | `read_dataset`, `read_panel_as_of`, or caller-supplied name. |
| `status` | string | yes | Mirrors `ReaderResult.status`. |
| `issue_codes` | list[string] | yes | From `ReaderResult.issues[*].code`. |
| `catalog_run_id` | string/null | no | `ReaderResult.catalog_entry.latest_manifest_run_id` if present. |
| `source_run_ids` | list[string] | yes | Unique `source_run_id` values in returned frame. |
| `requested_as_of` | string/null | no | Caller decision time when available. |
| `row_count` | integer | yes | 0 when no frame. |
| `strategy_run_id` | string/null | no | Optional FEAT-11 caller run ID. |
| `remediation_action` | string/null | no | From remediation spec for blocked reads. |

## 6. Interface Design

| Interface | Input | Output | Notes |
|---|---|---|---|
| `build_read_audit_record(dataset, result, reader_name, requested_as_of=None, strategy_run_id=None)` | `ReaderResult` | `ReadAuditRecord` | Pure; no IO. |
| `ReadAuditRecord.to_dict()` | N/A | JSON-safe dict | Stable for evidence index consumption. |
| Optional reader hook | `audit_sink` callable/list | Side-channel audit record | CP6 may add only if tests prove the hook is needed. Default remains no persistent write. |

## 7. Core Flow

1. Reader returns `ReaderResult`.
2. S27 helper builds audit record from result metadata and frame lineage.
3. Caller may pass audit dict to `RunEvidenceIndex` or evidence summary.
4. Blocked reads still produce audit evidence with issue codes and remediation action.

```mermaid
flowchart LR
  R[ReaderResult] --> A[build_read_audit_record]
  A --> D[ReadAuditRecord dict]
  D --> E[RunEvidenceIndex data_lineage consumer]
```

## 8. Technical Rules

| Rule | Detail |
|---|---|
| Deterministic ID | Use sorted `source_run_ids` and issue codes in hash input. |
| No persistence by default | `read_audit.py` constructs records; it does not write files or append manifests. |
| Consumer boundary | FEAT-11 consumes dict fields; S27 does not start trading runtime. |
| Blocked read support | A `required_missing`, `unavailable`, `quality_failed`, or `pit_failed` result is audit-valid. |
| Privacy | No credential/env values in audit record. |

## 9. Security And Performance

| Dimension | Design measure |
|---|---|
| IO safety | Pure helper and in-memory fixtures only. |
| Runtime safety | No QMT/MiniQMT/gateway/trading runtime. |
| Performance | Audit extraction is O(row_count) over `source_run_id` column only. |
| Compatibility | `ReaderResult` fields remain unchanged unless an optional hook is added with default `None`. |

## 10. Test Design

| Test ID | Scenario | Expected |
|---|---|---|
| T-S27-AUDIT-01 | Available read result with catalog current run and source rows. | Audit contains dataset, status `available`, catalog run, source run IDs, row count. |
| T-S27-AUDIT-02 | Blocked read result with issue/remediation. | Audit contains status and issue codes, row_count 0, remediation action. |
| T-S27-RUNID-01 | Audit dict is consumed by `RunEvidenceIndex` lineage fields or equivalent fixture adapter. | Same run-id appears in audit and evidence payload. |
| T-S27-BOUNDARY-01 | Inspect helper/hook for forbidden writes. | No catalog/manifest/pointer/lake/runtime write. |

## 11. Implementation Steps

| TASK-ID | Action | Target |
|---|---|---|
| TASK-S27-01 | Create pure audit model/helper. | `market_data/read_audit.py` |
| TASK-S27-02 | Add optional reader hook only if needed for acceptance. | `market_data/readers.py` |
| TASK-S27-03 | Add fixture/static tests. | `tests/test_cr139_read_audit_runid.py` |
| TASK-S27-04 | Write CP6 return/evidence/check. | `process/*` |

## 12. Risks And Clarifications

| ID | Status | Decision | Reason |
|---|---|---|---|
| LCQ-S27-01 | resolved | No persistent audit log write in this batch. | Real audit persistence would require a storage contract and write authorization. |
| LCQ-S27-02 | resolved | FEAT-11 remains consumer only. | Prevents reverse dependency from strategy runtime into data reader. |

No blocking open item.

## 13. Rollback And Release

Rollback is code-only: remove `read_audit.py`, optional reader hook, tests, and process evidence. No data rollback is needed because no production data write is authorized.

## 14. Definition Of Done

- [ ] `market_data/read_audit.py` provides deterministic read audit records.
- [ ] S27 tests prove available and blocked read audit records.
- [ ] Run-id lineage is visible to `RunEvidenceIndex`-style payloads.
- [ ] No real lake/catalog/manifest/pointer/NAS/provider/runtime/Git operation occurs.

## CP5 Checklist Summary

| # | Check | Status | Evidence |
|---|---|---|---|
| 1 | LLD covers AC | PASS | Sections 1, 2, 10 |
| 2 | Cross-boundary owner/consumer clear | PASS | Sections 3, 6, 12 |
| 3 | File impact clear | PASS | Section 4 |
| 4 | Tests calculable | PASS | Section 10 |
| 5 | Forbidden operations excluded | PASS | Sections 8, 9, 14 |

## Human Confirmation Area

- CP5 conclusion: `pending`
- Reviewer:
- Reviewed at:
- Notes:
