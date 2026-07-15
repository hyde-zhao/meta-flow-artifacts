---
title: "CR-169 Rollback"
status: "cp8-draft"
version: "1.0"
cr_id: "CR-169"
created_at: "2026-07-15T10:15:00+08:00"
owner: "host-orchestrator"
---

# CR-169 Rollback

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-15 | host-orchestrator | 定义 C4 additive foundation 的非破坏性回退边界。 |

## 回退范围

CR-169 是 additive repository-local foundation。回退对象是 C4 calculator/evidence/local adapter、C4 catalog activation、fixtures/tests/checker 和 CR169 文档/过程产物；不得覆盖或删除用户无关修改。

## 回退策略

- 未提交状态：停止门禁并逐文件审查，不使用 `git reset --hard`、force 或无差别 restore。
- 已经本地提交：优先用新的 revert commit 回退 CR-169 的两个仓库提交，随后运行 C1-C3/CR168 与 full suite。
- 不回退或修改 canonical Gate4、CR168 adapter、CR155 状态，因为本 CR 未拥有这些对象。

## 失败后状态

C4 可恢复为 reserved/typed unavailable；CR168 的 absent-C4 fail-closed 路径继续成立。Stage2 complete claim 必须撤回为 false，Stage3 仍不得进入。
