---
document_id: "TEST-STRATEGY-CR151"
project_id: "quant-lab"
cr_id: "CR-151"
title: "CR151 Multifactor Strategy E2E Statistical Admission Test Strategy"
status: "active"
owner: "meta-qa"
created_at: "2026-07-02T08:45:00+08:00"
updated_at: "2026-07-02T08:45:00+08:00"
validation_mode: "static-only"
canonical_for:
  - "CR-151"
source_refs:
  - "process/checks/CP6-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE-IMPLEMENTATION.result.json"
  - "process/checks/CP7-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE-VERIFICATION.result.json"
  - "process/evidence/CR151-CP7-VERIFICATION.index.json"
  - "process/returns/CR151-STRATEGY-ADMISSION-STATISTICAL-GATE.CP7.return.json"
---

# CR151 Test Strategy

## Revision History

| Version | Date | Author | Change |
|---|---|---|---|
| v1.0 | 2026-07-02 | host-orchestrator | Created canonical CR151 static-only test strategy and closed `CR151-CP7-R01`. |

## Scope

This document is the canonical test strategy for CR151, "Multifactor Strategy E2E Statistical Admission Framework".

CR151 validates the local/static/fixture strategy admission layer for multifactor research. The scope is limited to:

| Area | In Scope |
|---|---|
| Statistical report contracts | Multiple-testing/FDR, robust factor statistics, walk-forward/OOS plan, and backtest overfit risk report metadata. |
| Fail-closed evaluator | Aggregate `StrategyAdmissionStatisticalGate` status precedence, mandatory evidence handling, threshold checks, and forbidden operation counters. |
| Admission package linkage | Mapping CR151 statistical gate statuses to existing `AdmissionStatus` without creating a parallel package contract. |
| CR150 completion map linkage | Optional `statistical_admission_gate` node and `statistical_admission_gate_missing` gap while preserving CR150 default behavior. |
| Process evidence | CP6/CP7 result JSON, evidence index, return packet, meta-qa-critical dispatch evidence, and verify packet traceability. |

Out of scope:

| Area | Status |
|---|---|
| Real data lake or NAS validation | Not authorized. |
| Provider fetch, QMT, runtime, simulation, live, trading, broker, credentials, external framework, Git remote | Not authorized. |
| ML strategy E2E framework | Deferred to CR152. |
| Event-driven strategy E2E framework | Deferred to CR153. |
| Wave B evaluation artifacts such as capacity/impact, benchmark-relative reporting, PIT universe audit, extended factor diagnostics, regime-aware validation, and factor correlation clustering/de-duplication | Deferred to CR154 or later follow-up scope. |

## Validation Mode

`validation_mode = static-only`.

The CR151 evidence proves local Python contracts, fixture behavior, static boundary checks, and process traceability. It does not prove production runtime readiness, real-data correctness, NAS consistency, provider integration, QMT integration, simulation readiness, live readiness, or trading readiness.

## Test Design Method

| Method | CR151 Usage | Evidence |
|---|---|---|
| Equivalence partitioning | PASS/FAIL/NEEDS_REVIEW/BLOCKED gate states, required versus optional evidence, present versus missing CR150 linkage. | `tests/test_cr151_strategy_admission_statistical_gate.py`, `tests/test_cr150_multifactor_framework_completion.py` |
| Boundary value analysis | P-value count consistency, rejected count, fold pass rate, PBO/DSR thresholds, operation counter boundaries. | CR151 targeted tests and CP7 malformed-evidence probes. |
| State transition testing | Mandatory evidence missing or malformed must block before PASS; forbidden operation counters must block before threshold PASS. | CP7 evidence commands 4-6. |
| Error guessing | False PASS from malformed metadata, false runtime-readiness claims, accidental runtime/NAS/provider/broker/Git remote operations. | CP7 boundary check and dangerous command scan. |

## ISO 25010 Priorities

| Characteristic | Priority | CR151 Focus |
|---|---|---|
| Functional suitability | P0 | Statistical reports, fail-closed evaluator, admission package mapping, and completion-map linkage match CP5-approved LLD. |
| Reliability | P0 | Malformed mandatory evidence produces BLOCKED; forbidden operation counters preempt PASS. |
| Security | P0 | No credential, real lake, NAS, provider, QMT, runtime, simulation, live, trading, broker, external framework, or Git remote operation is authorized or executed. |
| Maintainability | P1 | Contracts are frozen dataclasses / JSON-safe outputs and reuse existing admission package structures. |
| Compatibility | P1 | CR150 default completion map behavior remains unchanged unless `require_statistical_gate=True`. |
| Portability | P1 | Validation runs under `uv run --python 3.11` with local tests and no external services. |
| Usability | P2 | Gate status and reasons are inspectable via structured summaries and stable gap codes. |
| Performance efficiency | P3 | Scope uses small local fixture tests; no full data scan or runtime benchmark is performed. |

## Entry Criteria

| Criteria | Status | Evidence |
|---|---|---|
| CP2 scope approved as local/static/fixture and no-runtime. | PASS | `process/checks/CP2-CR151-MULTIFACTOR-STRATEGY-E2E-SCOPE.result.json` |
| CP3 architecture approved. | PASS | `process/checks/CP3-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE.result.json` |
| CP5 design evidence approved. | PASS | `process/checks/CP5-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE-LLD-BATCH.result.json` |
| CP6 implementation completed. | PASS | `process/checks/CP6-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE-IMPLEMENTATION.result.json` |
| meta-qa-critical CP7 dispatch evidence exists. | PASS | `ADE-CR151-META-QA-CRITICAL-CP7-START-2026-07-02T065409+0800`, `ADE-CR151-META-QA-CRITICAL-CP7-COMPLETE-2026-07-02T072217+0800`, `ADE-CR151-META-QA-CRITICAL-CP7-REVERIFY-COMPLETE-2026-07-02T073039+0800` |

## Exit Criteria

| Criteria | Status | Evidence |
|---|---|---|
| P0/P1 source-code blockers are closed. | PASS | `CR151-CP7-F01` closed in `process/evidence/CR151-CP7-VERIFICATION.index.json`. |
| Targeted regression tests pass. | PASS | `20 passed` in CP7 evidence command 1. |
| Syntax and whitespace checks pass. | PASS | CP7 evidence commands 2 and 3. |
| Failure route and waiver checks are reproducible. | PASS | CP7 evidence commands 7 and 8 include full `--result` paths. |
| Verify packets include acceptance and traceability fields. | PASS | `CR151-CP7-R02` closed before CP8. |
| Runtime and data-operation boundaries remain closed. | PASS | CP7 boundary counters are all zero. |

## Verification Matrix

| Story | Verification Target | Test / Check | Status |
|---|---|---|---|
| CR151-S01 | Statistical report contracts and validation helpers. | `tests/test_cr151_strategy_admission_statistical_gate.py`; malformed multiple-testing probe. | PASS |
| CR151-S02 | Fail-closed gate evaluator and operation counter precedence. | `tests/test_cr151_strategy_admission_statistical_gate.py`; aggregate malformed evidence probe; forbidden counter probe. | PASS |
| CR151-S03 | Admission package status mapping and CR150 completion-map linkage. | `tests/research/test_strategy_admission_package.py`; `tests/test_cr150_multifactor_framework_completion.py`. | PASS |
| CR151-S04 | Static-only evidence and release wording boundary. | CP7 result/evidence/return packet review; TEST-STRATEGY boundary statement. | PASS |

## Required Commands

The canonical CR151 verification command set is:

```bash
uv run --python 3.11 pytest -q tests/test_cr151_strategy_admission_statistical_gate.py tests/test_cr150_multifactor_framework_completion.py tests/research/test_strategy_admission_package.py
uv run --python 3.11 python -m py_compile engine/strategy_admission_statistical_gate.py engine/strategy_admission_package.py engine/mature_multifactor_research.py
git diff --check
uv run --python 3.11 meta-flow cp result-check --result process/checks/CP7-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE-VERIFICATION.result.json --project-root /home/hyde/workspace/quant-lab
uv run --python 3.11 meta-flow failure route-check --result process/checks/CP7-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE-VERIFICATION.result.json --project-root /home/hyde/workspace/quant-lab
uv run --python 3.11 meta-flow waiver check --result process/checks/CP7-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE-VERIFICATION.result.json --project-root /home/hyde/workspace/quant-lab
```

## Non-Authorized Items

CR151 validation and CP8 readiness do not authorize:

| Item | Authorization |
|---|---|
| Real data lake read/write, catalog pointer mutation, or provider-backed data validation | Not authorized |
| NAS read/write/sync or NAS metadata normalization | Not authorized |
| Credential or `.env` read | Not authorized |
| QMT, broker, trading, simulation, live, live-readonly, small-live, or scale-up operations | Not authorized |
| External framework execution or upload | Not authorized |
| Git remote fetch, push, or publish | Not authorized |

## Current Risk Position

| Risk ID | Status | Notes |
|---|---|---|
| `CR151-CP7-R01` | CLOSED | This canonical `docs/quality/TEST-STRATEGY.md` closes the prior stale/missing TEST-STRATEGY risk. |
| `CR151-CP7-R02` | CLOSED | CP7 verify packets now contain acceptance criteria, non-empty verification plans, test refs, design contract refs, and design evidence refs. |
| `CR151-CP8-R03-STATE-V2-HYGIENE` | OPEN | `meta-flow state check` still fails because STATE artifacts exceed v2 size limits and retain long-running fields. This is a process readiness caveat, not a CR151 source-code blocker. |

## CP8 Recommendation

CR151 can proceed to CP8 as `READY_WITH_RISK` for local/static/fixture release readiness.

The only known open readiness caveat after this strategy update is `CR151-CP8-R03-STATE-V2-HYGIENE`. CP8 approval must not be interpreted as real runtime, real data, NAS, provider, QMT, simulation, live, trading, broker, credential, external framework, Git remote, or production release authorization.
