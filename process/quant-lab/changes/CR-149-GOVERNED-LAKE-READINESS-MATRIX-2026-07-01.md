---
id: "CR-149"
title: "Governed Lake Readiness Matrix Foundation"
status: "closed-current-delivery-nas-sync-deferred"
kind: "requirement-change"
lifecycle_status: "closed"
readiness_status: "ready_with_risk"
gate_status: "closed"
ra_cr149_001_status: "sync_executed_strict_metadata_blocked"
ra_cr149_001_content_consistency_basis: "rsync_checksum_18_of_18_same"
ra_cr149_001_metadata_status: "permission_group_itemization_blocked_18_of_18"
ra_cr149_001_follow_up_decision: "DQ-RA-CR149-POSTSYNC-METADATA-01"
ra_cr149_001_follow_up_tracking: "process/changes/CR-149-FOLLOW-UP-TRACKING-2026-07-01.md"
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

CR-149 is closed-current-delivery with NAS sync deferred by user decision on 2026-07-01. Phase A-G no-risk scope is implemented and recorded in CP6 evidence. The approved read-only NAS/shared-node consistency check executed on 2026-07-01: mounted path was unavailable, credential fallback was audited, evidence redaction scan passed, and no sync/write/restore/delete was executed. The check found the NAS/shared-node view stale relative to local N1 current truth: catalog checksum mismatch plus 17 missing `canonical/*/1.0/current/` paths. NAS current-truth sync is tracked as `RA-CR149-001`; it no longer blocks local no-risk roadmap work. A human gate is still required before any NAS write/sync, provider/runtime operation, catalog pointer mutation, historical conflict cleanup, simulation/live/trading or broker action.

Post-CR150 update: after CR150 CP8 approval at 2026-07-01T20:18:16+08:00, user stated the current PC can connect data lake and NAS and requested the previously unsynced NAS CR be added back into the next plan. `RA-CR149-001` is therefore ready for CP2 review from `process/checkpoints/CP2-CR149-NAS-CURRENT-TRUTH-SYNC.md`; this update does not authorize NAS dry-run or sync execution.

RA-CR149-001 execution update: user approved the CP2 sync gate at 2026-07-01T20:55:35+08:00. The scoped dry-run and scoped local-to-NAS current-truth sync both completed within the approved 18-object scope, with redaction checks passing and no delete/restore/pull/provider/catalog pointer mutation. The required post-sync read-only consistency check remains `BLOCKED`: all 18 comparisons returned rsync metadata itemization mismatches (`.f...p.g...`) with 0 errors. Per DQ-03, no automatic retry, delete, rebuild or metadata normalization was executed. Follow-up decision `DQ-RA-CR149-POSTSYNC-METADATA-01` is required.

RA-CR149-001 semantics update: no raw evidence was overwritten. The rsync-daemon-only interpretation layer records content consistency as `PASS` by rsync `--checksum` internal equality for 18/18 objects, while filesystem metadata consistency remains `BLOCKED` because permission/group itemization differs for 18/18 objects. The current access path has no usable mount and no authorized shell/pull path, so independent NAS-side sha256 and direct reader checks are deferred to a separate access authorization gate. NAS p/g parity is tracked as `FU-CR149-002`; it is not silently removed from CR-149 scope.

RA-CR149-001 disposition prep update: local source-of-truth evidence for the approved 18 objects is now pinned in `process/evidence/CR149-LOCAL-SOURCE-TRUTH-MANIFEST-2026-07-01.json` with local path, size, sha256, mode and uid/gid metadata. `process/checks/RA-CR149-001-POSTSYNC-METADATA-DISPOSITION-PREP-2026-07-01.md` records future closure options without launching a human gate or authorizing NAS access, credential read, mount, shell, pull, retry, chmod/chgrp, metadata normalization, checker source-code change or Phase 1 definition-of-done change.

## Runtime Evidence Update

| Date | Evidence | Result | Notes |
|---|---|---|---|
| 2026-07-01 | `process/evidence/CR149-NAS-MULTINODE-CONSISTENCY.index.json` | BLOCKED | Read-only rsync checksum dry-run compared 18 objects: catalog mismatch and 17 NAS current canonical paths missing. |
| 2026-07-01 | `process/checkpoints/CP2-CR149-NAS-CURRENT-TRUTH-SYNC.md` | DEFERRED | User chose to postpone NAS sync and proceed with multifactor framework completion first. |
| 2026-07-01 | `process/plans/POST-CR150-NEXT-PLAN-2026-07-01.md` | READY_FOR_CP2_REVIEW | After CR150 CP8 approval, user requested adding the previously unsynced NAS CR back into the next plan. |
| 2026-07-01 | `process/checks/RA-CR149-001-NAS-CURRENT-TRUTH-SYNC-EXECUTION-2026-07-01.md` | BLOCKED_POST_SYNC_METADATA_MISMATCH | CP2-approved scoped sync executed; strict post-sync consistency remains blocked by metadata itemization differences. |
| 2026-07-01 | `process/evidence/CR149-NAS-MULTINODE-CONSISTENCY-POST-SYNC-SEMANTICS-2026-07-01.json` | CONTENT_PASS_METADATA_BLOCKED | Interpretation layer only: rsync `--checksum` shows content equality for 18/18; p/g metadata parity remains blocked; no raw evidence mutation or NAS operation. |
| 2026-07-01 | `process/evidence/CR149-LOCAL-SOURCE-TRUTH-MANIFEST-2026-07-01.json` | PASS | Local 18-object source-of-truth manifest records sha256, size, mode and uid/gid for the approved sync objects; no NAS access. |
| 2026-07-01 | `process/checks/RA-CR149-001-POSTSYNC-METADATA-DISPOSITION-PREP-2026-07-01.md` | PASS_WITH_STRICT_EXIT_BLOCKED | Decision prep complete without launching a human gate; strict metadata exit remains blocked. |
