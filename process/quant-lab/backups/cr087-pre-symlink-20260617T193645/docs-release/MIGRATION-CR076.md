# Migration CR076

## 迁移结论

N/A。CR076 不迁移数据、不迁移目录、不修改 schema、不移动旧根、不切换 data lake，也不改变运行时配置。

## 未来触发条件

若后续需要真实访问或恢复 `reports/`、`data/`，必须另起 CR，并在该 CR 中单独定义：

- 访问目标：local retained、NAS archive 或 provider rebuild。
- 授权范围：读取 / 列出 / 复制 / 恢复 / 比对 / 删除中具体哪一种。
- 证据脱敏方式。
- 回退和 cleanup 边界。

## 禁止动作

本迁移说明不授权任何真实迁移、恢复、删除或 provider rebuild。
