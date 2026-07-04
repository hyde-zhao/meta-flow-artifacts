# CR079 Release Notes

## 版本决策

| 字段 | 内容 |
|---|---|
| change_id | CR-079 |
| release_decision | READY_WITH_RISK |
| release_artifact_profile | minimal |
| 版本类型 | process / evidence only |

## 用户可见变化

- CR079 已在 CP2 / CP3 / CP5 批准后完成一次 metadata-only 只读清点。
- 目标根 `/home/hyde/workspace/quant-lab` 当前缺少 `reports/` 和 `data/`。
- legacy 根 `/home/hyde/workspace/local_backtest` 仍保留 `reports/` 和 `data/` 聚合资产。
- 本轮没有执行复制、恢复、删除、内容读取或内容校验。

## 已知风险

| 风险 ID | 严重度 | 状态 | 摘要 |
|---|---|---|---|
| R-CR079-01 | HIGH | accepted-by-cp8 | target root 缺少 `reports/` 和 `data/`。 |
| R-CR079-02 | MEDIUM | accepted-by-cp8 | legacy retained assets 存在，但内容质量未验证。 |
| R-CR079-03 | LOW | accepted-by-cp8 | 风险命名命中只是文件名模式提示，不是内容证据。 |

## 后续候选

- `CR079-restore-candidate`：单独授权从 legacy 或其他已确认来源恢复 / 复制 `reports/`、`data/`。
- `CR079-nas-compare-candidate`：单独授权 NAS 归档对比。
- `CR079-provider-rebuild-candidate`：单独授权 provider / lake / catalog rebuild。
