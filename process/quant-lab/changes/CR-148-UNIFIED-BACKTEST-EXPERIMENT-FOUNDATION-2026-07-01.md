---
id: "CR-148"
title: "Unified Backtest and Experiment Foundation"
status: "closed-current-delivery"
kind: "requirement-change"
lifecycle_status: "closed"
readiness_status: "ready"
gate_status: "closed"
gate_profile: "standard"
created_at: "2026-07-01T12:22:00+08:00"
created_by: "host-orchestrator"
source_tracking: "docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP-2026-07-01.md#phase-3统一回测与实验框架"
source_decision_id: "USER-20260701-CONTINUE-PRODUCTION-ROADMAP"
parent_cr: "ROADMAP-QUANT-RESEARCH-PRODUCTION"
impact_level: "medium"
workflow_mode_after_change: "standard"
rollback_to: "pre-CR148 code and artifact state"
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

# CR-148: Unified Backtest and Experiment Foundation

## 背景

CR-147 已完成 Phase 2 research production contract foundation：研究数据集 spec、snapshot registry、feature availability gate、asset map 和离线证据均已闭环。路线图下一步是 Phase 3：统一回测与实验框架。

## 目标

CR-148 的目标是在不触发 runtime / live / broker / lake write 的前提下，收敛现有 backtest、multifactor framework、experiment manifest 和 strategy admission 资产，形成统一的离线 backtest / experiment foundation contract。

## In Scope

1. 审计既有 `engine/backtest.py`、`engine/mature_multifactor_framework.py`、`engine/research_manifest.py`、`engine/strategy_admission_package.py` 与相关测试资产。
2. 定义统一 `BacktestRunSpec` / `ExperimentBacktestLink` 等低风险 metadata contract。
3. 建立成本、滑点、可交易性、benchmark、attribution、report pack 的 offline contract map。
4. 增加 fixture / unit tests，证明同一 backtest spec 具有稳定 metadata、manifest refs 和 forbidden operation counters=0。
5. 生成 CP6/CP7 evidence 和后续 runtime/live/trading gate 分流。

## Out of Scope

1. 真实 lake 写入、catalog pointer mutation、provider fetch、NAS sync/write/restore drill、credential read。
2. QMT/MiniQMT/xtquant/gateway runtime、broker write、simulation/live/trading。
3. 真实策略晋级、真实下单、小额实盘或 live event trading。
4. 分钟级、Level2、Qlib / Backtrader 深度集成。

## Execution Plan

| Phase | 目标 | Exit Criteria |
|---|---|---|
| Phase A | Existing backtest asset audit | Machine-readable map of existing backtest / multifactor / manifest / admission assets and gaps. |
| Phase B | Contract hardening | Add low-risk dataclass/contracts for unified backtest run metadata and experiment linkage. |
| Phase C | Offline validation | Fixture tests for deterministic metadata, no forbidden operations and stable manifest/admission refs. |
| Phase D | Evidence and routing | CP results, evidence index and high-risk runtime/live/trading follow-up routing. |

## Risk Rules

| Risk | Rule |
|---|---|
| simulation/live/trading | Not authorized; any runtime execution requires a separate human gate. |
| provider/NAS/credential/lake write | Not authorized; local fixture metadata only. |
| strategy promotion | Admission contracts may be referenced, but no strategy is promoted to paper/live. |
| external framework lock-in | Qlib / Backtrader integration remains deferred unless separately approved. |

## Current Status

CR-148 is closed-current-delivery / READY as of 2026-07-01T13:15:00+08:00.

Closed scope:

1. Phase A asset audit: `BacktestFoundationAssetMap` maps existing backtest, multifactor, manifest, admission, reporting, portfolio and metrics assets.
2. Phase B contract hardening: `BacktestRunSpec` provides deterministic metadata, stable config hash and strict Phase 3 daily-multifactor scope checks.
3. Phase C report pack: `BacktestReportPack` provides stable in-memory report pack rows, artifact refs and content hash without report file writes.
4. Phase D cost/risk/attribution: `BacktestCostRiskAttributionPack` provides cost fields, risk metrics, attribution refs and content hash without runtime execution.
5. Asset-map gaps are closed: `gap_count=0`, `gate_required_gap_count=0`.

Evidence:

- `process/checks/CP6-CR148-BACKTEST-FOUNDATION-ASSET-MAP.result.json`
- `process/checks/CP6-CR148-BACKTEST-RUN-SPEC.result.json`
- `process/checks/CP6-CR148-BACKTEST-REPORT-PACK.result.json`
- `process/checks/CP6-CR148-BACKTEST-COST-RISK-ATTRIBUTION.result.json`
- `process/evidence/CR148-PHASE3-BACKTEST-COST-RISK-ATTRIBUTION.index.json`

Final verification:

- `uv run --python 3.11 pytest tests/research/test_backtest_production_contracts.py`: 10 passed.
- Related regression: 88 passed.
- `meta-flow cp result-check`: OK for all new CP6 result JSON files.
- `meta-flow event check --ledger process/state/CHECKPOINT-LEDGER.ndjson --type checkpoint`: OK.
- `meta-flow check cr-tracking`: OK.
- `git diff --check`: PASS.

No human gate was required for the closed scope because all actions were local metadata contracts, tests and process evidence. Any real lake write, NAS/provider/runtime operation, catalog pointer mutation, report overwrite, simulation/live/trading or broker action remains not authorized and requires a separate human gate.
