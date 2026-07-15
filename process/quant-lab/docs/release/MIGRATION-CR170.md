---
title: "CR-170 Migration"
status: "not-applicable"
version: "1.0"
cr_id: "CR-170"
created_at: "2026-07-15T16:14:00+08:00"
---

# CR-170 Migration

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-15 | host-orchestrator inline | 记录 public schema 与数据迁移均为 N/A。 |

本 CR 不修改 public callable、Gate ID、公开 result schema、数据 lake/catalog/store 或 runtime 配置，因此数据迁移、schema migration、历史 backfill 与部署迁移均为 `N/A`。

内部新增的 `reliability_na_policy` 仅由 canonical Gate 局部消费；未来 aggregate 或 Stage3 caller 必须通过各自正式 CR 评估，不属于本迁移文档自动授权。
