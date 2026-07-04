# CR079 Migration

## 结论

CR079 当前没有执行真实数据迁移。用户语句中的“迁移”在本 CR 中按已批准的 metadata-only 清点和交付门禁收敛处理，不扩大为复制、恢复、移动、重命名或删除授权。

## 数据状态

| 位置 | `reports/` | `data/` | 结论 |
|---|---|---|---|
| `/home/hyde/workspace/quant-lab` | 缺失 | 缺失 | target root 仍不完整。 |
| `/home/hyde/workspace/local_backtest` | 存在 | 存在 | legacy retained assets 可作为后续恢复候选输入。 |

## 后续迁移入口

真正迁移 / 恢复必须另起 CR，并至少明确：

- 来源路径和目标路径。
- copy / restore / sync 的具体命令族。
- 是否允许内容读取、hash、样本打印或解析。
- 是否允许 NAS、provider、lake 或 catalog。
- 验证方式和回滚方式。

默认推荐后续入口为 `CR079-restore-candidate`。
