---
story_id: "CR152-S04-ml-admission-gate-adapter"
change_id: "CR-152"
title: "ML admission gate adapter"
status: "ready-for-verification"
owner: "meta-dev"
feature_design_refs:
  - "docs/features/factor-research-loop/DESIGN.md"
  - "docs/features/factor-research-loop/TEST-PLAN.md"
  - "docs/features/runtime-authorization-safety/DESIGN.md"
source_hld: "process/docs/design/HLD-ML-STRATEGY-E2E-FRAMEWORK.md"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-ML-STRATEGY-E2E-FRAMEWORK.md"
lld_policy:
  required: true
  required_level: "full-lld"
  status: "approved"
depends_on:
  - "CR152-S01-pit-feature-label-contracts"
  - "CR152-S02-purged-embargo-cv-fixture-contract"
  - "CR152-S03-training-model-prediction-metadata"
implementation_allowed: true
authorization_boundary: "static/fixture only; no runtime readiness, real training, real data validation, lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/model registry/catalog pointer/Git remote"
---

# CR152-S04 ML Admission Gate Adapter

## Goal

Define ML-specific admission gate summary and adapter to CR151 four-state status semantics and admission package linkage.

## Scope

- ML gate evidence summary and status.
- Adapter mapping to `PASS / FAIL / NEEDS_REVIEW / BLOCKED`.
- Admission package linkage fields: `gate_present`, `gate_required`, `gate_status`, `gate_ref`, blocked reasons.
- Forbidden operation counters.

## Acceptance Criteria

- Missing mandatory ML evidence returns `BLOCKED`.
- Nonzero forbidden operation counters return `BLOCKED`.
- Adapter preserves CR151 four-state semantics without reusing CR151 report schema as ML schema.
- Admission package linkage does not imply runtime, simulation, paper, live, broker or trading readiness.
- If adapter field mapping or contract delta details need architecture deepening, CP5 records meta-se dispatch or explicit inline-fallback approval.

## File Ownership

| File | Intent |
|---|---|
| `engine/ml_strategy_admission_gate.py` | Candidate ML gate companion module, subject to CP5 approval. |
| `engine/strategy_admission_package.py` | Admission package linkage if CP5 approves. |
| `tests/research/test_ml_strategy_e2e_contracts.py` | Gate adapter and package linkage tests. |

## Design Evidence Required For CP5

Full LLD covering status mapping, package fields, blocked reasons, operation counters, adapter fallback, source anchors and no-runtime guardrails.
