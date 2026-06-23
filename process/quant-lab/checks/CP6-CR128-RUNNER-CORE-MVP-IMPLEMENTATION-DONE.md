---
checkpoint: "CP6"
cr_id: "CR-128"
title: "Runner Core MVP Implementation Evidence"
status: "PASS"
created_at: "2026-06-23T16:18:00+08:00"
owner: "host-orchestrator"
context_ref: "process/context/CP6-CR128.context.json"
summary_ref: "process/changes/summaries/CR-128.summary.json"
authorization: "offline runner core source/tests/process evidence and feature docs governance only"
---

# CP6 - CR128 Runner Core MVP Implementation Evidence

## Entry Criteria

| Item | Result | Evidence |
|---|---|---|
| CR128 active and authorized | PASS | `process/changes/CR-128-RUNNER-CORE-MVP-IMPLEMENTATION-2026-06-23.md` |
| Scope excludes external permission domains | PASS | CR128 frontmatter and `## 不授权范围` |
| Machine state and context entry available | PASS | `process/state/STATE.current.json`, `process/context/CP6-CR128.context.json` |
| Runner feature governance authorized | PASS | User request plus CR128 document handling decisions |

## Implemented Objects

| Object | Action | Notes |
|---|---|---|
| `trading/strategy_runner/run_spec.py` | added | Defines CR128 offline `RunSpec`, schema version, authorization flags, sensitive path guard and fail-closed validation. |
| `trading/strategy_runner/result.py` | added | Defines `RunResult`, result serialization, blocked result construction and `write_run_result`. |
| `trading/strategy_runner/runner.py` | added | Orchestrates package loader, adapter dispatch, fake readonly boundary, evidence summary and result output. Output write is allowed only after `RunSpec.validate()` passes. |
| `trading/strategy_runner/__init__.py` | updated | Exposes `RunSpec`, `RunSpecError`, `RunResult`, `run_strategy_package`, `run_strategy_package_from_path` and `write_run_result`. |
| `scripts/check_cr091_strategy_runner_package.py` | updated | Reuses the official CR128 runner core API while preserving existing JSON checker contract. |
| `tests/test_cr128_runner_core_mvp.py` | added | Covers fixture happy path, JSON output, authorization fail-closed, non-offline rejection, invalid-spec no-write, missing package fail-closed, no runtime transport, source import guard and checker script smoke. |

## Feature Directory Governance

| Feature area | Action | Result |
|---|---|---|
| `process/docs/features/strategy-runner-core/DESIGN.md` | added | Consolidates offline runner core design and declares CR102/CR103/CR104 runtime/NAS fragments outside the default feature surface. |
| `process/docs/features/strategy-runner-core/TEST-PLAN.md` | added | Captures CR128 offline validation and regression scope. |
| `process/docs/features/strategy-runner-core/TASKS.md` | added | Collapses RDS-01..S6 into CR128 internal implementation tasks. |
| `process/docs/features/cr102-nas-real-package-exchange-authorization/*` | removed from default feature surface | Historical trace remains in formal CR102, CP/release context and CR128 document handling decision. |
| `process/docs/features/cr103-qmt-miniqmt-non-trading-day-validation/VERIFICATION.md` | deleted | Runtime validation fragment no longer represents default runner feature scope. |
| `process/docs/features/cr104-qmt-miniqmt-trading-day-validation/INPUT-CHECKLIST.md` | deleted | Trading-day runtime checklist no longer represents default runner feature scope. |
| `process/docs/design/RUNNER-CORE-MVP-DESIGN-CR126.md` | updated | CR127 numbering corrected to CR128 for implementation; external follow-up gates shifted to CR129+. |

## Contract Mapping

| CR128 Acceptance Item | Implementation Evidence |
|---|---|
| Offline runner core exposes RunSpec | `RunSpec` in `run_spec.py`; rejects non-offline mode and any external authorization flag. |
| Offline runner core exposes RunResult | `RunResult` in `result.py`; includes status, counts, evidence status, forbidden counters and authorization markers. |
| Offline runner core exposes runner API | `run_strategy_package` and `run_strategy_package_from_path` in `runner.py` and public exports. |
| Fixture package runs end-to-end | Covered by `test_cr128_runner_core_runs_fixture_package_end_to_end`. |
| Feature docs consolidate runner scope | New `strategy-runner-core` feature directory. |
| CR102/CR103/CR104 runtime fragments removed from default runner feature surface | CR102 specific feature files absent; CR103/CR104 specific runtime fragments deleted. |
| No runtime/NAS/QMT/credential/provider/lake/catalog/trading/Git remote write | Runner validates flags false, uses only fake readonly boundary, no env/runtime transport imports, and process evidence records no external action. |

## Non-Authorized Boundary

CR128 did not authorize or execute:

- `.env`, token, secret, credential, account, funds, positions, orders, fills, session, cookie or raw log reads.
- NAS mount/list/copy/read/write/publish/pull/delete.
- QMT, MiniQMT, XtQuant or gateway runtime startup/connection.
- Provider fetch, lake write, catalog publish or current pointer modification.
- submit/cancel/buy/sell, simulation or live trading.
- `.gitignore` changes, commit, push or Git remote write.

## Exit Criteria

| Item | Result | Evidence |
|---|---|---|
| Source implementation complete | PASS | Implemented objects table |
| Tests added for new behavior | PASS | `tests/test_cr128_runner_core_mvp.py` |
| Feature directory governance complete | PASS | Feature governance table |
| External permission domains remain excluded | PASS | Non-authorized boundary and test guards |

## Decision

CP6 result: PASS. CR128 is ready for CP7 verification.
