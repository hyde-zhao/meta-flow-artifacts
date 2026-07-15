# CR-047 Migration

无需业务数据迁移。过程仓迁移点只有两项：canonical CR index 仅使用 `process/changes/CR-INDEX.json`，legacy YAML 已移除；根 `AGENTS.md` 是可生成 wrapper，tracked canonical 为 `delivery/rules/AGENTS.md`。已有 routed workspace 可重复运行 `meta-flow workspace link`，元数据不会仅因时间戳变脏。

若目标存在 regular `process` 路径，命令仍 fail-closed；必须由用户先明确迁移/移动，不会静默覆盖。
