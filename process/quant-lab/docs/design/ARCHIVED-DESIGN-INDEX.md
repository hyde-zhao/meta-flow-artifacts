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

## 入口

CR 命名历史 HLD、ADR、Feature Matrix 和治理设计说明已从默认 design root 移至：

- `process/archive/design-cr-docs/`
- `process/archive/design-cr-docs/ARCHIVE-INDEX.md`

默认 design root 只保留当前权威设计入口。需要审计历史 CR 设计时，从上述归档区读取；不得把归档文档当作当前默认 authority，除非当前 index 明确引用。

## 不授权边界

本索引不授权 runtime、NAS、QMT、凭据、provider、lake、catalog、trading 或 Git remote write。
