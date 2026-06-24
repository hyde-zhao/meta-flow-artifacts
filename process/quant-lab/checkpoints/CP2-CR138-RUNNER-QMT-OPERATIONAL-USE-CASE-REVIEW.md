---
checkpoint: "CP2"
cr_id: "CR-138"
title: "Runner & QMT Gateway Operational Use-Case Baseline Review"
status: "approved"
created_at: "2026-06-24T12:36:00+08:00"
created_by: "host-orchestrator"
gate_profile: "standard"
recommended_decision: "approve"
---

# CP2-CR138 Runner & QMT Gateway Operational Use-Case Baseline Review

## Decision Brief

### 推荐决策

已批准 CR138 的 `process/USE-CASES.md` v1.16，将 UC-33..UC-50 冻结为 Runner 与 QMT Gateway 的运营 use-case baseline。后续不再继续按 RunSpec / bundle / registry 等离线小能力碎片推进，而是先进入功能模型、边界设计和 HLD，再拆分可执行 Story。

### 背景

CR137 已关闭为 `READY_WITH_RISK`，并在 context-reset handoff 中明确下一步应启动 CR138：先回答真实用户如何使用 Runner 与 QMT Gateway，再讨论功能拆分和实现。用户已纠偏：online use-case 不能只写检查、准入或 smoke，必须覆盖多因子策略如何运行、日常如何检查和运维、事件型策略如何运行、日志如何检查和运维，以及策略如何调整、发布、暂停、回滚。

本轮只做 use-case baseline，不授权 runtime、QMT、MiniQMT、XtQuant、凭据、账户、行情、交易、NAS、provider/lake/catalog 或 Git remote 写入。

`meta-flow gate classify` 对本 CR 的 QMT / gateway 关键词给出 `runtime-high-risk` 分类，意味着后续仍需 CP2 / CP3 / CP5 / CP8 人工门禁；但 CR tracking 索引保持 `gate_profile=standard`，以避免把“场景基线”误登记为 runtime authorization。

### Baseline Delta

| 范围 | 内容 |
|---|---|
| 上一确认版本 | `process/USE-CASES.md` v1.15，UC-01..UC-32 |
| 当前确认版本 | `process/USE-CASES.md` v1.16 confirmed，UC-01..UC-50 |
| 新增 Runner use case | UC-33..UC-43：多因子调仓、盘前确认、执行跟踪、事件型策略、信号型策略、组合再平衡、盘中总览、日志证据、盘后复盘、异常恢复、策略调整/发布/暂停/下线/回滚 |
| 新增 QMT Gateway use case | UC-44..UC-50：启动健康与能力探测、账户/持仓/订单/成交查询、行情订阅缓存回补、下单撤单回报、连接异常恢复、审计日志定位、升级回滚和配置变更 |
| 合并 / 刷新审查 | 新增“CR-138 场景归一化与合并审查”；UC-16/17 保留为 Gateway 架构来源，UC-44..50 承接 Gateway 运营路径；UC-28..32 保留为双目标交付框架，UC-33..43 承接 Runner 运营路径 |
| 后续输入 | requirement extraction、feature boundary、HLD、Story 拆分、验证矩阵 |

### 研究报告映射

| 来源 | 被吸收的关键事实 | 映射到 |
|---|---|---|
| lite-qmt-executor | 信号接入、任务状态机、订单跟踪、WAL 恢复、柜台对账 | UC-37、UC-42、TS-138-05 |
| BulletTrade | Context / Portfolio、EventBus、订单队列、RiskController、QMT Guard、回测/实盘同构 | UC-33、UC-35、UC-38、UC-42 |
| EasyXT | 多因子 live、双均线、条件单、网格、跟单、轮询策略、数据源降级 | UC-33、UC-36、UC-37、UC-38 |
| quant-qmt-proxy | REST / gRPC / WebSocket、多环境、会话、订阅、事件流 | UC-44、UC-46、UC-47 |
| qmt-bridge | 服务生命周期、交易管理器、通知、WebSocket、调度器拆分、xtdata 串行化保护、故障恢复 | UC-44、UC-46、UC-48、UC-49、UC-50 |

### 推荐方案

| 方案 | 结论 | 理由 |
|---|---|---|
| A. 批准运营 use-case baseline | 推荐 | 与用户纠偏一致，先定义用户如何运营 Runner / Gateway，再进入架构和实现。 |
| B. 继续补离线 Runner 小能力 | 不推荐 | 会继续偏离用户对 online operational use-case 的要求，无法解释多因子、事件、日志、运维和策略变更工作流。 |
| C. 直接进入 QMT Gateway / runtime 实现 | 不推荐 | 运行授权、安全边界和职责划分尚未冻结，容易误触 QMT、账户、行情和交易权限。 |

## 待人工决策清单

| 决策 ID | 类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR138-01 | scope | 是否确认 CR138 的目标是 Runner / QMT Gateway 运营 use-case baseline，而不是继续补离线小 slice 或直接实现 online runtime？ | 确认；以 UC-33..UC-50 为 draft baseline，后续先进入功能模型 / HLD。 | 继续补离线 runner batch run / reporting 小能力；或直接启动 QMT Gateway / online runtime 设计。 | 推荐方案能修正方向并减少实现误授权；备选会延续碎片化或提前暴露高风险 runtime。 | 决定后续工作是否回到产品场景和功能模型。 | 若用户认为范围过大，可缩小为 Runner-only 或 Gateway-only baseline。 |
| DQ-CP2-CR138-02 | architecture | 是否确认 Runner 与 QMT Gateway 的职责边界？ | Runner 负责策略如何运行、观察、调整、复盘；QMT Gateway 负责安全稳定触达 QMT / MiniQMT / XtQuant。 | Runner 直接内嵌 Gateway / xtquant 调用；或 Gateway 同时承担策略调度和策略逻辑。 | 推荐方案职责清晰、便于测试和授权；备选短期简单但后续耦合强。 | 边界不清会造成策略直连 QMT、Gateway 过度业务化和测试不可分。 | HLD 若发现单机 PoC 需要更薄边界，可作为架构决策项重新评估。 |
| DQ-CP2-CR138-03 | runtime_authorization | CR138 是否授权 runtime、QMT、MiniQMT、XtQuant、凭据、账户查询、行情订阅、submit/cancel 或 simulation/live？ | 不授权；全部保持 deny-default，真实运行另起 runtime_authorization CR。 | 另起只读 Gateway runtime authorization gate；或由用户手动提供脱敏运行证据。 | 推荐方案防止 use-case baseline 被误读为运行许可；备选需要单独授权和回滚条件。 | 任何误授权都可能触发真实账户、行情、订单或交易风险。 | 用户明确给出运行窗口、边界、脱敏和回滚条件后，另起 CR。 |

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| CR138 已登记 | PASS | `process/changes/CR-138-RUNNER-QMT-GATEWAY-OPERATIONAL-USE-CASE-BASELINE-2026-06-24.md` |
| Context capsule 已生成 | PASS | `process/context/CP2-CR138-RUNNER-QMT-OPERATIONAL-USE-CASE-CONTEXT.yaml` |
| CP2 自动预检通过 | PASS | `process/checks/CP2-CR138-RUNNER-QMT-OPERATIONAL-USE-CASE-PRECHECK.md` |
| 场景讨论追溯已记录 | PASS | `process/discussions/CP2-SCENARIO-DISCUSSION-LOG.md`、`process/checks/CP2-DISCUSSION-CHECKPOINT.json` |

## Checklist

| 检查项 | 状态 | 说明 |
|---|---|---|
| UC-33..UC-50 能表达真实用户如何使用 Runner / QMT Gateway | PASS | 覆盖策略运行、观察、调整、复盘和 Gateway 运维 |
| 其他场景是否需要合并和刷新已复核 | PASS_WITH_RISK | `process/checks/CR138-USE-CASE-NORMALIZATION-AUDIT-2026-06-24.md`；旧确认基线不破坏性合并，后续按活动簇消费 |
| Runner / Gateway 边界有可评审表达 | PASS | Runner 是策略运行控制面；Gateway 是 QMT 触达服务化边界 |
| 研究报告已转化为场景输入 | PASS | 每份报告均映射到具体 UC / TS |
| 不授权范围明确 | PASS | deny-default 已写入 CR、context、precheck |
| 后续阶段入口清晰 | PASS | CP2 通过后进入 requirement extraction / feature model / HLD |

## Exit Criteria

| 条目 | 状态 | 说明 |
|---|---|---|
| 用户批准或提出修改 | approved | 用户回复 `approve` |
| CP2 通过后状态转换 | approved | 可进入 solution-design / 功能模型与 HLD 准备 |
| CP2 不通过后的回退路径 | ready | `reject` 或重大修改时回退到 CR137 handoff / USE-CASES v1.15 confirmed baseline |

## Deliverables

| 类型 | 路径 |
|---|---|
| CR | `process/changes/CR-138-RUNNER-QMT-GATEWAY-OPERATIONAL-USE-CASE-BASELINE-2026-06-24.md` |
| Use-case baseline | `process/USE-CASES.md` |
| Context capsule | `process/context/CP2-CR138-RUNNER-QMT-OPERATIONAL-USE-CASE-CONTEXT.yaml` |
| CP2 precheck | `process/checks/CP2-CR138-RUNNER-QMT-OPERATIONAL-USE-CASE-PRECHECK.md` |
| Normalization audit | `process/checks/CR138-USE-CASE-NORMALIZATION-AUDIT-2026-06-24.md` |
| Discussion log | `process/discussions/CP2-SCENARIO-DISCUSSION-LOG.md` |
| Discussion checkpoint | `process/checks/CP2-DISCUSSION-CHECKPOINT.json` |

## 人工审查结果

| 字段 | 值 |
|---|---|
| 审查状态 | approved |
| 审查人 | user |
| 审查时间 | 2026-06-24T13:20:00+08:00 |
| 决策 | 用户回复 `approve`，接受 DQ-CP2-CR138-01、DQ-CP2-CR138-02、DQ-CP2-CR138-03 的推荐方案。 |
| 备注 | 本 CP2 仅确认 use-case baseline，不授权任何运行、连接、账户、行情、交易、凭据、NAS 或 Git remote 写入。 |
