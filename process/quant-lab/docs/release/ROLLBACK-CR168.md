---
document_id: "ROLLBACK-CR168"
cr_id: "CR-168"
status: "READY"
created_at: "2026-07-14T15:06:00+08:00"
---

# CR-168 回滚方案

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline | 定义非破坏性的代码、流程与风险回退路径。 |

## 回滚原则

当前两个工作区均存在 active CR-168 工件及用户已有修改，因此不得使用 `git reset --hard`、覆盖式 checkout、删除 ledger 或批量还原工作区。若 CP8 不批准，CR-168 保持 active/pending，已实现源码和证据不被删除。

| 触发条件 | 回退目标 | 处理 |
|---|---|---|
| 新的 C3 formula/hash/validation defect | CP6/CP7 S01–S05 | 创建修复切片，保留失败 fixture/test 与现有 ledger。 |
| C3/C4 contract 或 Gate4 policy 变化 | CP3 | 重开架构门禁；不得以局部 adapter 隐藏全局语义变化。 |
| real data/TCA/C4/runtime/Stage3 请求 | CP2 或独立 CR | 先澄清范围与授权，不能复用本 CR。 |
| CR155 promotion 诉求 | FU-CR161-007 / 独立 CR | 维持 blocked 与 paper_candidate=false 直到独立证据链和门禁。 |
| Git publication 需要撤销 | 独立远端授权流程 | 本次没有远端写入；不得假定可 force-push。 |

回退始终保留 CP、return、evidence index、评审 finding 和风险记录的追溯链。
