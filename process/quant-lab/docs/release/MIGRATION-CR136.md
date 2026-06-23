# Migration CR136

## 迁移结论

| 对象 | 是否迁移 | 说明 |
|---|---|---|
| Bundle 文件结构 | 否 | CR136 不改变 CR135 valid bundle 文件集合。 |
| Manifest schema version | 否 | 仍为 `cr135-run-artifact-bundle-v1`。 |
| CLI 参数 | 是，兼容新增 | 新增 `--validate-bundle`；既有 `--inspect-bundle` 保留。 |
| Runtime / NAS / QMT / trading | 否 | 不授权、不访问。 |

## 兼容性说明

valid CR135 bundles 继续可 inspect/replay；损坏、不完整或状态不一致的 bundle 会被拒绝。
