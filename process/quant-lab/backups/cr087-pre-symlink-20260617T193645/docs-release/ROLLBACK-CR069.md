# CR069 Rollback

## 回滚范围

CR069 回滚只允许回滚本地 process ledger 说明，不允许执行任何外部系统操作。

| 对象 | 回滚动作 |
|---|---|
| CR062 formal CR | 从 `closed-blocked-superseded` 恢复为 pre-CR069 blocked wording。 |
| CR-INDEX | 将 CR062 从 closed_crs 恢复到 blocked_crs。 |
| STATE | 将 CR062 从 closed_crs 恢复到 blocked_crs，并将 CR069 恢复到 CP5 后待执行状态。 |
| Manifest | 标记 rollback evidence。 |

## 触发条件

| 条件 | 动作 |
|---|---|
| CP8 reject | 回滚或按用户修改重新生成 CR069 CP8。 |
| 发现 replacement evidence 不完整 | 回到 CP3/CP5 修订。 |
| 用户要求重新执行 CR062 原路线 | 拒绝在 CR069 内执行，另起 Git publication CR 并重新门禁。 |

## 不可回滚项

CR066 / CR067 / CR068 的历史事实不由 CR069 回滚；远端仓库事实不由 CR069 修改。
