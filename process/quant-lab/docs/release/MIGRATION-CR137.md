# Migration CR137

## 结论

| 项目 | 结论 |
|---|---|
| 状态 schema 迁移 | N/A |
| 数据库迁移 | N/A |
| 外部配置迁移 | N/A |
| 安装路径迁移 | N/A |
| CLI 行为迁移 | additive，可选新参数 |
| 文件格式迁移 | 新增 `runner-runs.index.json`；既有格式不变 |

## 兼容性说明

- 既有 RunSpec 文件无需修改。
- 仅当用户显式提供 `run_registry_output_path` 或 CLI `--registry-output` 时才写 registry。
- registry pass entry 依赖 valid artifact bundle；blocked diagnostic entry 不链接 pass bundle。
- registry 不替代 artifact bundle，只提供本地运行索引。

## 不授权项

迁移不涉及 runtime/NAS/QMT/credentials/provider/lake/catalog/trading。
