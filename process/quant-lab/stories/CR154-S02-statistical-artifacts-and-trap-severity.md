---
story_id: "CR154-S02-statistical-artifacts-and-trap-severity"
change_id: "CR-154"
title: "Gate 1 statistical artifacts and trap severity policy"
status: "dev-ready"
owner: "meta-dev"
feature_design_refs:
  - "process/docs/features/cross-strategy-reliability-gates/DESIGN.md"
  - "process/docs/features/cross-strategy-reliability-gates/TEST-PLAN.md"
  - "process/docs/features/cross-strategy-reliability-gates/TASKS.md"
source_hld: "process/docs/design/HLD-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES.md"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES.md"
lld_policy:
  required: true
  required_level: "full-lld"
  status: "approved"
confirmed: true
confirmed_by: "user"
confirmed_at: "2026-07-03T06:58:00+08:00"
depends_on:
  - "CR154-S01-shared-gate-contract-fixture-skeleton"
implementation_allowed: true
authorization_boundary: "local/static/fixture-only planning; no real statistical calibration or real data validation"
---

# CR154-S02 Gate 1 Statistical Artifacts and Trap Severity Policy

## Goal

Define Gate 1 as auditable statistical reliability evidence, not a generic backtest trap label.

## Scope

- Multiple-testing correction, FDR/BH, WRC/SPA, PBO/CSCV, DSR/deflation.
- Trial count and effective trials.
- OOS split, purge/embargo, survivorship, impact/capacity refs.
- WRC/SPA severity mapping and Gate 3/4 propagation.
- Strategy-family adapter subtask plan.

## Acceptance Criteria

- All 12 CP3-required artifact slots are explicit.
- Missing WRC/SPA or equivalent data-snooping correction maps to `NEEDS_REVIEW` or `BLOCKED` by release profile and claim type.
- Gate 3/4 blocked states propagate into Gate 1 refs, blocked claims and release-blocking reason.
- CP5 LLD splits adapter subtasks for multifactor, ML and event-driven strategies.
- No real lake/provider fetch, real model training or empirical calibration is performed.

## File Ownership

| File | Intent |
|---|---|
| `engine/cross_strategy_reliability_gates.py` | Gate 1 artifact model and severity policy section. |
| `tests/research/test_cross_strategy_reliability_gates.py` | Gate 1 positive, negative and propagation fixtures. |

## Design Evidence Required For CP5

Full LLD covering field names, validators, severity table, default/config threshold policy, adapter subtask table and fixture cases for missing evidence and propagation.
