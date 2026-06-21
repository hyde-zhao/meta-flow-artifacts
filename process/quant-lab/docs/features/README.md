---
status: "current-index"
version: "1.0"
change: "CR-109"
created_at: "2026-06-21T22:52:00+08:00"
created_by: "host-orchestrator"
---

# Feature Documentation Index

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-21 | host-orchestrator | 按 CR109 将 `docs/features` 顶层收敛为功能域目录；CR-scoped 历史材料迁入各功能域的 `evidence/by-cr/` 或 `design/by-cr/`。 |

## 当前入口

| 功能域 | 当前设计入口 | 历史 CR 证据 |
|---|---|---|
| market-data-lake | `docs/features/market-data-lake/DESIGN.md` | 按需读取对应历史 CR / process 证据。 |
| factor-research-loop | `docs/features/factor-research-loop/DESIGN.md` | 按需读取对应历史 CR / process 证据。 |
| execution-semantics-reference | `docs/features/execution-semantics-reference/DESIGN.md` | 按需读取对应历史 CR / process 证据。 |
| qmt-gateway-readonly | `docs/features/qmt-gateway-readonly/DESIGN.md` | `docs/features/qmt-gateway-readonly/evidence/by-cr/`、`docs/features/qmt-gateway-readonly/design/by-cr/` |
| qmt-trading-governance | `docs/features/qmt-trading-governance/DESIGN.md` | 按需读取对应历史 CR / process 证据。 |
| runtime-authorization-safety | `docs/features/runtime-authorization-safety/DESIGN.md` | `docs/features/runtime-authorization-safety/design/by-cr/` |
| qmt-miniqmt-dual-target-framework | `docs/features/qmt-miniqmt-dual-target-framework/DESIGN.md` | `docs/features/qmt-miniqmt-dual-target-framework/evidence/by-cr/`、`docs/features/qmt-miniqmt-dual-target-framework/design/by-cr/` |
| strategy-research-lifecycle | `docs/features/strategy-research-lifecycle/DESIGN.md` | `docs/features/strategy-research-lifecycle/evidence/by-cr/`、`docs/features/strategy-research-lifecycle/design/by-cr/` |
| quant-lab-migration-dry-run | `docs/features/quant-lab-migration-dry-run/DESIGN.md` | `docs/features/quant-lab-migration-dry-run/design/by-cr/` |
| package-identity-governance | `docs/features/package-identity-governance/DESIGN.md` | package identity historical quality evidence is under `docs/quality/by-feature/package-identity-governance/by-cr/`. |
| process-ledger-tooling-hygiene | `docs/features/process-ledger-tooling-hygiene/DESIGN.md` | `docs/features/process-ledger-tooling-hygiene/evidence/by-cr/`、`docs/features/process-ledger-tooling-hygiene/design/by-cr/` |
| docs-functional-organization | `docs/features/docs-functional-organization/DESIGN.md` | `docs/features/docs-functional-organization/design/by-cr/CR109/` |

## 读取规则

1. 默认先读功能域顶层的 `DESIGN.md`、`TEST-PLAN.md`、`TASKS.md`。
2. 只有需要审计某个历史 CR 的设计或验证事实时，才展开 `design/by-cr/` 或 `evidence/by-cr/`。
3. 历史 CR 证据只证明对应 CR 当时的范围、授权和结果，不自动授权 runtime、凭据、真实交易、production write 或 CR102-CR104 关闭。
