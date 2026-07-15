---
title: "CR-170 Rollback"
status: "draft-cp8"
version: "1.0"
cr_id: "CR-170"
created_at: "2026-07-15T16:14:00+08:00"
---

# CR-170 回退方案

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-15 | host-orchestrator inline | 定义 CP8 前后的安全回退与禁止动作。 |

| 触发 | 回退目标 | 动作 |
|---|---|---|
| CP8 用户 reject | CP5-approved / CR active | 保留当前证据，停止关闭；不得删除历史或强制覆盖。 |
| CP8 用户要求修改实现 | 对应 Story CP7→CP6 | 以 `NEEDS_REWORK` 重开 Story，重跑相关及 full suite。 |
| CP8 用户要求修改架构语义 | CP3/CP5 | 创建设计修订/CR 决策并重新确认，不在关闭阶段直接改 contract。 |
| 关闭后发现 canonical 回归 | 新 CR 或回开 CR170（按治理决策） | 使用正常 revert/修复提交；禁止 reset-hard/force-push。 |

CR-168/169 adapter guard 不作为回退删除对象；aggregate、runner、Stage3 与真实数据从未启用，无运行态回滚需求。
