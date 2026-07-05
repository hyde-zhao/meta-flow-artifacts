---
story_id: "CR157-S01-mature-admission-package-builder-contract"
change_id: "CR-157"
title: "LLD - Mature admission package builder contract"
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
  - "docs/features/mature-admission-package/DESIGN.md"
  - "docs/features/mature-admission-package/TEST-PLAN.md"
  - "docs/features/mature-admission-package/TASKS.md"
---

# CR157-S01 LLD - Mature Admission Package Builder Contract

## 0. 上游设计依据

| Source | Relevant decision |
|---|---|
| `docs/design/HLD-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md` | Stage 2 exits through a fixture/static-only mature admission package, evidence index and Stage 3 handoff; no lake/provider/runtime actions. |
| `docs/design/ARCHITECTURE-DECISION-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md` | ADR-CR157-001 contract-first package hardening; ADR-CR157-005 no-runtime guard is first-class acceptance. |
| `docs/features/mature-admission-package/DESIGN.md` | FEAT-17 owns mature admission package, mandatory refs, status model and fail-closed behavior. |
| `process/DEVELOPMENT-PLAN-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.yaml` | CP5 focus items require explicit Stage 2 exit refs and shared-file serial merge order. |
| `engine/mature_multifactor_framework.py` | Existing `MatureAdmissionSupport`, `StrategyCandidate`, `Stage2MatureFrameworkBundle` and CR030/CR039 builder are the compatibility base. |

## 1. Goal

Add a machine-verifiable Stage 2 mature admission package contract that can assemble and validate the required exit references without touching real data, providers, credentials, QMT, simulation, live trading, catalog/store/registry writes or publish paths.

## 2. Requirements

| ID | Requirement | Acceptance |
|---|---|---|
| S01-R1 | Represent mandatory Stage 2 exit refs: FactorSpec, FactorRunSpec, factor panel, label window, evaluation, portfolio/risk, admission support and evidence index. | A complete fixture bundle validates `PASS`; any missing mandatory ref validates `BLOCKED` unless represented as typed `n/a-with-reason`. |
| S01-R2 | Preserve existing CR150/CR151/CR154 contracts. | Existing public dataclasses and builders keep backward-compatible defaults; existing Stage 2 tests continue to pass. |
| S01-R3 | Fail closed on invalid factor spec, missing CR030/CR039 refs, stale/invalid evidence refs or nonzero forbidden operation counters. | Validation result contains explicit blocked reasons and never defaults unknown status to PASS. |
| S01-R4 | Keep the package static-only. | Builder accepts mappings/dataclasses/fixture refs only; it performs zero filesystem, network, credential, provider, lake, QMT or runtime reads. |

## 3. 模块拆分与职责

| Module/object | Responsibility | Owner Story |
|---|---|---|
| `STAGE2_MATURE_PACKAGE_REQUIRED_REFS` | Canonical tuple of required Stage 2 exit ref keys. | S01 |
| `Stage2MaturePackageRefSet` | Immutable reference set for Stage 2 exit completeness. | S01 |
| `Stage2MatureFrameworkBundle.stage2_exit_refs` | Backward-compatible optional field linking bundle to the explicit ref set. | S01 |
| `build_stage2_mature_package_ref_set(...)` | Deterministic builder from existing factor/evidence/risk/admission inputs. | S01 |
| `validate_stage2_mature_package_ref_set(...)` | Completeness/status/hash/counter validation for package refs. | S01 |
| Existing CR030/CR039 builder | Compatibility input path for `MatureAdmissionSupport`. | S01 |

## 4. 代码结构与文件影响范围

| File | Change |
|---|---|
| `engine/mature_multifactor_framework.py` | Add required-ref constants, `Stage2MaturePackageRefSet`, builder and validator; add optional `stage2_exit_refs` field to `Stage2MatureFrameworkBundle` at the end of the dataclass to avoid positional breakage. |
| `tests/research/test_mature_multifactor_framework.py` | Add S01 fixture tests after CP5 approval. If the repo keeps Stage 2 tests in `test_mature_multifactor_framework_stage2.py`, place tests there and record the deviation in CP6 evidence. |
| `process/evidence/CR157-*.index.json` | Future evidence refs only; no CP5 implementation writes. |

No other engine module should own this package contract. S02/S03/S04 may import the constants/validator after S01 is merged.

## 5. 数据模型与持久化设计

```python
STAGE2_MATURE_PACKAGE_REQUIRED_REFS = (
    "factor_spec_refs",
    "factor_run_spec_refs",
    "factor_panel_ref",
    "label_window_ref",
    "evaluation_report_refs",
    "portfolio_risk_policy_ref",
    "mature_admission_support_ref",
    "research_evidence_index_ref",
)
```

`Stage2MaturePackageRefSet` fields:

| Field | Type | Rule |
|---|---|---|
| `package_id` | `str` | Stable id; nonblank. |
| `status` | `str` | One of `PASS`, `FAIL`, `NEEDS_REVIEW`, `BLOCKED`. Unknown status is `BLOCKED`. |
| `factor_spec_refs` | `tuple[Mapping[str, Any], ...]` | At least 1 valid FactorSpec ref. |
| `factor_run_spec_refs` | `tuple[Mapping[str, Any], ...]` | At least 1 valid FactorRunSpec ref or typed unavailable with reason. |
| `factor_panel_ref` / `label_window_ref` | `Mapping[str, Any]` | Must include `id`, `schema_version`, `hash` or `typed_unavailable`. |
| `evaluation_report_refs` | `tuple[Mapping[str, Any], ...]` | Must include statistical/reliability refs from CR151/CR154. |
| `portfolio_risk_policy_ref` | `Mapping[str, Any]` | Must resolve to `PortfolioRiskPolicy` ref. |
| `mature_admission_support_ref` | `Mapping[str, Any]` | Must resolve to `MatureAdmissionSupport`. |
| `research_evidence_index_ref` | `Mapping[str, Any]` | Consumed by S02 and S03. |
| `permission_counters` | `Mapping[str, int]` | All `STAGE2_FORBIDDEN_COUNTERS` must be zero. |
| `blocked_reasons` | `tuple[Stage2BlockedReason, ...]` | Required when status is `BLOCKED`/`FAIL`. |

No new persistence is introduced in S01. Future CP6 output remains dataclass/mapping objects and test fixtures.

## 6. API / Interface 设计

| Function | Input | Output | Failure |
|---|---|---|---|
| `build_stage2_mature_package_ref_set(...)` | Existing `FactorSpec` refs, FactorRunSpec refs, factor panel ref, label window ref, evaluation refs, `PortfolioRiskPolicy`, `MatureAdmissionSupport`, `ResearchEvidenceIndex`, counters. | `Stage2MaturePackageRefSet`. | Returns `status=BLOCKED` with reasons; does not raise for validation failures. |
| `validate_stage2_mature_package_ref_set(ref_set)` | Dataclass or mapping. | `Stage2ValidationResult`. | Nonzero counters, missing refs, invalid status or missing blocked reasons are `BLOCKED`. |
| `build_mature_admission_support_from_cr030_cr039_outputs(...)` | Existing CR030/CR039 outputs. | Existing `Stage2MatureFrameworkBundle`, with optional `stage2_exit_refs` when inputs are supplied. | Existing behavior remains compatible; missing optional exit refs make only the new ref-set validator block. |

Call direction: CP6 implementation tests call builder/validator directly. S02 evidence index and S03 handoff consume only refs and validation results, not implementation internals.

## 7. 核心处理流程

1. Normalize factor specs, factor run specs, panel, label, evaluation, portfolio/risk, admission support and evidence index into refs.
2. Normalize permission counters through existing `STAGE2_FORBIDDEN_COUNTERS`.
3. Validate each required ref: nonblank id, schema/version where applicable, hash/freshness where applicable, or explicit typed unavailable reason.
4. Derive status:
   - any nonzero forbidden counter -> `BLOCKED`;
   - any missing mandatory ref without typed unavailable -> `BLOCKED`;
   - stale/invalid hash -> `NEEDS_REVIEW` or `BLOCKED` according to severity;
   - all required refs present -> `PASS`.
5. Return immutable ref set and attach it to `Stage2MatureFrameworkBundle` when building the mature bundle path.

## 8. 技术设计细节

- Add new dataclass fields only at the end of existing dataclasses to avoid breaking positional construction in current tests.
- Reuse `Stage2BlockedReason`, `TypedUnavailable`, `_ref`, `_validation`, `_normalize_permission_counters`, `_is_real_ref` and `_dedupe_reasons`.
- Do not introduce JSON serialization helpers in S01; use existing dataclass/mapping patterns.
- Do not introduce adapters for event or ML strategy types; those remain `DF-CR157-001` and `DF-CR157-002`.
- `n/a-with-reason` is acceptable only when represented as `TypedUnavailable(reason=...)` with an explicit source and remediation.

## 9. 安全与性能设计

| Area | Decision |
|---|---|
| Runtime | No imports or calls to provider/QMT/credential/lake/runtime packages. |
| Filesystem | No fixture read/write inside builder. Tests may load static fixtures only after CP5 approval. |
| Performance | O(number of refs); no dataframe operations. |
| Security | Any nonzero forbidden counter blocks the package. |

## 10. 测试设计

| Test | Expected |
|---|---|
| complete mature package fixture | `validate_stage2_mature_package_ref_set(...).status == "PASS"` |
| missing FactorRunSpec ref | `BLOCKED` with field `factor_run_spec_refs` |
| `n/a-with-reason` label window | `NEEDS_REVIEW` only when typed reason/source/remediation are present; otherwise `BLOCKED` |
| invalid or stale evidence hash | `BLOCKED` or `NEEDS_REVIEW` with evidence field named |
| nonzero forbidden counter | `BLOCKED`; reason names the counter |
| backward compatibility | Existing CR150/CR151/CR154 Stage 2 tests still pass |

## 11. 实施步骤

1. Add constants and `Stage2MaturePackageRefSet`.
2. Add builder and validator without changing current public behavior.
3. Wire optional `stage2_exit_refs` into bundle builder.
4. Add fixture tests for complete/missing/stale/nonzero-counter cases.
5. Run the Stage 2 test subset and CP6 return/evidence checks.

## 12. 风险、难点与预研建议

| Risk | Mitigation |
|---|---|
| Existing bundle callers break on dataclass changes. | Add optional field at end with default; avoid required constructor args. |
| Required ref model overfits CR157 fixture. | Keep schema based on Stage 2 exit objects, not fixture file names. |
| `n/a-with-reason` masks true gaps. | Require explicit `TypedUnavailable` with source/remediation; default missing stays `BLOCKED`. |

### 12.1 Clarification Queue

| ID | Status | Question | Resolution |
|---|---|---|---|
| CQ-CR157-S01-001 | RESOLVED | Does S01 authorize real-data validation of refs? | No. Refs are static/fixture-only until a later explicit gate. |

No OPEN or Spike question blocks CP5.

## 13. 回滚与发布策略

- Rollback is file-local: remove new ref-set dataclass, constants, builder, validator and optional bundle field.
- No persistence migration is required.
- No release/publish action is authorized by this Story.

## 14. Definition of Done

- Full LLD is approved in CP5.
- Package ref-set builder and validator are implemented under CP6 only after CP5 approval.
- Tests cover complete, missing, stale/invalid, typed-unavailable and forbidden-counter paths.
- CP6 evidence index points to tests and implementation; it does not copy full reports.

## 人工确认区

CP5 reviewer should confirm S01 as the owner of the Stage 2 mature package ref set and serial merge order before S02-S04 implementation consumes this contract.
