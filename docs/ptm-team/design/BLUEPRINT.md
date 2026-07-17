---
status: draft
version: "1.0"
source_story_map: "docs/product/STORY-MAP.md"
source_mvp_scope: "docs/product/MVP-SCOPE.md"
confirmed_by: ""
confirmed_at: ""
source_cr: "CR-030"
---

# ptm-tse 逆向分析能力蓝图

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-15 | host-orchestrator（用户批准的 inline meta-se fallback） | 基于 CP2 基线定义四个能力域、数据归属和禁止依赖；等待 CP3 确认。 |

## 能力地图

| Capability ID | 能力域 | 用户价值 | 覆盖 Story | Owner Feature |
|---|---|---|---|---|
| CAP-RA-01 | 资格与证据治理 | 只让可信、适用且安全的事件进入分析 | ST-RA-01 | FEAT-RA-ANALYSIS |
| CAP-RA-02 | 逆向问题分析 | 把根因、引入点和流出点转为可审计结论 | ST-RA-02 | FEAT-RA-ANALYSIS |
| CAP-RA-03 | 改进输入治理 | 只将人工批准的 CA/PA 传递到正确消费者 | ST-RA-03 | FEAT-RA-IMPROVEMENT |
| CAP-RA-04 | 闭环与有效性 | 证明行动项完成并避免未验证关闭 | ST-RA-04 | FEAT-RA-TRACKING |

## Feature / Epic 边界

| Feature ID | 名称 | 负责事项 | 不负责事项 | 拥有数据 | 只读数据 | 禁止依赖 |
|---|---|---|---|---|---|---|
| FEAT-RA-ANALYSIS | 逆向分析 | 资格检查、证据目录、事实/假设、RCA、引入点、流出矩阵、报告草案 | 生产采集、事件恢复、自动批准、下游写入 | `RA Report` 草案及人工确认字段 | 脱敏事件摘要、已批准策略 | 不连接外部系统；不写消费者产物；不拥有行动项状态 |
| FEAT-RA-IMPROVEMENT | 改进输入治理 | CA/PA 草案、批准状态、目标消费者映射、已批准改进输入 | 自动分配或修改下游任务 | `Approved Improvement Input` | 已确认 RA 结论、消费者能力声明 | 不分发未批准项；不写 ptm-tde/te/tae/qa 文件 |
| FEAT-RA-TRACKING | 闭环跟踪 | 行动项状态、有效性、观察、复发和关闭检查 | 代替 Owner 完成行动，发送外部提醒 | `Action Item`、`Effectiveness Check`、`Closure Decision` | 已批准改进输入、度量摘要 | 不修改 RA 根因结论；不自行关闭；不连接通知渠道 |

## 跨 Feature 流程

| Flow ID | 触发 | 参与 Feature | 数据写入 Owner | 失败路径 | 验证入口 |
|---|---|---|---|---|---|
| FLOW-RA-01 | 已恢复 P1/P2 事件摘要 | ANALYSIS | FEAT-RA-ANALYSIS | 资格/证据不合格 → 缺口状态 | SCN-RA-01/02/06/07 |
| FLOW-RA-02 | 人工确认问题分析 | ANALYSIS → IMPROVEMENT | FEAT-RA-IMPROVEMENT | 未确认 → 不产生改进输入 | SCN-RA-03/04 |
| FLOW-RA-03 | CA/PA 已批准 | IMPROVEMENT → TRACKING | FEAT-RA-TRACKING | 目标消费者未就绪 → 待分发 | SCN-RA-04/05 |
| FLOW-RA-04 | 关闭请求 | TRACKING → ANALYSIS report status | FEAT-RA-TRACKING | 关闭条件不足 → 保持开放 | SCN-RA-05 |

## 共享能力

| Shared ID | 名称 | 使用方 | Owner | 调用方向 | 降级策略 |
|---|---|---|---|---|---|
| SH-RA-01 | 脱敏证据引用与事实标签 | ANALYSIS / TRACKING | FEAT-RA-ANALYSIS | 只读引用 → 报告 | 证据不足时仅输出缺口，不给根因结论 |
| SH-RA-02 | 人工确认状态机 | 三个 Feature | FEAT-RA-ANALYSIS（结论）/TRACKING（关闭） | 只允许显式状态迁移 | 状态缺失视为未批准 |
| SH-RA-03 | 已批准改进输入契约 | IMPROVEMENT / 下游消费者 | FEAT-RA-IMPROVEMENT | 单向生成 → 下游只读消费 | 消费者不支持时保留待分发记录 |

## 待确认边界

| Decision ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 推荐 / 备选优劣 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| CP3-DQ-CR030-01 | architecture | 分析与跟踪是否拆为两个 Skill？ | 拆为 `reverse-analysis` 与 `improvement-tracker`，共用文件契约 | 单一大 Skill；或只做报告模板 | 拆分单一职责、便于独立验证；单 Skill 文件少但状态/权限耦合；模板-only 风险最低但闭环弱 | Skill 布局、Story 和验证边界 | 若实施总 Story ≤2 且无独立跟踪状态，可合并 |
| CP3-DQ-CR030-02 | architecture | 已批准改进输入如何持久化与被消费？ | 文件化、不可变 provenance 的 `Approved Improvement Input`，消费者只读 | 直接改下游文档；或通过外部工单 API | 文件化最小授权、可审计；直接改写耦合高；API 需要 runtime/凭据 | 数据归属、下游契约、安全 | 若未来批准 runtime CR，再评估 API adapter |
| CP3-DQ-CR030-03 | implementation | 首版自动化等级？ | Level 1–2：整理/映射/草案，人工确认 | Level 3 CA/PA 自动分发；或模板-only | 推荐先证明安全与准确；自动分发效率高但风险大；模板-only学习慢 | 安全、验证、交付速度 | 连续 5 份报告审计通过后再评估 Level 3 |
