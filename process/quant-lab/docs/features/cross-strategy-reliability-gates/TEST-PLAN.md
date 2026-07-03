---
feature_id: "FEAT-15"
change_id: "CR-154"
status: "cp4-story-planning"
version: "0.1"
created_at: "2026-07-03T10:40:00+08:00"
owner: "host-orchestrator"
implementation_allowed: false
---

# Cross-Strategy Reliability Gates Test Plan

## Purpose

Define CP5/CP6/CP7 validation expectations for CR154 without authorizing implementation. All validation remains local/static/fixture-only unless a later human gate explicitly changes the boundary.

## Test Levels

| Level | Scope | Required Coverage |
|---|---|---|
| Contract unit tests | Shared summary, artifact refs, status enum, blocked claims. | PASS/FAIL/NEEDS_REVIEW/BLOCKED, `n/a-with-reason`, forbidden operation counters. |
| Gate policy tests | Gate 1-6 evidence policy behavior. | Missing mandatory evidence, release profile severity, fail-closed unknown profile, impact enum validation. |
| Adapter tests | Multifactor / ML / event-driven mapping. | Strategy-specific refs, ML-only n/a policy, CR153 `universe_pit_audit` delegation. |
| Integration tests | Admission package summary and wording. | Blocked claims, release-blocking reason, no runtime/trading readiness claims. |
| Regression tests | CR151 / CR152 / CR153 existing fixture behavior. | Existing first-wave gates remain consumable and no prior package-visible semantics are silently removed. |

## Fixture Matrix

| Fixture Group | Story Owner | Required Cases |
|---|---|---|
| Shared fixture skeleton | S01 | Minimal PASS, missing artifact `BLOCKED`, `NEEDS_REVIEW`, forbidden operation counter `BLOCKED`. |
| Statistical artifacts | S02 | Complete 12-slot evidence, missing WRC/SPA severity, missing PBO/DSR, trial count invalid, Gate 3/4 propagation. |
| CV governance | S03 | Walk-forward present, OOS split missing, purge/embargo missing, ML-specific CV mapped without forcing ML-only semantics on other strategies. |
| PIT universe | S04 | PIT pass, non-PIT blocked, CR153 slot delegated, CR153 slot absent with blocked claim. |
| Capacity / impact | S05 | Allowed impact enum, custom with rationale, invalid enum, cost-underestimation blocked, no-real-TCA wording. |
| Regime / attribution / reconciliation | S06 | Slot pass, slot n/a-with-reason, missing reason blocked/review per tier, no runtime reconciliation claim. |
| Admission tier | S07 | T0 opt-in, T1 default-required, T2 release-blocking, T3 not-authorized, unknown profile fail-closed. |
| Compatibility / wording | S08 | MF-GAP-2/6/7 deferred wording, CR151/152/153 summary compatibility, no overclaim release text. |

## Forbidden Validation

The following must not appear in CR154 tests unless a later authorization gate changes the boundary:

- Reading `.env`, secrets, accounts, tokens, session or broker credentials.
- Real lake / NAS / provider access.
- Real event feed, live listener, model registry, event store, feature store, prediction store or catalog pointer write.
- QMT / MiniQMT / xtquant / gateway runtime, simulation, paper, live, trading, broker query, submit or cancel.
- External framework clone/install/run or Git remote write.

## CP5 Acceptance Expectations

CP5 design evidence must show the planned tests before implementation starts. In particular:

- S01 must own fixture schema and the first runnable fixture.
- S02 must explicitly test WRC/SPA severity and Gate 3/4 propagation.
- S05 must test `square_root`, `almgren_chriss`, `gatheral`, `custom` and `n/a-with-reason`.
- S07 must test fail-closed unknown release profile.
- S08 must prove release wording cannot claim production, paper/live, broker, real data or trading readiness.
