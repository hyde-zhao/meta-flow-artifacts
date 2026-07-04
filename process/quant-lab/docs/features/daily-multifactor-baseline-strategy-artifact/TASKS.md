---
status: "cp4-planned"
version: "0.1"
feature_id: "FEAT-16"
cr_id: "CR-155"
title: "Daily Multifactor Baseline Strategy Artifact Tasks"
---

# Daily Multifactor Baseline Strategy Artifact Tasks

## Revision Record

| Version | Date | Author | Change |
|---|---|---|---|
| 0.1 | 2026-07-04 | host-orchestrator | Initial CP4 task index for CR155 Story planning. |

## Story Task Index

| Story | CP5 Design Evidence Tasks | CP6 Candidate Implementation Tasks |
|---|---|---|
| CR155-S01 | Define artifact schema, mandatory refs, baseline factor policy and claim boundary. | Add contract object/helpers and schema tests after CP5 approval. |
| CR155-S02 | Define readonly adapter/provenance contract and forbidden counters. | Add adapter guardrails and provenance evidence generation after CP5 approval. |
| CR155-S03 | Define backtest/OOS/walk-forward validation plan and metric set. | Add runner composition and validation report generation after CP5 approval. |
| CR155-S04 | Define gate composition and admission package contract. | Add composition helper and package tests after CP5 approval. |
| CR155-S05 | Define rerun comparison and release evidence wording. | Add rerun diff evidence and release wording checks after CP5 approval. |

## Dependency Order

1. S01 defines artifact contract.
2. S02 defines allowed readonly input and proof.
3. S03 consumes S01/S02 for validation.
4. S04 consumes S01/S03 plus CR151/CR154 gates.
5. S05 consumes S03/S04 for deterministic rerun and release evidence.

## Not Authorized

CP4 does not authorize LLD approval, source implementation, test implementation, real lake write, NAS/provider/credential/runtime/trading/broker operation, catalog/store/registry write, publish or production deployment.
