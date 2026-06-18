# CR046 Rollback Plan

> 回滚类型：文档与台账状态回滚。
> 不涉及代码部署、数据迁移、交易终端、NAS、凭据或账户数据。

## 触发条件

| 触发 | 回退目标 |
|---|---|
| 用户在 CP8 回复 `reject` | 回退到 CR046 `active-cp7-pass-with-risk-pending-cp8` |
| 用户回复 `修改: <具体修改点>` | 保持 CP8 pending，按修改点重出 release context / checkpoint |
| 发现 CP8 文档错误扩大了授权边界 | 立即修正 CP8 材料并重新发起人工确认 |
| 发现风险 ID 或证据路径错误 | 修正 Release Context、CP8 自动检查和人工 checkpoint |

## 回滚步骤

1. 保留 CP7 `PASS_WITH_RISK` 证据不变。
2. 将 CR046 状态恢复为 `active-cp7-pass-with-risk-pending-cp8`。
3. 将 `STATE.md` 的 pending gate 指回 CR046 CP8。
4. 修订 `process/release/RELEASE-CONTEXT-CR046.yaml`、`process/checks/CP8-CR046-DELIVERY-READINESS.md` 和 `process/checkpoints/CP8-CR046-DELIVERY-READINESS.md`。
5. 重新运行 workspace check、CR tracking consistency 和 human gate 校验。

## 禁止回滚动作

- 不删除源文档、验证报告或检查点。
- 不执行 destructive cleanup。
- 不读取 `.env`、凭据、账号、账户、资金、持仓、委托、成交或日志原文。
- 不访问 NAS。
- 不启动 QMT/MiniQMT/XtQuant/gateway。
- 不执行 submit/cancel、simulation/live。

## 回滚后状态

回滚只影响 CR046 的 CP8 发布就绪判断，不撤销 CP6 `PASS` 和 CP7 `PASS_WITH_RISK` 事实。后续仍只能在人工确认后关闭当前 framework-first 交付。
