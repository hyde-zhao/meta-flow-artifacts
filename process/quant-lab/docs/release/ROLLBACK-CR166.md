---
document_id: "ROLLBACK-CR166"
cr_id: "CR-166"
status: "READY"
created_at: "2026-07-13T14:38:00+08:00"
---

# CR-166 回滚方案

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-13 | host-orchestrator inline | 定义局部源码回滚、状态回退和禁止 destructive Git 操作边界。 |

## 回滚目标

若 CP8 前发现新的 blocker，保持 CR-166 active，并按问题类型回退到 CP7、CP5、CP3 或 CP2。若 CP8 后发现 fixture/static foundation 回归，重开 CR-166 或创建修复 CR；不得通过删除审计记录、重写 ledger 或执行 destructive Git 命令掩盖问题。

## 可回滚资产

| 资产面 | 回滚策略 | 保留项 |
|---|---|---|
| 新增 engine 模块 | 通过独立修复提交移除或恢复调用面 | 失败证据、CR、CP result、ledger 均保留 |
| 既有 consumer 集成 | 恢复 CR-166 前的 projection 接口，同时保留 CR155 blocked 语义 | C1 golden compatibility 测试必须继续通过 |
| fixtures/tests | 与对应实现一起回退；不得单独删除失败测试 | provenance 与问题记录保留 |
| 产品/设计文档 | 追加修订记录或新 CR 说明，不覆盖历史基线 | CR-166 的范围与 claim ceiling 追溯保留 |
| 过程状态 | 由 status-sync/state transition 回退到最近合法 CP | append-only ledgers 不重写 |

## 触发与路由

| 触发条件 | 回退目标 | 理由 |
|---|---|---|
| 真实数据、Stage 3 或授权边界发生变化 | CP2 / 新独立 CR | 产品范围和授权已改变 |
| envelope、fold、event 或 consumer contract 发生变化 | CP3 | 架构冻结失效 |
| LLD 无法覆盖新边界 | CP5 | 设计证据需要重新确认 |
| 实现或测试回归 | CP6/CP7 回修 | 保持原范围，修复交付缺陷 |
| 需要凭据、外部运行或生产写入 | BLOCKED / authorization-required | CR-166 未授权 |

## 禁止操作

- 不执行 `git reset --hard`、覆盖式 checkout 或 ledger 重写。
- 不因回滚而读取真实数据、凭据或访问 runtime。
- 不把 typed-unavailable、blocked 或 tampered evidence 提升为 PASS。
