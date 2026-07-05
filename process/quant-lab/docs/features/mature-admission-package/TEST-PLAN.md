---
feature_id: "FEAT-17"
change_id: "CR-157"
status: "cp4-story-planning"
version: "0.1"
created_at: "2026-07-05T13:18:00+08:00"
owner: "host-orchestrator"
implementation_allowed: false
---

# Mature Admission Package Test Plan

## Purpose

Define CP5/CP6/CP7 validation expectations for CR157 mature admission package builder work. All validation remains local/static/fixture-only unless a later human gate explicitly changes the boundary.

## Test Levels

| Level | Scope | Required Coverage |
|---|---|---|
| Contract tests | Package schema, status enum, mandatory ref families, blocked claims. | Complete package, missing mandatory ref, invalid hash, stale ref, `n/a-with-reason`. |
| Builder tests | Deterministic assembly from existing Stage 2 refs. | Stable ordering, duplicate ref rejection, source CR preservation, no embedded report body. |
| Safety tests | Forbidden operation counters. | Nonzero lake/NAS/provider/QMT/runtime/trading/store/publish counters force `BLOCKED`. |
| Handoff tests | Package readiness consumed by FEAT-19. | Complete package passes to handoff; blocked package carries explicit reasons. |

## Fixture Matrix

| Fixture Group | Story Owner | Required Cases |
|---|---|---|
| Complete static package | S01 | All mandatory refs present and evidence index linked. |
| Missing mandatory refs | S01 | Missing FactorSpec, FactorRunSpec, factor panel, label window, evaluation, portfolio/risk, evidence index. |
| Ref integrity | S01 / S02 | Invalid hash, stale ref, duplicate ref id, unsupported source CR. |
| Forbidden operations | S01 / S04 | Any nonzero forbidden counter results in `BLOCKED`. |

## Forbidden Validation

No test may read `.env`, credentials, real lake/NAS/provider data, QMT/MiniQMT/xtquant/gateway runtime, simulation/paper/live/trading, broker account, feed, store, catalog, registry, publish target, external framework runtime or Git remote.
