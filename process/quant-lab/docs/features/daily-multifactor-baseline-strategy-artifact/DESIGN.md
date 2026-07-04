---
status: "cp4-planned"
version: "0.1"
feature_id: "FEAT-16"
cr_id: "CR-155"
title: "Daily Multifactor Baseline Strategy Artifact Design"
source_hld: "docs/design/HLD-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-DAILY-MULTIFACTOR-BASELINE-STRATEGY-ARTIFACT.md"
---

# Daily Multifactor Baseline Strategy Artifact Design

## Revision Record

| Version | Date | Author | Change |
|---|---|---|---|
| 0.1 | 2026-07-04 | host-orchestrator | Initial CP4 Feature design for CR155 Story planning and CP5 design evidence preparation. |

## Capability Boundary

FEAT-16 owns one auditable daily multifactor baseline strategy artifact. The artifact is a concrete research object, not an optimizer result, production strategy, paper/live authorization or trading readiness claim.

Required artifact fields:

| Field Group | Required Content |
|---|---|
| Identity | `strategy_id`, version, artifact schema version, created run id, claim boundary. |
| Universe | PIT universe ref, date range, inclusion/exclusion policy, survivorship/PIT evidence refs. |
| Factors | Factor spec refs, weighting policy, neutralization/standardization policy, missing value policy. |
| Signal | Signal construction, lag policy, rebalance calendar, rank/score direction, tie handling. |
| Portfolio | Portfolio policy, sizing bounds, turnover controls, cost/slippage policy, benchmark/risk-free refs. |
| Validation | Historical backtest refs, OOS/walk-forward refs, split manifest, cost/risk/attribution refs. |
| Admission | Statistical gate result ref, reliability gate result ref, final admission package ref. |
| Evidence | Two rerun refs, metric diff summary, evidence index refs, release wording refs. |

## Story Split Decisions

| Story | Responsibility | Must Not Do |
|---|---|---|
| CR155-S01 | Define artifact contract and non-optimal baseline claim boundary. | Read lake, run backtest, implement source. |
| CR155-S02 | Define readonly provenance adapter and forbidden operation counters. | Write lake, mutate catalog, read credentials, access NAS/provider/runtime. |
| CR155-S03 | Define historical backtest and OOS/walk-forward validation flow. | Introduce optimizer/model search or external framework execution. |
| CR155-S04 | Define statistical + reliability gate composition and admission package. | Collapse gate reasons into one untraceable status. |
| CR155-S05 | Define rerun consistency evidence and release wording. | Claim paper/live readiness or production deployment. |

## Authorization Boundary

CP2 approved CR155-scoped local governed lake/current truth readonly for historical backtest, OOS/walk-forward and rerun consistency. CP4 only plans Story and design evidence. CP4 does not authorize LLD approval, source implementation, test implementation, lake write, catalog mutation, NAS/provider/credential access, runtime, broker, trading, external framework execution, store/registry writes, publish or production deployment.

## CP5 Attention Items

| ID | Requirement |
|---|---|
| CP5-FOCUS-CR155-001 | S01 LLD must define exact required artifact fields and failure behavior for missing refs. |
| CP5-FOCUS-CR155-002 | S02 LLD must define readonly provenance proof, forbidden operation counters and downgrade behavior if provenance cannot be proven. |
| CP5-FOCUS-CR155-003 | S03 LLD must define split manifest, purge/embargo policy, OOS/walk-forward metrics and CR148 consumption refs. |
| CP5-FOCUS-CR155-004 | S04 LLD must preserve separate statistical gate, reliability gate and final admission package statuses. |
| CP5-FOCUS-CR155-005 | S05 LLD must define two-run metric comparison keys, tolerance policy and release wording that blocks production readiness overclaim. |

## Gotchas

1. Do not treat `paper_candidate=true` as paper trading authorization.
2. Do not infer lake write permission from readonly approval.
3. Do not hide rerun drift with free-text explanations; drift must become structured evidence.
4. Do not replace CR151/CR154 gates with a CR155-specific shortcut.
