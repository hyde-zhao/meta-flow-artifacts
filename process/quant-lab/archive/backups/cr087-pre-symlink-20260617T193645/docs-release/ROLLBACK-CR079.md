# CR079 Rollback

## 回滚范围

CR079 当前交付只新增和更新 process / release 证据文件；没有代码变更、安装变更、外部发布、真实数据复制或删除。

## 可回滚项

| 项 | 回滚方式 | 说明 |
|---|---|---|
| CP8 process 证据 | 删除或修订 CR079 CP8 文件 | 仅限本地过程文档，需保留审计说明。 |
| STATE / CR-INDEX 状态 | 回退为 `active-approved-ready-for-metadata-inventory` 或 `active-cp7-pass-with-risk-pending-cp8` | 只在 CP8 reject 或需要重审时使用。 |
| release docs | 修订 `docs/release/*-CR079.md` | 无外部部署影响。 |

## 不可在本回滚内执行

- 不删除、移动、重命名或清理旧根。
- 不删除 `reports/`、`data/`。
- 不恢复、复制或覆盖 target root 数据。
- 不读取 `.env`、凭据、账户或交易事实。

## 验证

回滚或修订后重新运行：

```text
uv run --python 3.11 python scripts/check_human_gate_decision_brief.py --checkpoint-file process/checkpoints/CP8-CR079-DELIVERY-READINESS.md --launch-message-file process/checks/CP8-CR079-HUMAN-GATE-LAUNCH-MESSAGE.md
uv run --python 3.11 python scripts/check_cr_tracking_consistency.py --project-root .
```
