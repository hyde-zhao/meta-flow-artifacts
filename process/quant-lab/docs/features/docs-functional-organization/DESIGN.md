---
status: "current"
version: "1.0"
change: "CR-109"
created_at: "2026-06-21T22:52:00+08:00"
created_by: "host-orchestrator"
---

# Feature Design: Docs Functional Organization

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-21 | host-orchestrator | 将 docs/design、docs/features、docs/quality 的 CR-number 组织方式收敛为功能域组织方式。 |

## 目标

`docs/design`、`docs/features` 和 `docs/quality` 的当前读取入口必须按功能能力组织；历史 CR 材料保留为审计证据，但不再占用顶层目录。

## 边界

| 类型 | 内容 |
|---|---|
| In Scope | 文档目录迁移、功能索引、migration map、静态引用扫描。 |
| Out of Scope | 业务代码整改、凭据读取、runtime、真实交易、production write、CR-033、CR102-CR104 关闭。 |

## 组织规则

| 层 | 当前入口 | 历史证据 |
|---|---|---|
| `docs/design` | current-facing HLD / ADR / BLUEPRINT / MATRIX / PACKAGE-IDENTITY | CR-scoped 设计文件迁入功能域 `design/by-cr/`。 |
| `docs/features` | 功能域目录 | CR-scoped feature 目录迁入功能域 `evidence/by-cr/`。 |
| `docs/quality` | README 和当前 guardrail | CR-scoped quality 文件迁入 `by-feature/<feature>/by-cr/<CR>/`。 |
