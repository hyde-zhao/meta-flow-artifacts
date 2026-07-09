---
status: "draft"
version: "1.0"
cr_id: "CR-160"
title: "Stage 4 Observation Review Checklist"
owner: "meta-se"
created_at: "2026-07-08"
source_hld: "docs/design/HLD-STAGE4-OBSERVATION-REVIEW-WORKFLOW.md"
review_mode: "human-review-tool"
automatic_checker: "out-of-scope-for-CR160"
---

# Observation Review Checklist

This checklist is a human review tool. It does not execute a checker, run a strategy, access lake/NAS/provider/broker credentials, mutate catalog/store/registry, or authorize paper/simulation/runtime/live/trading. After human review, use the HLD decision table to classify the artifact.

## Result Legend

| Status | Meaning | Decision impact |
|---|---|---|
| PASS | Evidence is present, internally consistent and within authorization boundary. | May support next decision table step. |
| NEEDS_REVIEW | Evidence exists but has non-blocking ambiguity or accepted risk requiring reviewer note. | Cannot by itself authorize paper/simulation/runtime. |
| FAIL | Required evidence is missing, contradicted, or blocked. | Fail-closed classification required. |
| N/A | Item is not applicable with explicit reason. | Must not hide a missing required item. |

## Review Header

| Field | Reviewer entry |
|---|---|
| Review ID |  |
| Strategy / package ID |  |
| Stage 3 package ref |  |
| Observation plan instance ref |  |
| Observation plan template ref | `docs/design/HLD-STAGE4-OBSERVATION-REVIEW-WORKFLOW.md#7-observation-plan-template-contract` |
| Evidence lane | `contract_only` / `real_data_validated` / `runtime_authorized` / `unknown` |
| Reviewer |  |
| Reviewed at |  |
| Non-authorization acknowledged | yes / no |

## Stage 1: Data Foundation Review

| # | Review item | Expected evidence | Status | Reviewer notes |
|---|---|---|---|---|
| S1-01 | Universe is explicit and stable. | `ResearchDatasetSpec.universe` or equivalent universe ref. |  |  |
| S1-02 | Leakage policy is explicit. | `ResearchDatasetSpec.leakage_policy`; PIT and embargo assumptions. |  |  |
| S1-03 | Output snapshot is traceable. | `ResearchDatasetSpec.output_snapshot_id` or immutable snapshot ref. |  |  |
| S1-04 | Lineage refs are present. | `ResearchDatasetSpec.lineage_refs` for key input datasets. |  |  |
| S1-05 | PIT gate is satisfied or blocked reason recorded. | PIT gate result, not a narrative claim only. |  |  |
| S1-06 | Backtest/run input spec is present. | `BacktestRunSpec` or equivalent run input contract. |  |  |
| S1-07 | Data quality gaps are classified. | typed unavailable, blocked reasons, or governance route. |  |  |
| S1-08 | No new data action is implied by review. | Review cites existing refs only; no lake/NAS/provider operation request. |  |  |

Stage 1 fail-closed rule: any missing PIT/leakage/snapshot/lineage evidence on the critical path blocks observation candidate classification and routes to `needs_real_data_validation` or `data_lake_governance`.

## Stage 2: Research Production Review

| # | Review item | Expected evidence | Status | Reviewer notes |
|---|---|---|---|---|
| S2-01 | Factor specs are referenced. | `Stage2MaturePackageRefSet.factor_spec_refs`. |  |  |
| S2-02 | Factor run specs are referenced. | `Stage2MaturePackageRefSet.factor_run_spec_refs`. |  |  |
| S2-03 | Factor panel ref is present. | `Stage2MaturePackageRefSet.factor_panel_ref`. |  |  |
| S2-04 | Evaluation report refs are present. | `Stage2MaturePackageRefSet.evaluation_report_refs`. |  |  |
| S2-05 | Research evidence index is present. | `Stage2MaturePackageRefSet.research_evidence_index` or ref. |  |  |
| S2-06 | `typed_unavailable` is reviewed. | Every unavailable critical evidence item has reason and remediation. |  |  |
| S2-07 | `blocked_reasons` are reviewed. | No unresolved blocker on observation critical path. |  |  |
| S2-08 | Stage 2 output stays refs-only. | No copied report body is treated as new truth source. |  |  |
| S2-09 | Stage 2 does not claim runtime authorization. | `not_runtime_authorization=true` or equivalent wording. |  |  |

Stage 2 fail-closed rule: unresolved critical `typed_unavailable` or `blocked_reasons` prevents observation candidate classification. Contract-only Stage 2 evidence can produce remediation or real-data-validation route only.

## Stage 3: Research Machine Admission Review

| # | Review item | Expected evidence | Status | Reviewer notes |
|---|---|---|---|---|
| S3-01 | Run manifest is present and consistent. | `Stage3MatureResearchPackage.run_manifest`. |  |  |
| S3-02 | Required input refs are present. | `Stage3MatureResearchPackage.input_refs`. |  |  |
| S3-03 | Required evidence refs are present. | `Stage3MatureResearchPackage.evidence_refs`. |  |  |
| S3-04 | Research evidence index is present. | `Stage3MatureResearchPackage.research_evidence_index`. |  |  |
| S3-05 | Admission package ref is present. | `Stage3MatureResearchPackage.admission_package_ref` or mature admission package ref. |  |  |
| S3-06 | Observation plan instance ref is present. | `Stage3MatureResearchPackage.observation_plan_ref` points to an instance, not only CR160 template. |  |  |
| S3-07 | Observation plan instance complies with template. | Period, cadence, metrics, exit criteria, remediation triggers, pause triggers, auth boundary. |  |  |
| S3-08 | Blocked claims are explicit. | `blocked_claims` includes runtime/simulation/live/gateway forbidden claims where applicable. |  |  |
| S3-09 | Unlock conditions do not imply authorization. | Unlock conditions require separate gates. |  |  |
| S3-10 | Admission package status is reviewed. | PASS / BLOCKED / FAIL and reason refs. |  |  |
| S3-11 | Rerun consistency is reviewed if claimed. | Rerun consistency evidence ref and outcome. |  |  |
| S3-12 | Economic/OOS/capacity/risk blockers are reviewed. | Admission report, validation report, risk/capacity refs. |  |  |

Stage 3 fail-closed rule: Stage 3 admission `BLOCKED` or statistical gate `FAIL` maps to `blocked_admission_failed` unless future evidence changes under a separate CR. Missing observation plan instance maps to `not_reviewable` or `needs_remediation`.

## Cross-Cutting Authorization and No-Overclaim Review

| # | Review item | Expected evidence | Status | Reviewer notes |
|---|---|---|---|---|
| C-01 | Evidence lane is declared. | `contract_only`, `real_data_validated`, `runtime_authorized`, or `unknown`. |  |  |
| C-02 | Contract-only ceiling is enforced. | No `paper_candidate=true`, `simulation_ready=true`, or `runtime_authorized=true`. |  |  |
| C-03 | Existing evidence consumption is separated from new data access. | Prior evidence refs are cited without new lake/NAS/provider operation. |  |  |
| C-04 | Forbidden operations remain denied. | No runtime, paper, simulation, live, broker, trading, credential, provider, NAS, catalog/store/registry, Git remote, release, publish. |  |  |
| C-05 | Stage 5 handoff is non-authorizing. | `PaperSimulationAdmissionView.not_simulation_auth=true`; `simulation_candidate=false` under CR160. |  |  |
| C-06 | Review wording does not promote blocked artifacts. | BLOCKED/FAIL artifacts cannot be observation candidates. |  |  |
| C-07 | Decision route is separate from execution. | Routes create follow-up candidates only; no action is executed. |  |  |
| C-08 | Waiver or N/A rationale is explicit. | Every N/A has reason; no waiver bypasses forbidden authorization. |  |  |

Cross-cutting fail-closed rule: any authorization ambiguity or forbidden claim maps to `authorization_blocked` and must be relayed to host-orchestrator. No reviewer may use this checklist to authorize runtime or data operations.

## Observation Plan Template Compliance

| Template field | Required instance content | Status | Reviewer notes |
|---|---|---|---|
| `strategy_id` | Matches Stage 3 package. |  |  |
| `observation_period` | Minimum duration and start/end rules. |  |  |
| `checkpoint_frequency` | Cadence and reviewer responsibility. |  |  |
| `tracking_metrics` | Return, drawdown, turnover/cost, exposure/drift, data quality, risk/capacity, blocked claims. |  |  |
| `exit_criteria` | Graduate / continue / fail / escalate conditions. |  |  |
| `remediation_triggers` | Conditions returning to strategy remediation or validation. |  |  |
| `pause_triggers` | Conditions requiring pause before further observation. |  |  |
| `authorization_boundary` | Explicitly not paper/simulation/runtime authorization. |  |  |
| `evidence_refs` | Stage 1/2/3 refs and missing evidence refs. |  |  |

## Post-Review Decision Table

| Review outcome | Classification | Escalation route | Paper candidate | Simulation ready | Runtime authorized |
|---|---|---|---:|---:|---:|
| Contract-only evidence without real validation | `needs_real_data_validation` | real_data_validation | false | false | false |
| Real-data evidence, Stage 3 admission PASS, compliant plan | `observation_candidate` | no_action or future paper_authorization request | false under CR160 | false under CR160 | false |
| Stage 3 admission BLOCKED or statistical gate FAIL | `blocked_admission_failed` | strategy_remediation | false | false | false |
| Missing or non-compliant observation plan instance | `not_reviewable` or `needs_remediation` | strategy_remediation | false | false | false |
| PIT/lineage/data quality foundation incomplete | `needs_real_data_validation` or `data_lake_governance` | real_data_validation or data_lake_governance | false | false | false |
| Forbidden claim or authorization ambiguity | `authorization_blocked` | runtime_authorization or host decision | false | false | false |
| Unknown or contradictory evidence | `not_reviewable` | host decision | false | false | false |

## Reviewer Sign-Off

| Item | Value |
|---|---|
| Final classification |  |
| Escalation route candidate |  |
| Blocking findings |  |
| Non-blocking risks |  |
| Evidence refs used |  |
| Reviewer confirms no runtime/data/paper/simulation authorization | yes / no |

