# CR062 Rollback / Fail-Closed Reference

## 当前状态

CR062 gate 创建阶段没有执行 Git 写动作，因此当前 rollback 是 no-op：不需要撤销 remote、commit、tag、branch 或 history 操作。

## Post-Approval 回滚原则

| 场景 | 处理 |
|---|---|
| preflight scan 失败 | 不 stage、不 commit、不 remote、不 push；记录 BLOCKED。 |
| manifest 包含 forbidden path | 停止并返工 manifest；不继续。 |
| existing remote 与目标冲突 | 停止并询问用户；不 set-url。 |
| commit 失败 | 保留工作树和错误证据；不强制清理用户改动。 |
| push 失败 | 保留本地 commit 和失败证据；不 force push、不 history rewrite。 |
| 远端已部分写入 | 不删远端、不 rewrite；通过后续 forward-fix CR 处理。 |

## 禁止回滚方式

- `git reset --hard`
- `git push --force`
- history rewrite
- branch rename
- tag 创建 / 删除
- remote deletion
- 删除或移动用户未明确授权的文件

## 退出条件

CR062 CP8 前必须有 publication evidence 或明确 `NOT_READY / BLOCKED`，且风险接受项必须说明是否存在远端状态、如何观察、如何 forward-fix。
