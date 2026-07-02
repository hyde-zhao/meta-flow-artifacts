---
story_id: "CR151-S03-admission-completion-linkage"
change_id: "CR-151"
title: "Admission and completion linkage"
status: "lld-ready-for-review"
owner: "meta-dev"
feature_design_refs:
  - "docs/features/factor-research-loop/DESIGN.md"
  - "docs/features/factor-research-loop/TASKS.md"
source_hld: "process/docs/design/HLD-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE.md"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE.md"
lld_policy:
  required: true
  required_level: "full-lld"
  status: "pending-cp5"
depends_on:
  - "CR151-S01-statistical-report-contracts"
  - "CR151-S02-gate-evaluator-fail-closed-rules"
implementation_allowed: false
authorization_boundary: "static/fixture only; no real lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote"
---

# CR151-S03 Admission And Completion Linkage

## Goal

Connect the CR151 statistical gate to the existing multifactor admission/completion chain without changing runtime authorization semantics or CR150 release history.

## Scope

- Statistical gate evidence ref in `StrategyAdmissionPackage` or equivalent admission surface
- CR150 completion map / CR151 linkage helper reference
- Missing-gate linkage gap
- Runtime-readiness hard boundary

## Acceptance Criteria

- A statistical gate ref can be carried through the admission/completion chain.
- Missing statistical gate creates `statistical_admission_gate_missing` or equivalent gap.
- Statistical `PASS` never becomes QMT, broker, simulation, paper, live or trading readiness.
- Existing CR150 static-only release decision remains historically unchanged.

## File Ownership

| File | Intent |
|---|---|
| `engine/mature_multifactor_research.py` | Completion map / linkage integration. |
| `engine/strategy_admission_package.py` | Admission package ref or blocked-reason consumption if needed. |
| `tests/test_cr150_multifactor_framework_completion.py` | Regression for CR150 completion map compatibility. |
| `tests/test_cr151_strategy_admission_statistical_gate.py` | CR151 linkage fixture. |

## Design Evidence Required For CP5

Full LLD covering linkage API, compatibility with `MultifactorFrameworkCompletionMap`, blocked claim mapping, tests and rollback.
