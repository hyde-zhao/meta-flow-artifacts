---
doc_type: discussion-log
id: CP2-SCENARIO-DISCUSSION-LOG-CR047
cr_id: CR-047
stage: requirement-clarification
created_at: "2026-08-19"
owner: host-orchestrator
participants:
  - meta-pm（委托调研与候选整理）
  - host-orchestrator（问题代理与汇总）
  - 用户（CP2 门禁确认）
---

# CP2-CR047 Scenario Gray Areas 讨论日志

> 回源型 CR（同 CR-039 模式）：场景主体 target-artifact（ptm-te case-execution），灰区聚焦回源边界与文档面，非新场景发现。用户经 CP2 人工门禁统一确认（approve 即接受全部推荐；CR-039 先例同模式）。

## 灰区识别（SGA-047-01~04）

| 灰区 | 影响交付的方面 | 选项 | 推荐 | 处理 |
|---|---|---|---|---|
| SGA-047-01 SKILL.md 文档面 | post_delay 参数对用例作者可发现性 | A 补一行+修订记录 / B 不补写 N/A | A | **待用户**（= DQ-047-01 / SGQ-047-01） |
| SGA-047-02 workflow_mode | 审计链深度与并行 CR 基线证据 | A standard / B fast-lane | A | **待用户**（= DQ-047-02 / SGQ-047-02） |
| SGA-047-03 并行 CR-036 基线口径 | CR-036 收尾验证若在 CR-047 落地后执行，基线是否含 post_delay | A 注明包含增量 / B 不注明 | A | **agent 默认处理**（冲突预检用户已选并行推进，注明有据可查，approve 即接受） |
| SGA-047-04 post_delay 解析语义边界 | 负数 / 非法值 / 类型行为 | A 与工作区实测一致（None/空/非法返回 0.0，负数按原值）/ B 收紧（负数 clamp 0） | A | **agent 默认处理**（回源原则：与工作区已验证行为逐块一致，不引入新语义；单测覆盖边界） |

## SGQ 场景确认交互（4 条，2 条转 agent 默认处理）

| SGQ | 对应灰区 | 问题 | 选项 | 推荐 | 处理状态 |
|---|---|---|---|---|---|
| SGQ-047-01 | SGA-047-01 | SKILL.md 是否补 post_delay step 级参数说明 | A 补一行 / B 不补写 N/A | A | 已转 DQ-047-01（CP2 门禁待决策） |
| SGQ-047-02 | SGA-047-02 | workflow_mode 判定 | A standard / B fast-lane | A | 已转 DQ-047-02（CP2 门禁待决策） |
| SGQ-047-03 | SGA-047-03 | CR-036 收尾验证基线是否注明包含 post_delay 增量 | A 注明 / B 不注明 | A | **agent 默认处理**（注明为并行推进的既定结论，approve 即接受） |
| SGQ-047-04 | SGA-047-04 | 非法/边界 post_delay 值解析行为 | A 与工作区一致 / B 收紧 | A | **agent 默认处理**（回源逐块一致原则，approve 即接受） |

## 未选灰区（Deferred）

- post_delay 语义扩展（pre_delay / 全局默认值 / yaml 单位扩展）--Out of Scope 第 5 条，实战需求出现时走 BACKLOG。
- rollback 循环 post_delay 支持--工作区未改，Out of Scope 第 1 条。

## 恢复点

`process/checks/CP2-DISCUSSION-CHECKPOINT-CR047.json`
