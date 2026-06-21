---
status: "executed"
change: "CR-109"
created_at: "2026-06-21T22:55:00+08:00"
created_by: "host-orchestrator"
---

# CR109 Docs Functional Migration Map

## Scope

This map records the physical documentation moves authorized by CP2 for CR109. The migration did not change business code, credentials, runtime, trading behavior, production writes, CR-033, or CR102-CR104 lifecycle status.

## Feature Directory Moves

| Old Path | New Path | Functional Domain |
|---|---|---|
| `docs/features/CR093-ledger-hygiene/` | `docs/features/process-ledger-tooling-hygiene/evidence/by-cr/CR093-ledger-hygiene/` | process-ledger-tooling-hygiene |
| `docs/features/CR094-warning-cleanup/` | `docs/features/process-ledger-tooling-hygiene/evidence/by-cr/CR094-warning-cleanup/` | process-ledger-tooling-hygiene |
| `docs/features/CR095-standalone-checker-cli-output-convergence/` | `docs/features/process-ledger-tooling-hygiene/evidence/by-cr/CR095-standalone-checker-cli-output-convergence/` | process-ledger-tooling-hygiene |
| `docs/features/cr045-goldminer-bridge/` | `docs/features/qmt-gateway-readonly/evidence/by-cr/cr045-goldminer-bridge/` | qmt-gateway-readonly |
| `docs/features/cr061-package-import-layout/` | `docs/features/strategy-research-lifecycle/evidence/by-cr/cr061-package-import-layout/` | strategy-research-lifecycle |
| `docs/features/cr098-runner-readonly-integration/` | `docs/features/qmt-miniqmt-dual-target-framework/evidence/by-cr/cr098-runner-readonly-integration/` | qmt-miniqmt-dual-target-framework |
| `docs/features/cr099-runner-real-readonly-smoke/` | `docs/features/qmt-miniqmt-dual-target-framework/evidence/by-cr/cr099-runner-real-readonly-smoke/` | qmt-miniqmt-dual-target-framework |
| `docs/features/cr102-nas-real-package-exchange-authorization/` | `docs/features/qmt-miniqmt-dual-target-framework/evidence/by-cr/cr102-nas-real-package-exchange-authorization/` | qmt-miniqmt-dual-target-framework |
| `docs/features/cr103-qmt-miniqmt-non-trading-day-validation/` | `docs/features/qmt-miniqmt-dual-target-framework/evidence/by-cr/cr103-qmt-miniqmt-non-trading-day-validation/` | qmt-miniqmt-dual-target-framework |
| `docs/features/cr104-qmt-miniqmt-trading-day-validation/` | `docs/features/qmt-miniqmt-dual-target-framework/evidence/by-cr/cr104-qmt-miniqmt-trading-day-validation/` | qmt-miniqmt-dual-target-framework |

## Design File Moves

| Old Path | New Path | Functional Domain |
|---|---|---|
| `docs/features/runtime-authorization-safety/design/by-cr/CR044/FEATURE-DESIGN-MATRIX-CR044.md` | `docs/features/runtime-authorization-safety/design/by-cr/CR044/FEATURE-DESIGN-MATRIX-CR044.md` | runtime-authorization-safety |
| `docs/features/qmt-gateway-readonly/design/by-cr/CR045/ARCHITECTURE-DECISION-CR045.md` | `docs/features/qmt-gateway-readonly/design/by-cr/CR045/ARCHITECTURE-DECISION-CR045.md` | qmt-gateway-readonly |
| `docs/features/qmt-gateway-readonly/design/by-cr/CR045/HLD-CR045-GOLDMINER-WINDOWS-BRIDGE.md` | `docs/features/qmt-gateway-readonly/design/by-cr/CR045/HLD-CR045-GOLDMINER-WINDOWS-BRIDGE.md` | qmt-gateway-readonly |
| `docs/features/qmt-gateway-readonly/design/by-cr/CR045/FEATURE-DESIGN-MATRIX-CR045.md` | `docs/features/qmt-gateway-readonly/design/by-cr/CR045/FEATURE-DESIGN-MATRIX-CR045.md` | qmt-gateway-readonly |
| `docs/features/qmt-miniqmt-dual-target-framework/design/by-cr/CR046/ARCHITECTURE-DECISION-CR046.md` | `docs/features/qmt-miniqmt-dual-target-framework/design/by-cr/CR046/ARCHITECTURE-DECISION-CR046.md` | qmt-miniqmt-dual-target-framework |
| `docs/features/qmt-miniqmt-dual-target-framework/design/by-cr/CR046/HLD-CR046-QMT-MINIQMT-DUAL-TARGET-FRAMEWORK.md` | `docs/features/qmt-miniqmt-dual-target-framework/design/by-cr/CR046/HLD-CR046-QMT-MINIQMT-DUAL-TARGET-FRAMEWORK.md` | qmt-miniqmt-dual-target-framework |
| `docs/features/strategy-research-lifecycle/design/by-cr/CR051/HLD-CR051-STRATEGY-RESEARCH-LIFECYCLE-FRAMEWORK.md` | `docs/features/strategy-research-lifecycle/design/by-cr/CR051/HLD-CR051-STRATEGY-RESEARCH-LIFECYCLE-FRAMEWORK.md` | strategy-research-lifecycle |
| `docs/features/quant-lab-migration-dry-run/design/by-cr/CR053/ARCHITECTURE-DECISION-CR053.md` | `docs/features/quant-lab-migration-dry-run/design/by-cr/CR053/ARCHITECTURE-DECISION-CR053.md` | quant-lab-migration-dry-run |
| `docs/features/quant-lab-migration-dry-run/design/by-cr/CR053/HLD-CR053-QUANT-LAB-MIGRATION-INVENTORY-AND-DRY-RUN.md` | `docs/features/quant-lab-migration-dry-run/design/by-cr/CR053/HLD-CR053-QUANT-LAB-MIGRATION-INVENTORY-AND-DRY-RUN.md` | quant-lab-migration-dry-run |
| `docs/features/quant-lab-migration-dry-run/design/by-cr/CR058/ARCHITECTURE-DECISION-CR058.md` | `docs/features/quant-lab-migration-dry-run/design/by-cr/CR058/ARCHITECTURE-DECISION-CR058.md` | quant-lab-migration-dry-run |
| `docs/features/quant-lab-migration-dry-run/design/by-cr/CR058/HLD-CR058-REPO-LOCAL-MECHANICAL-MIGRATION-RELAYOUT-GATE.md` | `docs/features/quant-lab-migration-dry-run/design/by-cr/CR058/HLD-CR058-REPO-LOCAL-MECHANICAL-MIGRATION-RELAYOUT-GATE.md` | quant-lab-migration-dry-run |
| `docs/features/quant-lab-migration-dry-run/design/by-cr/CR058/FEATURE-DESIGN-MATRIX-CR058.md` | `docs/features/quant-lab-migration-dry-run/design/by-cr/CR058/FEATURE-DESIGN-MATRIX-CR058.md` | quant-lab-migration-dry-run |
| `docs/features/strategy-research-lifecycle/design/by-cr/CR061/FEATURE-DESIGN-MATRIX-CR061.md` | `docs/features/strategy-research-lifecycle/design/by-cr/CR061/FEATURE-DESIGN-MATRIX-CR061.md` | strategy-research-lifecycle |
| `docs/features/process-ledger-tooling-hygiene/design/by-cr/CR093/HLD-CR093-LEDGER-HYGIENE.md` | `docs/features/process-ledger-tooling-hygiene/design/by-cr/CR093/HLD-CR093-LEDGER-HYGIENE.md` | process-ledger-tooling-hygiene |
| `docs/features/process-ledger-tooling-hygiene/design/by-cr/CR093/FEATURE-DESIGN-MATRIX-CR093.md` | `docs/features/process-ledger-tooling-hygiene/design/by-cr/CR093/FEATURE-DESIGN-MATRIX-CR093.md` | process-ledger-tooling-hygiene |
| `docs/features/process-ledger-tooling-hygiene/design/by-cr/CR107/HLD-CR107-QUANT-LAB-REMEDIATION-DESIGN.md` | `docs/features/process-ledger-tooling-hygiene/design/by-cr/CR107/HLD-CR107-QUANT-LAB-REMEDIATION-DESIGN.md` | process-ledger-tooling-hygiene |
| `docs/features/docs-functional-organization/design/by-cr/CR109/DOCS-FUNCTIONAL-ORGANIZATION.md` | `docs/features/docs-functional-organization/design/by-cr/CR109/DOCS-FUNCTIONAL-ORGANIZATION.md` | docs-functional-organization |

## Quality Evidence Moves

| CR | Old Pattern | New Path | Functional Domain |
|---|---|---|---|
| CR044 | `docs/quality/*CR044.md` | `docs/quality/by-feature/runtime-authorization-safety/by-cr/CR044/` | runtime-authorization-safety |
| CR045 | `docs/quality/*CR045.md` | `docs/quality/by-feature/qmt-gateway-readonly/by-cr/CR045/` | qmt-gateway-readonly |
| CR046 | `docs/quality/*CR046.md` | `docs/quality/by-feature/qmt-miniqmt-dual-target-framework/by-cr/CR046/` | qmt-miniqmt-dual-target-framework |
| CR051 | `docs/quality/*CR051.md` | `docs/quality/by-feature/strategy-research-lifecycle/by-cr/CR051/` | strategy-research-lifecycle |
| CR053 | `docs/quality/*CR053.md` | `docs/quality/by-feature/quant-lab-migration-dry-run/by-cr/CR053/` | quant-lab-migration-dry-run |
| CR058 | `docs/quality/*CR058.md` | `docs/quality/by-feature/quant-lab-migration-dry-run/by-cr/CR058/` | quant-lab-migration-dry-run |
| CR060 | `docs/quality/*CR060.md` | `docs/quality/by-feature/package-identity-governance/by-cr/CR060/` | package-identity-governance |
| CR067 | `docs/quality/*CR067.md` | `docs/quality/by-feature/quant-lab-migration-dry-run/by-cr/CR067/` | quant-lab-migration-dry-run |
| CR068 | `docs/quality/*CR068.md` | `docs/quality/by-feature/quant-lab-migration-dry-run/by-cr/CR068/` | quant-lab-migration-dry-run |
| CR069 | `docs/quality/*CR069.md` | `docs/quality/by-feature/quant-lab-migration-dry-run/by-cr/CR069/` | quant-lab-migration-dry-run |
| CR070 | `docs/quality/*CR070.md` | `docs/quality/by-feature/quant-lab-migration-dry-run/by-cr/CR070/` | quant-lab-migration-dry-run |
| CR071 | `docs/quality/*CR071.md` | `docs/quality/by-feature/quant-lab-migration-dry-run/by-cr/CR071/` | quant-lab-migration-dry-run |
| CR072 | `docs/quality/*CR072.md` | `docs/quality/by-feature/quant-lab-migration-dry-run/by-cr/CR072/` | quant-lab-migration-dry-run |
| CR080 | `docs/quality/*CR080.md` | `docs/quality/by-feature/quant-lab-migration-dry-run/by-cr/CR080/` | quant-lab-migration-dry-run |
| CR081 | `docs/quality/*CR081.md` | `docs/quality/by-feature/process-ledger-tooling-hygiene/by-cr/CR081/` | process-ledger-tooling-hygiene |
| CR082 | `docs/quality/*CR082.md` | `docs/quality/by-feature/process-ledger-tooling-hygiene/by-cr/CR082/` | process-ledger-tooling-hygiene |
| CR091 | `docs/quality/CR091*.md` | `docs/quality/by-feature/qmt-miniqmt-dual-target-framework/by-cr/CR091/` | qmt-miniqmt-dual-target-framework |
| CR092 | `docs/quality/CR092*.md` | `docs/quality/by-feature/qmt-miniqmt-dual-target-framework/by-cr/CR092/` | qmt-miniqmt-dual-target-framework |
| CR100 | `docs/quality/CR100*.md` | `docs/quality/by-feature/qmt-miniqmt-dual-target-framework/by-cr/CR100/` | qmt-miniqmt-dual-target-framework |
