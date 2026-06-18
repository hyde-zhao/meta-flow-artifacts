# CR091 Rollback

> release_decision: `READY_WITH_RISK`
> profile: `compact`
> generated_at: `2026-06-18T15:24:12+08:00`

## 回滚结论

CR091 当前交付是离线代码 / fixture / checker / 文档 / 过程账本。没有真实发布、没有数据库迁移、没有 NAS 写入、没有 runtime 启动，因此回滚是代码和账本状态回滚，不涉及生产数据。

## 触发条件

| 触发 | 处理 |
|---|---|
| 用户在 CP8 回复 `reject` | 回退到 `active-cp7-pass-with-risk-pending-cp8`，不关闭 CR091。 |
| 用户要求缩小范围 | 修改 CP8 Decision Brief、Release Context 和不授权项后重新发起确认。 |
| 后续发现离线合同存在阻断缺陷 | 新建回修任务，回到 CP6/CP7 复验。 |
| 用户要求真实 runtime 验证 | 不作为回滚处理；应新建 runtime authorization gate。 |

## 回滚范围

| 范围 | 路径 / 对象 |
|---|---|
| 代码 | `trading/strategy_runner/` |
| checker | `scripts/check_cr091_strategy_runner_package.py` |
| 测试 | `tests/test_cr091_strategy_runner_contracts.py` |
| fixtures | `tests/fixtures/cr091_strategy_runner/` |
| 过程账本 | CR091 CP6 / CP7 / CP8 检查点、release context、STATE / CR-INDEX 摘要 |

## 不需要回滚的内容

- NAS 内容：未访问、未写入。
- `.env` / 凭据 / 账户数据：未读取、未写入。
- QMT / MiniQMT / XtQuant / gateway / runner runtime：未启动。
- provider / lake / publish：未执行。

## 验证

回滚后至少执行：

| 命令 | 预期 |
|---|---|
| `uv run --python 3.11 meta-flow workspace check --project-root /home/hyde/workspace/quant-lab` | process route healthy |
| `git diff --check` | 无空白错误 |
| CR091 相关状态检查 | CR091 不应显示 closed-current-delivery |

## 责任和授权

回滚不授权 destructive cleanup。若需要删除代码文件、修改远端仓库历史或恢复旧提交，必须由用户另行明确授权。
