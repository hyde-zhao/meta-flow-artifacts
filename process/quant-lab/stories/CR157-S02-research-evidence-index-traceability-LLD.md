---
story_id: "CR157-S02-research-evidence-index-traceability"
change_id: "CR-157"
title: "LLD - Research evidence index traceability"
design_type: "full-lld"
status: "ready-for-cp5-review"
created_at: "2026-07-05T13:30:00+08:00"
created_by: "host-orchestrator-inline"
owner: "meta-dev"
cp5_review_required: true
implementation_allowed: false
source_hld: "docs/design/HLD-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md"
feature_design_refs:
  - "docs/features/research-evidence-traceability/DESIGN.md"
  - "docs/features/research-evidence-traceability/TEST-PLAN.md"
  - "docs/features/research-evidence-traceability/TASKS.md"
---

# CR157-S02 LLD - Research Evidence Index Traceability

## 0. 上游设计依据

| Source | Relevant decision |
|---|---|
| HLD | `ResearchEvidenceIndex` must cover data release, run manifest, metrics, lineage, limitations and typed unavailable refs. |
| ADR-CR157-002 | Evidence index is refs-only; no full report, review, diff, TEST-MATRIX or transcript body embedding. |
| FEAT-18 | Evidence entries need id, source ref, artifact type, hash, status, owner, source CR and freshness metadata. |
| S01 LLD | Package ref set must resolve mandatory Stage 2 exit refs through evidence ids. |

## 1. Goal

Harden `ResearchEvidenceIndex` into a refs-only traceability layer that can prove where each Stage 2 package claim came from while preventing large body copying or implicit runtime validation claims.

## 2. Requirements

| ID | Requirement | Acceptance |
|---|---|---|
| S02-R1 | Represent evidence items with stable id, source ref, artifact type, owner, source CR, hash/freshness/status and blocked claim refs. | Duplicate ids, missing source refs and invalid statuses are rejected by validation. |
| S02-R2 | Remain refs-only. | Embedded body fields such as `body`, `content`, `full_report`, `full_test_matrix`, `review_text`, `diff`, `transcript` cause `BLOCKED`. |
| S02-R3 | Integrate with S01 mandatory refs and S03 handoff. | Package and handoff can reference evidence ids without copying evidence bodies. |
| S02-R4 | Preserve current `ResearchEvidenceIndex` callers. | Existing fields remain valid; new item list is optional with default empty tuple. |

## 3. 模块拆分与职责

| Object/function | Responsibility |
|---|---|
| `ResearchEvidenceItem` | Atomic evidence ref metadata. |
| `ResearchEvidenceIndex.evidence_items` | Optional tuple of evidence item refs. |
| `validate_research_evidence_index_refs_only(...)` | Deep refs-only validation for item metadata and forbidden body keys. |
| `build_stage2_research_evidence_index(...)` | Existing builder extended to accept evidence item mappings. |
| `EVIDENCE_BODY_FORBIDDEN_KEYS` | Canonical set of fields that must never appear in evidence items. |

## 4. 代码结构与文件影响范围

| File | Change |
|---|---|
| `engine/mature_multifactor_framework.py` | Add `ResearchEvidenceItem`, forbidden-key constant, refs-only validator and optional `evidence_items` field. |
| `tests/research/test_mature_multifactor_framework.py` | Add duplicate id, missing source ref, invalid hash/freshness and embedded-body negative tests after CP5 approval. |
| `process/evidence/CR157-*.index.json` | Future evidence index output path; CP5 only designs the contract. |

S02 may depend on S01 constants but must not import documentation files or read process artifacts at runtime.

## 5. 数据模型与持久化设计

`ResearchEvidenceItem` fields:

| Field | Type | Rule |
|---|---|---|
| `evidence_id` | `str` | Stable, unique within index. |
| `source_ref` | `Mapping[str, Any]` | Must include local path/id and artifact kind; no body payload. |
| `artifact_type` | `str` | One of `data_release`, `run_manifest`, `metric`, `lineage`, `evaluation`, `risk_policy`, `admission_package`, `handoff`, `guard`, `documentation`. |
| `source_cr` | `str` | Nonblank CR id or baseline id. |
| `owner` | `str` | Nonblank owner role or component. |
| `hash` | `str` | Optional for legacy refs; required for new CR157 evidence items except typed unavailable. |
| `freshness_status` | `str` | `current`, `stale`, `unknown`, or `n/a-with-reason`. |
| `status` | `str` | `PASS`, `NEEDS_REVIEW`, `BLOCKED`, `N/A_WITH_REASON`. |
| `blocked_claim_refs` | `tuple[str, ...]` | Claims blocked by this evidence item. |
| `typed_unavailable` | `tuple[TypedUnavailable, ...]` | Explicit unavailable reasons. |

No database or external catalog is introduced. Future CP6/CP7 evidence JSON remains a local process artifact and contains refs/metadata only.

## 6. API / Interface 设计

| Function | Contract |
|---|---|
| `build_research_evidence_item(mapping)` | Normalize a mapping to `ResearchEvidenceItem`, rejecting forbidden body keys. |
| `validate_research_evidence_item(item)` | Return `Stage2ValidationResult` with id/source/status/hash/freshness/body checks. |
| `validate_research_evidence_index_refs_only(index)` | Validate uniqueness, mandatory refs, item-level body exclusion and index status. |
| `build_stage2_research_evidence_index(..., evidence_items=())` | Existing builder extension; default preserves current tests. |

Callers get validation results, not exceptions, for expected data-quality failures. Programming errors can still raise normal Python exceptions.

## 7. 核心处理流程

1. Normalize existing index fields: `data_release_ref`, `run_manifest_ref`, `metric_refs`, `lineage_refs`, `limitations`, `typed_unavailable`.
2. Normalize each evidence item and reject forbidden body keys recursively.
3. Check unique `evidence_id` values.
4. Ensure mandatory S01 ref keys can resolve to either direct refs or evidence item ids.
5. Derive index status:
   - embedded body or duplicate id -> `BLOCKED`;
   - missing hash/freshness -> `NEEDS_REVIEW` or `BLOCKED` according to claim importance;
   - all mandatory refs traceable -> `PASS`.

## 8. 技术设计细节

- Forbidden body-key check is recursive over mappings/sequences because copied reports often appear nested.
- Keep an allowlist for short scalar summaries such as `summary`, `reason`, `limitation`, `remediation`; forbid only fields that carry bodies/diffs/transcripts.
- Add optional field at the end of `ResearchEvidenceIndex` for constructor compatibility.
- Evidence ids should be deterministic where practical, e.g. `EVID-CR157-{artifact_type}-{slug}`.
- Do not compute hashes by reading files in the builder. Hash values are supplied refs in fixtures/process artifacts.

## 9. 安全与性能设计

| Area | Decision |
|---|---|
| Runtime/data | No filesystem, provider, lake, credential or runtime reads in builder/validator. |
| Memory | Reject full bodies to avoid copying large evidence into state/context. |
| Security | No path dereference; `source_ref` is metadata only. |
| Performance | O(number of evidence items + nested metadata keys). |

## 10. 测试设计

| Test | Expected |
|---|---|
| complete evidence index with package refs | `PASS`; ids resolve mandatory Stage 2 refs. |
| duplicate `evidence_id` | `BLOCKED`, duplicate id named. |
| missing `source_ref` | `BLOCKED`. |
| item includes `full_report` or `diff` | `BLOCKED` by body exclusion. |
| stale metric ref | `NEEDS_REVIEW` or `BLOCKED` based on linked claim. |
| legacy index without `evidence_items` | Existing behavior remains valid when direct refs are complete. |

## 11. 实施步骤

1. Add evidence item dataclass and forbidden-key constant.
2. Add item/index refs-only validators.
3. Extend builder with optional evidence item input.
4. Wire S01 package ref-set validation to evidence ids where available.
5. Add CP6 fixture tests and evidence index output.

## 12. 风险、难点与预研建议

| Risk | Mitigation |
|---|---|
| Refs-only validation becomes too strict for old CR evidence. | Keep legacy direct refs valid; strict item validation applies when `evidence_items` are supplied. |
| Body-key list misses copied content under a new name. | Use denylist plus size/type guard for large multiline strings in item metadata. |
| Evidence id resolution creates circular coupling with package builder. | S01 owns package required keys; S02 owns evidence item validity and lookup only. |

### 12.1 Clarification Queue

| ID | Status | Question | Resolution |
|---|---|---|---|
| CQ-CR157-S02-001 | RESOLVED | Should CP5 approve copying full TEST-MATRIX/REVIEW into evidence index? | No. Only refs and short metadata are allowed. |

No OPEN or Spike question blocks CP5.

## 13. 回滚与发布策略

- Roll back by removing optional evidence item extensions and validators.
- Existing `ResearchEvidenceIndex` fields remain backward-compatible.
- No release/publish operation is part of S02.

## 14. Definition of Done

- CP5 approves the refs-only evidence index LLD.
- CP6 implementation validates duplicate, missing, stale and embedded-body cases.
- CP7 evidence index references evidence paths instead of copying evidence bodies.

## 人工确认区

CP5 reviewer should confirm the refs-only boundary and body exclusion list because downstream context packs will rely on this to stay small and auditable.
