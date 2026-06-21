---
status: "executed"
version: "1.0"
change: "CR-109"
created_at: "2026-06-21T22:26:34+08:00"
created_by: "host-orchestrator"
---

# CR109 Docs Functional Organization Design

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-06-21 | host-orchestrator | 明确 CR108 未完成目录功能化；定义 docs/design、docs/features、docs/quality 的功能化目标形态和迁移规则。 |
| 1.0 | 2026-06-21 | host-orchestrator | 用户批准 CP2 后执行功能化目录迁移；扩展功能域和历史证据归档路径。 |

## Problem

`docs/design`、`docs/features`、`docs/quality` 仍混有大量 CR-number 文件或 CR-number 目录。CR108 修复了 current-truth wording，但没有把文档组织方式从 CR-driven 转成 capability-driven。

## Target Shape

| Layer | Current Problem | Target Organization |
|---|---|---|
| `docs/design` | CR-scoped HLD / ADR / matrix files 与 current index 混排。 | 顶层只保留 current index 和功能化设计入口；CR-scoped 历史设计进入功能域 evidence 或 legacy references。 |
| `docs/features` | 顶层同时有功能目录和 CR 目录，如 `CR093-*`、`cr098-*`、`cr102-*`。 | 顶层只保留功能能力目录；CR-scoped feature evidence 移入对应功能域下 `evidence/by-cr/<CR-id>/`。 |
| `docs/quality` | 大量 `REVIEW-CRxxx` / `TEST-REPORT-CRxxx` / `VERIFICATION-REPORT-CRxxx` 平铺。 | 顶层保留 `README.md` 和 current guardrails；历史质量证据按功能域归档到 `docs/quality/by-feature/<feature>/by-cr/<CR-id>/` 或由功能目录引用。 |

## Functional Domains

| Domain | Canonical Feature Dir | Design Inputs | Quality Evidence Target |
|---|---|---|---|
| market-data-lake | `docs/features/market-data-lake/` | data lake HLD/ADR/matrix refs | `docs/quality/by-feature/market-data-lake/by-cr/` |
| factor-research-loop | `docs/features/factor-research-loop/` | factor research HLD/ADR refs | `docs/quality/by-feature/factor-research-loop/by-cr/` |
| execution-semantics-reference | `docs/features/execution-semantics-reference/` | execution semantics refs | `docs/quality/by-feature/execution-semantics-reference/by-cr/` |
| qmt-gateway-readonly | `docs/features/qmt-gateway-readonly/` | QMT gateway readonly refs | `docs/quality/by-feature/qmt-gateway-readonly/by-cr/` |
| qmt-trading-governance | `docs/features/qmt-trading-governance/` | trading governance refs | `docs/quality/by-feature/qmt-trading-governance/by-cr/` |
| runtime-authorization-safety | `docs/features/runtime-authorization-safety/` | authorization / no-real-operation refs | `docs/quality/by-feature/runtime-authorization-safety/by-cr/` |
| qmt-miniqmt-dual-target-framework | `docs/features/qmt-miniqmt-dual-target-framework/` | CR046 / CR098..CR104 strategy runner refs | `docs/quality/by-feature/qmt-miniqmt-dual-target-framework/by-cr/` |
| strategy-research-lifecycle | `docs/features/strategy-research-lifecycle/` | CR051 lifecycle / migration refs | `docs/quality/by-feature/strategy-research-lifecycle/by-cr/` |
| quant-lab-migration-dry-run | `docs/features/quant-lab-migration-dry-run/` | CR053 / CR058 / CR060 / CR071 migration refs | `docs/quality/by-feature/quant-lab-migration-dry-run/by-cr/` |
| process-ledger-tooling-hygiene | `docs/features/process-ledger-tooling-hygiene/` | CR093..CR095 process / checker refs | `docs/quality/by-feature/process-ledger-tooling-hygiene/by-cr/` |
| package-identity-governance | `docs/features/package-identity-governance/` | package identity / legacy alias refs | `docs/quality/by-feature/package-identity-governance/by-cr/` |
| docs-functional-organization | `docs/features/docs-functional-organization/` | docs structure and migration refs | N/A unless future quality evidence is produced. |

## Migration Rules

| Rule | Meaning |
|---|---|
| current index stays current | `BLUEPRINT.md`, `DOMAIN-MAP.md`, `DEPENDENCY-MAP.md`, `HLD.md`, `ARCHITECTURE-DECISION.md`, `FEATURE-DESIGN-MATRIX.md`, `PACKAGE-IDENTITY.yaml` stay as current design entrypoints unless replaced by explicit functional indexes. |
| CR-scoped docs move under function | CR-specific dirs/files should no longer live at top level; move to the owning function domain under `evidence/by-cr/` or `by-feature/<feature>/by-cr/`. |
| do not rewrite history | Historical CR text remains intact after move; update indexes and references, not historical claims. |
| preserve audit trail | Each move must be recorded in a migration map with old path, new path, function domain, and reason. |
| no business code | This CR only changes docs and process artifacts. |

## Initial Migration Map

| Old Path Pattern | Target Domain | Proposed Target |
|---|---|---|
| `docs/features/CR093-ledger-hygiene/**` | process-ledger-tooling-hygiene | `docs/features/process-ledger-tooling-hygiene/evidence/by-cr/CR093-ledger-hygiene/` |
| `docs/features/CR094-warning-cleanup/**` | process-ledger-tooling-hygiene | `docs/features/process-ledger-tooling-hygiene/evidence/by-cr/CR094-warning-cleanup/` |
| `docs/features/CR095-standalone-checker-cli-output-convergence/**` | process-ledger-tooling-hygiene | `docs/features/process-ledger-tooling-hygiene/evidence/by-cr/CR095-standalone-checker-cli-output-convergence/` |
| `docs/features/cr098-runner-readonly-integration/**` | qmt-miniqmt-dual-target-framework | `docs/features/qmt-miniqmt-dual-target-framework/evidence/by-cr/cr098-runner-readonly-integration/` |
| `docs/features/cr099-runner-real-readonly-smoke/**` | qmt-miniqmt-dual-target-framework | `docs/features/qmt-miniqmt-dual-target-framework/evidence/by-cr/cr099-runner-real-readonly-smoke/` |
| `docs/features/cr102-nas-real-package-exchange-authorization/**` | qmt-miniqmt-dual-target-framework | `docs/features/qmt-miniqmt-dual-target-framework/evidence/by-cr/cr102-nas-real-package-exchange-authorization/` |
| `docs/features/cr103-qmt-miniqmt-non-trading-day-validation/**` | qmt-miniqmt-dual-target-framework | `docs/features/qmt-miniqmt-dual-target-framework/evidence/by-cr/cr103-qmt-miniqmt-non-trading-day-validation/` |
| `docs/features/cr104-qmt-miniqmt-trading-day-validation/**` | qmt-miniqmt-dual-target-framework | `docs/features/qmt-miniqmt-dual-target-framework/evidence/by-cr/cr104-qmt-miniqmt-trading-day-validation/` |
| `docs/features/cr045-goldminer-bridge/**` | qmt-gateway-readonly | `docs/features/qmt-gateway-readonly/evidence/by-cr/cr045-goldminer-bridge/` |
| `docs/features/cr061-package-import-layout/**` | strategy-research-lifecycle | `docs/features/strategy-research-lifecycle/evidence/by-cr/cr061-package-import-layout/` |

## Open Items

| ID | Question | Recommendation |
|---|---|---|
| CR109-O1 | 是否允许物理移动 CR-scoped docs 文件？ | resolved-by-user；已批准并执行。 |
| CR109-O2 | `docs/design/*-CR*.md` 是否移动？ | resolved-by-user；已迁入 `docs/features/<domain>/design/by-cr/`。 |
| CR109-O3 | `docs/quality/*CR*.md` 是否移动？ | resolved-by-user；已迁入 `docs/quality/by-feature/<domain>/by-cr/`。 |
