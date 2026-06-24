---
discussion_id: "CP2-CR138-SCENARIO-DISCUSSION"
status: "completed"
created_at: "2026-06-24T12:18:00+08:00"
created_by: "host-orchestrator"
cr_id: "CR-138"
source_context: "process/context/CR137-CLOSURE-TO-RUNNER-QMT-USE-CASE-CONTEXT-RESET-HANDOFF-2026-06-24.md"
---

# CP2 Scenario Discussion Log - CR138

## 讨论目标

冻结 Runner 与 QMT Gateway 的 operational use-case baseline。重点从“离线 Runner 小能力规格”切换为“用户如何使用 Runner 和 QMT Gateway 完成日常策略运行、运维、恢复、发布和审计”。

## Scenario Gray Areas

| 灰区 ID | 问题 | 为什么重要 | 影响面 | 当前处理 | 状态 |
|---|---|---|---|---|---|
| SGA-138-01 | CR138 是继续补 Runner 离线能力，还是先冻结用户如何运营 Runner 与 QMT Gateway？ | 决定后续继续小能力实现，还是先回到产品场景和功能模型。 | 范围、复杂度、验证、后续门控 | 采用运营 use-case baseline。 | resolved-by-user |
| SGA-138-02 | online use-case 是否只覆盖检查 / 准入 / smoke？ | 只写检查会遗漏真实运行、日志运维、策略发布和异常恢复。 | 范围、验证、后续 HLD | UC-33..UC-50 覆盖真实运行和运维。 | resolved-by-user |
| SGA-138-03 | Runner 与 QMT Gateway 的职责边界如何划分？ | 防止 Runner 直连 xtquant 或 Gateway 决定策略逻辑。 | 架构、接口、测试、权限 | Runner 负责策略如何运行；Gateway 负责如何安全稳定触达 QMT。 | decision-item |
| SGA-138-04 | CR138 是否授权真实 QMT、账户查询、行情订阅或下单撤单？ | 防止 use-case baseline 被误读为 runtime authorization。 | 安全、运行授权、交付风险 | 不授权；真实运行另起 gate。 | decision-item |
| SGA-138-05 | 是否需要 clone / install / run 外部项目？ | 会引入依赖、许可证和外部运行风险。 | 调研、运行授权、安全 | 本轮充分参考两份研究报告，不 clone / run。 | resolved-by-user |
| SGA-138-06 | UC-01..UC-50 中是否还有其他场景需要合并和刷新？ | 防止旧 QMT C/S、双目标交付框架与新 Runner / Gateway 运营场景重复消费。 | 场景归一化、后续 HLD、测试矩阵 | 新增归一化与合并审查；保留旧确认编号，按活动簇刷新消费规则。 | resolved-by-user |

## Scenario Confirmation Interactions

| question_id | question | options | recommendation | user_response | confirmed_understanding | impact_surface | source_refs | status |
|---|---|---|---|---|---|---|---|---|
| SGQ-001 | CR138 是否应继续按 RunSpec / bundle / registry 等离线规格推进，还是先冻结真实用户如何运营 Runner 与 QMT Gateway？ | A. 运营 use-case baseline；B. 继续离线小功能；C. 直接进入 online runtime / Gateway 实现 | 推荐 A：先冻结运营场景，再进入功能模型 / HLD | CR137 handoff 记录用户纠偏：use-case 不是 RunSpec、bundle、registry、validation 这些规格；use-case 必须描述用户如何使用 Runner；online use-case 不能只写检查 / 准入 / smoke。本轮用户要求推进 CR138 并充分参考两份 QMT 生态研究报告。 | 采用运营 use-case baseline；不继续只补离线小功能，不实现 online runtime，不授权真实 QMT。 | scope, architecture, verification, runtime_authorization | CR137 handoff, two QMT research reports, process/USE-CASES.md v1.16 draft | confirmed |
| SGQ-002 | UC-01..UC-50 是否还有其他场景需要合并和刷新？ | A. 非破坏性归一化并保留旧编号；B. 删除 / 合并旧 UC 并重编号；C. 暂不处理 | 推荐 A：保留历史追溯，新增合并/刷新映射 | 用户在 CP2 前反馈：你再检查一下 use-cases 文档中的其他场景是否需要合并和刷新。 | 已执行非破坏性归一化；UC-16/17 作为 Gateway 架构来源，UC-44..50 承接 Gateway 运营路径；UC-28..32 作为双目标交付框架，UC-33..43 承接 Runner 运营路径。 | scope_normalization, traceability, architecture, verification | process/USE-CASES.md, process/checks/CR138-USE-CASE-NORMALIZATION-AUDIT-2026-06-24.md | addressed |

## Deferred Ideas

| ID | 想法 / 风险 / 扩展场景 | 来源 | 延后原因 | 重启条件 |
|---|---|---|---|---|
| DEF-138-01 | Runner 功能模型与能力地图 | UC-33..UC-43 | 需要 CP2 批准 use-case baseline。 | 用户 approve CR138 CP2。 |
| DEF-138-02 | QMT Gateway HLD / protocol boundary | UC-44..UC-50 | 需要先确认 Gateway 运营场景和不授权边界。 | CR138 CP2 通过后启动 solution-design。 |
| DEF-138-03 | Runtime authorization gate | SGA-138-04 | 真实运行、账户查询、行情订阅和交易写入高风险。 | 用户明确授权并提供边界。 |
| DEF-138-04 | 外部项目源码级 Spike | SGA-138-05 | 两份研究报告已足够支撑 use-case baseline。 | HLD 发现报告不足且用户批准 clone 到临时目录。 |

## 结论

`completed`。CR138 的场景灰区已收敛为 UC-33..UC-50 draft，并已完成 UC-01..UC-50 合并 / 刷新归一化审查；等待 CP2 人工确认。
