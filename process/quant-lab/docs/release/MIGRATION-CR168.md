---
document_id: "MIGRATION-CR168"
cr_id: "CR-168"
status: "NOT_APPLICABLE"
created_at: "2026-07-14T15:06:00+08:00"
---

# CR-168 迁移说明

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline | 声明无真实数据、持久化状态或 runtime 迁移，并记录 component schema 演进约束。 |

本 CR 的 data/state/runtime migration 全部为 `N/A`：没有读取或转换真实数据，没有 lake/NAS/provider/catalog/store/registry 写入，没有数据库 schema 或服务配置迁移。

- `economic_cost@v1` 是新增 versioned component；v1 formula、rounding 和 active `square_root` family 冻结不可变。
- 新 active family、rebate、公式或舍入变化必须使用新的 schema version（默认 v2）和独立 method CR。
- C4 仍为 typed unavailable；C4-present adapter 的演进由 FU-CR161-005 决定。
- 不得把 static fixtures 或 component schema 宣称为真实 TCA/capacity migration readiness。
