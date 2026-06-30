---
title: "Archived CR-Named Design Documents"
status: "current-archive-index"
version: "1.0"
source_cr: "CR-131"
created_at: "2026-06-23T18:05:00+08:00"
owner: "host-orchestrator"
---

# Archived CR-Named Design Documents

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-23 | host-orchestrator | CR131 将默认 design root 中的 CR 命名历史设计文档移入本归档区，并保留完整正文。 |
| 1.1 | 2026-06-30 | host-orchestrator | CR141 继续收敛 CR131 design surface 红基线，将 CR138/CR139 专属 HLD/ADR 从 current design root 归档。 |

## 定位

本目录保存从 `process/docs/design/` 默认 design surface 移出的 CR 命名历史 HLD、ADR、Feature Matrix 和治理设计说明。默认 design root 只保留当前权威入口；需要审计历史 CR 设计时从本目录读取。

本归档不授权 runtime、NAS、QMT、凭据、provider、lake、catalog、trading 或 Git remote write。

## 归档清单

| 文件 | 历史用途 |
|---|---|
| `ARCHITECTURE-DECISION-CR045.md` | CR045 Goldminer Windows bridge ADR |
| `ARCHITECTURE-DECISION-CR046.md` | CR046 QMT / MiniQMT 双目标策略交付 legacy ADR cluster |
| `ARCHITECTURE-DECISION-CR053.md` | CR053 migration inventory / dry-run ADR |
| `ARCHITECTURE-DECISION-CR058.md` | CR058 repo-local mechanical migration ADR |
| `ARCHITECTURE-DECISION-RUNNER-QMT-OPERATIONAL-CONTROL-PLANE.md` | CR138 Runner & QMT Gateway operational control plane ADR |
| `ARCHITECTURE-DECISION-STRATEGY-DATA-FOUNDATION.md` | CR139 Strategy Data Foundation ADR |
| `CONTEXT-CAPSULE-HUMAN-GATE-CONSISTENCY-CR114.md` | CR114 context / human gate consistency design notes |
| `CONTEXT-PATH-ALIAS-NORMALIZATION-NOTES-CR117.md` | CR117 path alias normalization notes |
| `CR-TRACKING-CHECKER-EXPECTATION-NOTES-CR112.md` | CR112 CR tracking checker expectation notes |
| `CR-TRACKING-FORMAL-FU-ROW-CONVENTION-HARDENING-CR120.md` | CR120 formal/FU row convention hardening notes |
| `FEATURE-DESIGN-MATRIX-CR044.md` | CR044 scoped feature design matrix |
| `FEATURE-DESIGN-MATRIX-CR045.md` | CR045 scoped feature design matrix |
| `FEATURE-DESIGN-MATRIX-CR058.md` | CR058 scoped feature design matrix |
| `FEATURE-DESIGN-MATRIX-CR061.md` | CR061 scoped feature design matrix |
| `FEATURE-DESIGN-MATRIX-CR093.md` | CR093 scoped feature design matrix |
| `GOVERNANCE-STATE-CONTRACT-CLEANUP-CR109.md` | CR109 governance / state contract cleanup notes |
| `HLD-CR045-GOLDMINER-WINDOWS-BRIDGE.md` | CR045 Goldminer Windows bridge HLD |
| `HLD-CR046-QMT-MINIQMT-DUAL-TARGET-FRAMEWORK.md` | CR046 QMT / MiniQMT 双目标策略交付 legacy HLD |
| `HLD-CR051-STRATEGY-RESEARCH-LIFECYCLE-FRAMEWORK.md` | CR051 strategy research lifecycle HLD |
| `HLD-CR053-QUANT-LAB-MIGRATION-INVENTORY-AND-DRY-RUN.md` | CR053 migration inventory / dry-run HLD |
| `HLD-CR058-REPO-LOCAL-MECHANICAL-MIGRATION-RELAYOUT-GATE.md` | CR058 repo-local migration HLD |
| `HLD-CR093-LEDGER-HYGIENE.md` | CR093 ledger hygiene HLD |
| `HLD-RUNNER-QMT-OPERATIONAL-CONTROL-PLANE.md` | CR138 Runner & QMT Gateway operational control plane HLD |
| `HLD-STRATEGY-DATA-FOUNDATION.md` | CR139 Strategy Data Foundation HLD |
| `HUMAN-GATE-LAUNCH-MESSAGE-CHECKABLE-DRAFT-CONTRACT-CR115.md` | CR115 human gate launch message draft contract |
| `HUMAN-GATE-LAUNCH-MESSAGE-CHECKER-IMPLEMENTATION-CANDIDATE-CR119.md` | CR119 human gate launch message checker candidate |
| `PATH-ALIAS-CHECKER-ENFORCEMENT-CANDIDATE-CR118.md` | CR118 path alias checker enforcement candidate |
| `PENDING-DECISION-QUEUE-CHECKPOINT-ALIGNMENT-AUDIT-CR116.md` | CR116 pending decision queue audit |
| `REDESIGN-BACKLOG-DECISION-CR108.md` | CR108 redesign backlog decision |
| `REDESIGN-REMAINING-BACKLOG-TRIAGE-CR113.md` | CR113 remaining backlog triage |
| `REDESIGN-SCOPE-INTAKE-CR107.md` | CR107 redesign scope intake |
| `RUNNER-CORE-MVP-DESIGN-CR126.md` | CR126/CR128 runner MVP implementation intake / source design |
| `STATE-CR-TRACKING-CONTRACT-NORMALIZATION-CR110.md` | CR110 state / CR tracking contract normalization |
| `STATE-SUMMARY-STALENESS-CLEANUP-CR111.md` | CR111 state summary staleness cleanup |
