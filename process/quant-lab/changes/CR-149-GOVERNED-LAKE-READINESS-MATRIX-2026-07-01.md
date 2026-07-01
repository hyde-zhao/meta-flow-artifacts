---
id: "CR-149"
title: "Governed Lake Readiness Matrix Foundation"
status: "active-cp2-nas-current-truth-sync-gate-pending"
kind: "requirement-change"
lifecycle_status: "active"
readiness_status: "ready_with_risk"
gate_status: "cp2_pending"
gate_profile: "standard"
created_at: "2026-07-01T13:25:00+08:00"
created_by: "host-orchestrator"
source_tracking: "docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP-2026-07-01.md#phase-1生产级数据湖治理"
source_decision_id: "USER-20260701-CONTINUE-PRODUCTION-ROADMAP"
parent_cr: "ROADMAP-QUANT-RESEARCH-PRODUCTION"
impact_level: "medium"
workflow_mode_after_change: "standard"
rollback_to: "pre-CR149 code and artifact state"
not_authorized:
  - "provider_fetch"
  - "nas_sync_or_write_without_cp2_sync_approval"
  - "credential_read_without_human_gate"
  - "qmt_miniqmt_xtquant_gateway_runtime"
  - "simulation_live_or_trading"
  - "broker_write"
  - "git_remote_write"
  - "real_lake_write"
  - "catalog_pointer_mutation"
  - "historical_business_conflict_semantic_cleanup"
---

# CR-149: Governed Lake Readiness Matrix Foundation

## 背景

CR-146 已完成 current-truth 数据湖迁移、fail-closed reader 加固和 runtime validation。CR-147/CR-148 已完成研究数据集与回测框架的 metadata-only foundation。路线图 Phase 1 仍要求把 current truth 从“可读”推进为可治理的数据湖：17/17 dataset readiness、PIT 状态、run ordering / registry、quarantine 和 recurring validation 必须有机器可读入口。

## 目标

在不执行真实 lake 写入、不改 catalog pointer、不清理历史 business-conflict 的前提下，收敛现有 catalog/readiness 能力，新增 17-dataset governed lake readiness matrix 合约。

## In Scope

1. 从 `CatalogEntry` / catalog metadata 构建 17-dataset readiness/PIT/run-registry matrix。
2. 为每个 dataset 输出 `production_ready / research_ready / quarantined / unsupported` 之一。
3. 输出 PIT status：`pit_available / not_applicable / unsupported-with-reason`，不得长期保留 `null`。
4. 输出 run registry row，替代生产排序场景对 `source_run_id` 字典序 fallback 的依赖。
5. 输出 operation counters，并证明 provider/NAS/lake write/catalog pointer/trading counters 全零。
6. 输出 recurring validation plan，覆盖 inventory、golden baseline、PIT reader smoke、duplicate profile、readiness matrix、published pointer local consistency 和 NAS multi-node consistency gate。
7. 增加 fixture tests 和 CP6 evidence。

## Out of Scope

1. 真实 lake 写入、catalog pointer mutation、provider fetch、NAS sync/write/restore drill or credential read without a human gate.
2. 历史 business-conflict 语义择优、重写、删除或 quarantine 物理迁移。
3. N2 历史路径 rename、legacy archive delete。
4. QMT/MiniQMT/xtquant/gateway runtime、broker write、simulation/live/trading。

## Execution Plan

| Phase | 目标 | Exit Criteria |
|---|---|---|
| Phase A | Governed readiness contract | 17-dataset matrix dataclasses / builder / validator implemented. |
| Phase B | Run registry bridge | Catalog current run metadata can be represented without source_run_id lexical ordering. |
| Phase C | Fixture validation | Tests prove 17/17 coverage, PIT normalization, quarantine classification and zero side effects. |
| Phase D | Evidence and routing | CP result, evidence index and remaining high-risk follow-up routing recorded. |
| Phase E | Recurring validation plan | 7 validation tasks are machine-readable; 6 are local read-only/metadata-only, 1 NAS multi-node task is human-gate-required. |
| Phase F | Business-conflict policy | CR146 split-plan counts are represented as dataset-level policy; 4,272,624 business-conflict groups are classified as full-group quarantine by default. |
| Phase G | Phase 1 exit matrix | 8 Phase 1 criteria are machine-readable; 7 pass and 1 NAS multi-node criterion is blocked-human-gate. |

## Risk Rules

| Risk | Rule |
|---|---|
| business-conflict cleanup | Not authorized; CR-149 may label quarantine policy only, not select survivor rows. |
| provider/NAS/credential/lake write | Not authorized unless an explicit human gate grants a scoped runtime action. |
| catalog pointer mutation | Not authorized; no publish, no pointer switch. |
| runtime/trading/broker | Not authorized; any runtime or broker action requires separate human gate. |

## Current Status

CR-149 is active with CP2 pending for scoped NAS current-truth sync. Phase A-G no-risk scope is implemented and recorded in CP6 evidence. The approved read-only NAS/shared-node consistency check executed on 2026-07-01: mounted path was unavailable, credential fallback was audited, evidence redaction scan passed, and no sync/write/restore/delete was executed. The check found the NAS/shared-node view stale relative to local N1 current truth: catalog checksum mismatch plus 17 missing `canonical/*/1.0/current/` paths. CR149 Phase 1 remains blocked until a separate CP2 gate approves or rejects scoped local-to-NAS current-truth sync. A human gate is required before any NAS write/sync, provider/runtime operation, catalog pointer mutation, historical conflict cleanup, simulation/live/trading or broker action.

## Runtime Evidence Update

| Date | Evidence | Result | Notes |
|---|---|---|---|
| 2026-07-01 | `process/evidence/CR149-NAS-MULTINODE-CONSISTENCY.index.json` | BLOCKED | Read-only rsync checksum dry-run compared 18 objects: catalog mismatch and 17 NAS current canonical paths missing. |
| 2026-07-01 | `process/checkpoints/CP2-CR149-NAS-CURRENT-TRUTH-SYNC.md` | PENDING | New human gate asks whether to sync local N1 current truth to NAS using scoped object list only. |
