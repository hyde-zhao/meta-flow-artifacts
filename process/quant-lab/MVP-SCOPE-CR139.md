---
status: draft
version: "1.0"
cr_id: "CR-139"
confirmed_by: ""
confirmed_at: ""
---

# MVP Scope — CR-139 Strategy Data Foundation

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-28 | meta-pm | 初始 MVP 范围，Wave1 P0 为 MVP 核心 |

## 本轮目标

| 目标 ID | 用户 outcome | 度量方式 | 目标值 | 来源 |
|---|---|---|---|---|
| GOAL-CR139-01 | 整改前冻结基线，可归因 | T8 清册 + T7 黄金值 | 全 dataset 清册 + 黄金值快照生成 | UC-51 / REQ-212,213 |
| GOAL-CR139-02 | 回测信任基础（四道 P0 防线） | C1/C2/V1/R1 实现 | PIT 前视阻断 + 去重唯一 + published 快照 + as-of panel | UC-52 / REQ-204,205,214,232 |
| GOAL-CR139-03 | 不先改物理分区 | 物理迁移后置 | Wave1 N1 物理迁移在 T7 归因后 | D3/D6 / REQ-247 |

## In Scope（MVP 核心 = Wave1 P0）

| Scope ID | 内容 | 覆盖 Story 候选 | 必须完成原因 | 验证入口 |
|---|---|---|---|---|
| IN-CR139-01 | T8 整改对象自动化清册 | STC-01 | 事实校验前置，冻结基线 | SCN-001 |
| IN-CR139-02 | T7 整改回归基线 + 黄金值快照 | STC-02 | D6 强制，归因基础 | SCN-001 |
| IN-CR139-03 | C2a 重复画像 | STC-03 | 看清 38 分区重复 | SCN-007 |
| IN-CR139-04 | V1 published pointer / read selector | STC-04 | 四道 P0 防线（冻结快照） | SCN-004,005 |
| IN-CR139-05 | C1 PIT as-of reader | STC-05 | 四道 P0 防线（PIT 强制） | SCN-002,003 |
| IN-CR139-06 | R1 统一 published/as-of panel reader | STC-06 | 四道 P0 防线（多表 join） | SCN-004 |
| IN-CR139-07 | C2b 读层去重 | STC-07 | 四道 P0 防线（去重唯一） | SCN-006 |
| IN-CR139-08 | N1 读层优先 + 物理迁移后置 | STC-08 | 不先改物理分区 | SCN-022,024 |

## Out of Scope（本轮 MVP 不做）

| Scope ID | 内容 | 排除原因 | 影响 | 重启条件 |
|---|---|---|---|---|
| OUT-CR139-01 | Wave2 P1 项（ML/模拟盘解锁） | MVP 聚焦 Wave1 P0 信任基础 | ML/模拟盘暂不解锁 | Wave1 完成后进入 Wave2 |
| OUT-CR139-02 | Wave3 P2 项（实盘运维） | P2 优先级 | 实盘运维延后 | Wave2 完成后按需 |
| OUT-CR139-03 | 物理分区迁移执行 | Wave1 N1 后置到基线冻结后 | 存量分区暂不物理迁移 | T7 黄金值归因完成后 |
| OUT-CR139-04 | DuckDB 依赖引入 | 需 CP3/CP5 批准 | DuckDB adapter 暂不实现 | CP3/CP5 批准后 |
| OUT-CR139-05 | ML feature store 切换独立 store | D5 附条件 deferred | 维持 lake features/ 子层 | DEF-139-01 触发条件 |
| OUT-CR139-06 | runtime/NAS/QMT/trading/provider-lake 写入 | 不授权范围 | 真实执行需独立授权 | 后续 runtime_authorization |

## Deferred

| Deferred ID | 来源 | 内容 | 延后原因 | 后续处理 |
|---|---|---|---|---|
| DEF-139-01 | SGA-139-03 / D5 | ML feature 层切换独立 feature store | 当前 lake features/ 子层带版本够用 | backlog / 后续 CR |
| DEF-139-02 | SGA-139-04 / Wave1 N1 | 物理分区迁移 / 候选压缩 | 后置到基线冻结后 | T7 归因完成后评估 |
| DEF-139-03 | SGA-139-01 | Wave3 P2 项（L5/R4/T1/M3/M4/F4） | P2 优先级 | Wave1/2 完成后 backlog |
| DEF-139-04 | D3 | 存量 run_id 一次性重命名 | D3 不重命名 | 重跑替换不可行且冲突阻塞时 |

## 人工决策项

决策类型只能使用：`scope`、`architecture`、`security`、`implementation`、`runtime_authorization`、`risk_acceptance`、`follow_up_tracking`。

| Decision ID | 决策类型 | 问题 | 推荐方案 | 备选方案 | 推荐 / 备选优劣 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| D8a | scope | (d1) 纯新建 6 项是否纳入范围 | 纳入，按新建能力处理 | 不纳入/部分纳入 | 推荐：信任基础必要；备选：范围缩小但审计不完整 | 范围/复杂度/CP3 门禁 | CP2 修改口径则增量回更 |
| D8b | scope | (d2) 既有合同闭环 14 项是否纳入范围 | 纳入，既有合同闭环/版本化扩展，不重复设计 | 不纳入/部分纳入 | 推荐：审计闭环完整；备选：闭环断链 | 范围/设计路由/CP3 Feature | CP2 修改口径则增量回更 |
| DQ-CR139-01 | architecture | ML feature 层 lake 子层 vs 独立 store | lake features/ 子层带版本（D5 已决） | 独立 feature store | 推荐：成本低；备选：切换成本高 | V3 设计 | DEF-139-01 触发条件 |
| DQ-CR139-02 | implementation | DuckDB 依赖引入时机 | CP3/CP5 批准后引入 | 不引入 | 推荐：大查询必需；备选：pandas 撑不住 | R3/R4 | 内存瓶颈时 |
