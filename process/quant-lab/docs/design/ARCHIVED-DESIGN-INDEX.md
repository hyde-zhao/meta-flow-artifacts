---
title: "Archived Design Index"
status: "current-index"
version: "1.0"
source_cr: "CR-131"
archive_root: "process/archive/design-cr-docs"
---

# Archived Design Index

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-23 | host-orchestrator | CR131 新增默认 design root 的归档入口，指向 CR 命名历史设计文档归档区。 |
| 1.1 | 2026-07-12 | host-orchestrator-inline | CR165 将 32 份非 canonical 设计快照迁入归档区；当前根目录只保留允许的 authority 文件。 |

## 入口

CR 命名历史 HLD、ADR、Feature Matrix 和治理设计说明已从默认 design root 移至：

- `process/archive/design-cr-docs/`
- `process/archive/design-cr-docs/ARCHIVE-INDEX.md`

默认 design root 只保留当前权威设计入口。需要审计历史 CR 设计时，从上述归档区读取；不得把归档文档当作当前默认 authority，除非当前 index 明确引用。

## 不授权边界

本索引不授权 runtime、NAS、QMT、凭据、provider、lake、catalog、trading 或 Git remote write。

## CR-165 归档迁移（2026-07-12）

32 份历史 CR / 专题设计快照已从 `process/docs/design/` 移至 `process/archive/design-cr-docs/`，文件名保持不变。当前设计权威入口仅保留根目录中的 `HLD.md`、`ARCHITECTURE-DECISION.md`、`BLUEPRINT.md`、`DOMAIN-MAP.md`、`DEPENDENCY-MAP.md` 和 `FEATURE-DESIGN-MATRIX.md`（及其允许的 supporting files）。

| 归档类别 | 文件模式 | 保留理由 |
|---|---|---|
| HLD | `HLD-*.md`（除 `HLD.md`） | 历史 CR / 专题架构快照。 |
| ADR | `ARCHITECTURE-DECISION-*.md`（除 `ARCHITECTURE-DECISION.md`） | 历史决策记录，不再充当默认 authority。 |
| Blueprint / Domain / Dependency | `BLUEPRINT-*`、`DOMAIN-MAP-*`、`DEPENDENCY-MAP-*` | 变更期领域和依赖快照。 |
| Roadmap / review | `QUANT-RESEARCH-PRODUCTION-ROADMAP.md`、`STRATEGY-*.md` | 历史评估与路线材料。 |
