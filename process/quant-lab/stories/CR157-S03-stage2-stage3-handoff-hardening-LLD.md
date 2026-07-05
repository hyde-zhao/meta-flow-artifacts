---
story_id: "CR157-S03-stage2-stage3-handoff-hardening"
change_id: "CR-157"
title: "LLD - Stage 2 to Stage 3 handoff hardening"
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
  - "docs/features/stage-handoff-guardrails/DESIGN.md"
  - "docs/features/stage-handoff-guardrails/TEST-PLAN.md"
  - "docs/features/stage-handoff-guardrails/TASKS.md"
---

# CR157-S03 LLD - Stage 2 to Stage 3 Handoff Hardening

## 0. 上游设计依据

| Source | Relevant decision |
|---|---|
| HLD | Stage 3 handoff must include 12 required inputs, 13 required evidence refs, blocked claims and no-runtime boundary. |
| ADR-CR157-003 | Handoff fails closed; absent package/index/status cannot be treated as PASS. |
| FEAT-19 | Stage handoff guardrails own package ref, evidence index ref, readiness status and blocked claim propagation. |
| S01/S02 LLD | Handoff consumes package ref set and evidence index results; it does not duplicate their validation bodies. |

## 1. Goal

Make the Stage 2 to Stage 3 handoff explicit enough that an incomplete, stale, blocked or runtime-contaminated mature strategy package cannot be mistaken for Stage 3 readiness.

## 2. Requirements

| ID | Requirement | Acceptance |
|---|---|---|
| S03-R1 | Handoff carries package ref, evidence index ref, readiness status, blocked claims and no-runtime counters. | Missing package/index/status validates `BLOCKED`; no default PASS. |
| S03-R2 | Required Stage 3 inputs/evidence stay complete and visible. | Handoff lists exactly 12 `STAGE2_DATA_REQUIREMENTS` and 13 `STAGE3_REQUIRED_EVIDENCE` unless explicitly versioned later. |
| S03-R3 | Blocked package reasons propagate to Stage 3 candidate intake. | Handoff validation returns blocked reasons with source refs to package/evidence. |
| S03-R4 | Stage 3 consumer contract remains static-only. | No simulation/live/runtime/registry/publish claims are created by handoff. |

## 3. 模块拆分与职责

| Object/function | Responsibility |
|---|---|
| `Stage3ResearchMachineHandoff` | Existing handoff object; extended with optional package/evidence readiness metadata. |
| `Stage2Stage3HandoffReadiness` | Optional nested readiness view with package/evidence status and blocked claims. |
| `build_stage3_research_machine_handoff(...)` | Add optional package/evidence refs while preserving current inputs. |
| `validate_stage3_research_machine_handoff(...)` | Fail-closed checks for required inputs/evidence, readiness status, blocked claims and forbidden counters. |

## 4. 代码结构与文件影响范围

| File | Change |
|---|---|
| `engine/mature_multifactor_framework.py` | Extend handoff dataclass at end with optional `package_ref`, `research_evidence_index_ref`, `readiness_status`, `blocked_claim_refs`, `permission_counters`; harden validator. |
| `tests/research/test_mature_multifactor_framework.py` | Add handoff complete/missing/unknown-status/blocked-counter tests after CP5 approval. |

No Stage 3 runner, external framework adapter, registry write or runtime path is introduced.

## 5. 数据模型与持久化设计

Optional handoff fields:

| Field | Type | Rule |
|---|---|---|
| `package_ref` | `Mapping[str, Any]` | Required for CR157 mature package handoff; must name package id/schema/status. |
| `research_evidence_index_ref` | `Mapping[str, Any]` | Required for CR157; must resolve to S02 index id/status. |
| `readiness_status` | `str` | One of `PASS`, `NEEDS_REVIEW`, `BLOCKED`. Missing/unknown is `BLOCKED`. |
| `blocked_claim_refs` | `tuple[str, ...]` | Includes package/evidence blocked claims and forbidden Stage 3 claims. |
| `permission_counters` | `Mapping[str, int]` | Same forbidden counters as S04. |

Persistence remains local object/fixture only. CP6 may emit handoff evidence refs but not a published runtime package.

## 6. API / Interface 设计

| Function | Input | Output | Failure |
|---|---|---|---|
| `build_stage3_research_machine_handoff(...)` | Existing support/candidate refs plus optional package/evidence/readiness/counters. | `Stage3ResearchMachineHandoff`. | Missing optional CR157 refs yields blocked readiness when validator runs. |
| `validate_stage3_research_machine_handoff(handoff)` | Dataclass or mapping. | `Stage2ValidationResult`. | Missing required input/evidence/package/index/status/counters are blocked reasons. |

Stage 3 consumers must read `readiness_status` and validation result. They must not infer readiness from object existence.

## 7. 核心处理流程

1. Build handoff from mature support/candidate/package/evidence refs.
2. Include canonical `STAGE2_DATA_REQUIREMENTS` and `STAGE3_REQUIRED_EVIDENCE`.
3. Pull S01/S02 status into handoff readiness:
   - any package/evidence `BLOCKED` -> handoff `BLOCKED`;
   - any `NEEDS_REVIEW` -> handoff `NEEDS_REVIEW`;
   - all complete -> handoff `PASS`.
4. Validate no forbidden Stage 3 claims (`qmt_ready`, `simulation_ready`, `live_ready`, `small_live_ready`, `runtime_authorized`) are true.
5. Validate all permission counters are zero.

## 8. 技术设计细节

- Preserve current `Stage3ResearchMachineHandoff` constructor by adding new fields with defaults at the end.
- Keep required input/evidence constants as the single source of truth.
- Use `blocked_until` to describe missing evidence when handoff is blocked.
- Handoff may reference evidence item ids; it must not inline full evidence items or reports.
- Do not add event/ML adapter-specific fields in this Story.

## 9. 安全与性能设计

| Area | Decision |
|---|---|
| Authorization | Handoff never grants runtime, simulation, live, broker, QMT or publish authorization. |
| Fail-closed | Missing or unknown readiness is blocked. |
| Performance | Pure mapping/dataclass validation; O(required refs). |
| Security | No path dereference, no credential read, no network. |

## 10. 测试设计

| Test | Expected |
|---|---|
| complete package and evidence handoff | `PASS`; required inputs=12 and evidence=13. |
| missing `package_ref` | `BLOCKED`. |
| missing `research_evidence_index_ref` | `BLOCKED`. |
| unknown `readiness_status` | `BLOCKED`. |
| package `NEEDS_REVIEW` | handoff `NEEDS_REVIEW`; not PASS. |
| forbidden Stage 3 claim true | `BLOCKED`. |
| nonzero forbidden counter | `BLOCKED`. |

## 11. 实施步骤

1. Extend handoff dataclass with optional CR157 refs/status fields.
2. Harden builder to accept package/evidence readiness metadata.
3. Harden validator for required package/index/status and counter checks.
4. Add tests for fail-closed negative paths.
5. Update CP6 evidence index with handoff validation refs.

## 12. 风险、难点与预研建议

| Risk | Mitigation |
|---|---|
| Existing Stage 3 tests rely on current constructor. | Add optional fields at end with defaults and keep existing semantics for non-CR157 callers. |
| Handoff duplicates package/evidence validation logic. | Only consume S01/S02 validation statuses and refs; do not duplicate body checks. |
| Stage 3 readiness wording overclaims runtime. | Keep wording `research machine handoff`, not paper/live/runtime readiness. |

### 12.1 Clarification Queue

| ID | Status | Question | Resolution |
|---|---|---|---|
| CQ-CR157-S03-001 | RESOLVED | Can object existence imply Stage 3 readiness? | No. `readiness_status` and validation result are mandatory; unknown blocks. |

No OPEN or Spike question blocks CP5.

## 13. 回滚与发布策略

- Roll back optional handoff fields and validator additions.
- Existing handoff object remains usable by earlier tests.
- No release/publish operation is authorized.

## 14. Definition of Done

- CP5 approves handoff fail-closed design.
- CP6 implementation covers package/evidence/status/counter negative cases.
- CP7 confirms Stage 3 handoff evidence does not claim runtime readiness.

## 人工确认区

CP5 reviewer should confirm Stage 3 consumer semantics: absent fields and unknown statuses are always blocked, never interpreted as PASS.
