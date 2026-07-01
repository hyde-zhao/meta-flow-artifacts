---
id: "CR-147"
title: "Quant Research Production Foundation"
status: "active"
kind: "requirement-change"
lifecycle_status: "active"
readiness_status: "ready_with_risk"
gate_status: "cp2_pending"
gate_profile: "standard"
created_at: "2026-07-01T10:47:00+08:00"
created_by: "host-orchestrator"
source_tracking: "docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP-2026-07-01.md#cr-147"
source_decision_id: "USER-20260701-CONTINUE-PRODUCTION-ROADMAP"
parent_cr: "ROADMAP-QUANT-RESEARCH-PRODUCTION"
impact_level: "medium"
workflow_mode_after_change: "standard"
rollback_to: "pre-CR147 code and artifact state"
not_authorized:
  - "provider_fetch"
  - "nas_sync_or_write"
  - "credential_read"
  - "qmt_miniqmt_xtquant_gateway_runtime"
  - "simulation_live_or_trading"
  - "broker_write"
  - "git_remote_write"
  - "historical_lake_cleanup_or_rewrite"
  - "business_conflict_semantic_selection"
  - "catalog_pointer_mutation"
---

# CR-147: Quant Research Production Foundation

## 背景

CR-146 已完成 CR139 post-close 数据湖生产级 current truth 改造：fail-closed 加固、N1 current layout、catalog current truth 验证、reader smoke、golden attribution 和回归均通过。用户已批准 historical-root cleanup gate 的推荐方案：不立即执行高风险 cleanup，只准备低风险 dataset-level 草案。

路线图 `docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP-2026-07-01.md` v0.2 已将下一步定位为 Phase 2：研究数据集、Feature Store、Label Store、Event Store 和实验注册的生产化底座。评审要求 CR-147 不从零重建，而是收敛既有资产。

## 目标

CR-147 的目标是把现有研究数据集与特征/标签/实验资产收敛为可复现、PIT-safe、可审计的研究生产底座，为后续多因子、机器学习和事件驱动策略共用。

## In Scope

1. 明确 `ResearchDatasetSpec` / `ResearchDatasetRequest` 的生产契约映射。
2. 将 `market_data/features/artifacts.py` 的 `FeatureArtifactSpec` / `LabelArtifactSpec` 纳入 Phase 2 contract。
3. 将 `engine/training_snapshot_contract.py` 与 `engine/research_manifest.py` 纳入 experiment / snapshot registry contract。
4. 建立 no-risk contract audit：PIT-safe consumption、snapshot metadata、feature/label lineage、leakage gate、forbidden operation counters。
5. 增加离线 fixture / unit tests，证明同一 spec 可复跑并得到稳定 metadata/checksum。
6. 生成 CR147 Phase 2 implementation plan 和 evidence index。
7. 保持日频 baseline，不引入 minute / Level2 / Qlib / Backtrader 深度集成。

## Out of Scope

1. 真实 lake 写入、catalog pointer mutation 或 published pointer advance。
2. provider fetch、NAS sync/write/restore drill、credential read。
3. historical-root cleanup/rewrite 或 business-conflict semantic selection。
4. QMT/MiniQMT/xtquant/gateway runtime、broker write、simulation/live/trading。
5. ML production training、event live trading、小额实盘。

## Execution Plan

| Phase | 目标 | Exit Criteria |
|---|---|---|
| Phase A | Existing asset audit | Produce machine-readable map of ResearchDataset / FeatureArtifact / TrainingSnapshot / ExperimentManifest assets and current gaps. |
| Phase B | Contract hardening | Add or refine low-risk dataclass/contracts around ResearchDatasetSpec, Feature/Label artifact refs, leakage policy and snapshot metadata. |
| Phase C | Offline validation | Add fixture tests proving deterministic replay, PIT-safe metadata, no forbidden operation counters and leakage gate behavior. |
| Phase D | Evidence and closure | Write CR147 evidence index, CP result, and route any runtime/high-risk items to follow-up gates. |

## Risk Rules

| Risk | Rule |
|---|---|
| real lake mutation | Not authorized; all CR147 initial work is local code / fixture / process evidence only. |
| provider/NAS/credential/runtime | Not authorized; any need becomes a separate human gate. |
| strategy direction lock-in | Baseline multi-factor can only validate pipeline shape; it does not select a production strategy. |
| data leakage | Any feature/label contract must expose `available_at`, `decision_time` and label window checks. |
| duplicate business-conflict cleanup | Remains FU-CR139-001; not part of CR147 implementation. |

## Current Status

CR-147 is active. Initial implementation should start with no-risk contract audit and fixture tests. No human gate is required for local code / tests / process evidence. A human gate is required before any real lake write, NAS/provider/runtime operation, catalog pointer mutation, business-conflict semantic selection, or trading action.
