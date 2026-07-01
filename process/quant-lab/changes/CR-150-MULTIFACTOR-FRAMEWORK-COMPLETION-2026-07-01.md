---
id: "CR-150"
title: "Multifactor Framework Completion"
status: "active-no-risk-implementation"
kind: "requirement-change"
lifecycle_status: "active"
readiness_status: "ready"
gate_status: "not_started"
gate_profile: "standard"
created_at: "2026-07-01T15:20:00+08:00"
created_by: "host-orchestrator"
source_tracking: "docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP-2026-07-01.md#phase-3统一回测与实验框架"
source_decision_id: "USER-20260701-DEFER-NAS-SYNC-PROCEED-MULTIFACTOR"
parent_cr: "ROADMAP-QUANT-RESEARCH-PRODUCTION"
impact_level: "medium"
workflow_mode_after_change: "standard"
rollback_to: "pre-CR150 code and artifact state"
not_authorized:
  - "provider_fetch"
  - "nas_sync_or_write"
  - "credential_read"
  - "qmt_miniqmt_xtquant_gateway_runtime"
  - "simulation_live_or_trading"
  - "broker_write"
  - "git_remote_write"
  - "real_lake_write"
  - "catalog_pointer_mutation"
---

# CR-150: Multifactor Framework Completion

## 背景

用户决定将 CR149 NAS current-truth sync 后置，先推进多因子框架补齐。CR-148 已完成统一回测与实验框架 foundation：asset map gap=0、BacktestRunSpec、report pack、cost/risk/attribution metadata contract 均已闭环。下一步不应重建回测引擎，而应补齐既有 `engine/mature_multifactor_framework.py`、`engine/mature_multifactor_research.py`、`engine/backtest.py` 和 admission/report contracts 之间的多因子研究生产链路。

## 目标

在不触发 runtime、真实 lake write、NAS sync、provider fetch、simulation/live/trading 或 broker write 的前提下，补齐日频多因子 baseline 的本地研究生产框架契约，让 factor spec、signal set、portfolio construction、backtest run spec、report pack 和 strategy admission 具备单一可追溯入口。

## In Scope

1. 审计既有多因子资产：factor specs、factor panel、composite score、target portfolio、backtest spec、report pack、strategy admission。
2. 定义或收敛机器可读 `MultifactorFrameworkCompletionMap`，明确必备链路和缺口。
3. 补齐本地 metadata-only contract，覆盖 factor weighting、signal set、portfolio construction、backtest linkage、admission linkage 和 forbidden operation counters。
4. 增加 fixture/unit tests，证明日频多因子 baseline 在本地 deterministic、PIT-aware metadata refs 完整、无 forbidden operation。
5. 生成 CP6/CP7 evidence 和后续 runtime/live/trading/NAS gate 分流。

## Out of Scope

1. 真实 lake 写入、catalog pointer mutation、provider fetch、NAS sync/write/restore drill、credential read。
2. QMT/MiniQMT/xtquant/gateway runtime、broker write、simulation/live/trading。
3. 真实策略晋级、真实下单、小额实盘、live event trading。
4. 分钟级、Level2、Qlib / Backtrader 深度集成。
5. ML / event-driven baseline；Phase 3 exit 仍只要求日频多因子 baseline。

## Execution Plan

| Phase | 目标 | Exit Criteria |
|---|---|---|
| Phase A | Multifactor asset audit | Machine-readable map covers existing mature multifactor framework, research runner, backtest, report and admission assets. |
| Phase B | Contract completion | Metadata-only contract links factor specs → signal set → portfolio → backtest spec → admission refs. |
| Phase C | Fixture validation | Tests prove deterministic hashes/refs, operation counters zero and strategy family remains `multifactor`. |
| Phase D | Evidence and routing | CP result, evidence index and follow-up runtime/live/trading gates recorded. |

## Risk Rules

| Risk | Rule |
|---|---|
| runtime / simulation / live / broker | Not authorized; any runtime execution requires a separate human gate. |
| provider / NAS / credential / lake write | Not authorized; local metadata and fixture tests only. |
| strategy promotion | Admission linkage can be represented, but no strategy is promoted to paper/live. |
| scope creep | ML and event-driven strategy baselines remain deferred; CR150 only closes daily multifactor baseline chain. |

## Current Status

CR-150 is active for no-risk local implementation. No human gate is required for local metadata contracts, tests and process evidence. Any real lake write, NAS/provider/runtime operation, catalog pointer mutation, report overwrite, simulation/live/trading or broker action remains not authorized and requires a separate human gate.
