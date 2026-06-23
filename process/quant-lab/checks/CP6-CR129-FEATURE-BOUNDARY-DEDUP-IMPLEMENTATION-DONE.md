---
checkpoint: "CP6"
cr_id: "CR-129"
title: "Feature Boundary Dedup Implementation Evidence"
status: "PASS"
created_at: "2026-06-23T16:58:00+08:00"
owner: "host-orchestrator"
context_ref: "process/context/CP6-CR129.context.json"
summary_ref: "process/changes/summaries/CR-129.summary.json"
authorization: "feature docs boundary, empty directory cleanup, lightweight checker/tests and process evidence only"
---

# CP6 - CR129 Feature Boundary Dedup Implementation Evidence

## Entry Criteria

| Item | Result | Evidence |
|---|---|---|
| CR128 closed | PASS | `process/changes/summaries/CR-128.summary.json` shows closed/ready |
| CR129 active and authorized | PASS | `process/changes/CR-129-FEATURE-BOUNDARY-DEDUP-STRATEGY-RUNNER-CORE-AUTHORITY-2026-06-23.md` |
| Scope excludes external permission domains | PASS | CR129 `## 不授权范围` |
| Machine context exists | PASS | `process/context/CP6-CR129.context.json` |

## Implemented Objects

| Object | Action | Notes |
|---|---|---|
| `process/docs/features/cr102-nas-real-package-exchange-authorization/` | removed | Empty default feature directory removed; history remains in CR102 / CR128 / CR129 evidence. |
| `process/docs/features/cr103-qmt-miniqmt-non-trading-day-validation/` | removed | Empty default feature directory removed. |
| `process/docs/features/cr104-qmt-miniqmt-trading-day-validation/` | removed | Empty default feature directory removed. |
| `process/docs/features/qmt-miniqmt-dual-target-framework/DESIGN.md` | updated | Added `retained_as=legacy_cross_target_framework`, `superseded_by.offline_runner_implementation_authority`, revision row and boundary section. |
| `process/docs/features/execution-semantics-reference/DESIGN.md` | updated | Added Strategy Runner Core boundary section. |
| `process/docs/features/factor-research-loop/DESIGN.md` | updated | Added Strategy Runner Core boundary section. |
| `process/docs/features/qmt-gateway-readonly/DESIGN.md` | updated | Added Strategy Runner Core boundary section. |
| `process/docs/features/qmt-trading-governance/DESIGN.md` | updated | Added Strategy Runner Core boundary section. |
| `process/docs/features/runtime-authorization-safety/DESIGN.md` | updated | Added Strategy Runner Core boundary section. |
| `scripts/check_feature_runner_boundary.py` | added | Checks forbidden CR102/CR103/CR104 default feature entries and required boundary markers. |
| `tests/test_cr129_feature_runner_boundary.py` | added | Covers current docs pass, forbidden CR102 entry fail and CLI JSON output. |

## Contract Mapping

| CR129 Acceptance Item | Implementation Evidence |
|---|---|
| Delete CR102/CR103/CR104 empty directories | `find -L process/docs/features -maxdepth 1 -type d` no longer lists them. |
| Mark qmt-miniqmt dual-target as legacy | Frontmatter contains `retained_as: legacy_cross_target_framework`. |
| Point offline runner authority to strategy-runner-core | Frontmatter and boundary section point to `process/docs/features/strategy-runner-core/DESIGN.md`. |
| Add adjacent boundary notes | Five adjacent Feature DESIGN files contain `## 与 Strategy Runner Core 的边界`. |
| Add lightweight guardrail | `scripts/check_feature_runner_boundary.py` and `tests/test_cr129_feature_runner_boundary.py`. |

## Non-Authorized Boundary

CR129 did not authorize or execute:

- `.env`, token, secret, credential, account, funds, positions, orders, fills, session, cookie or raw log reads.
- NAS mount/list/copy/read/write/publish/pull/delete.
- QMT, MiniQMT, XtQuant or gateway runtime startup/connection.
- Provider fetch, lake write, catalog publish or current pointer modification.
- submit/cancel/buy/sell, simulation or live trading.
- `.gitignore` changes, commit, push or Git remote write.

## Exit Criteria

| Item | Result | Evidence |
|---|---|---|
| Feature boundary docs updated | PASS | Implemented objects table |
| Forbidden legacy feature entries removed | PASS | Local feature root listing and checker |
| Guardrail implemented with tests | PASS | `tests/test_cr129_feature_runner_boundary.py` |
| External permission domains remain excluded | PASS | Non-authorized boundary |

## Decision

CP6 result: PASS. CR129 is ready for CP7 verification.
