# Backlog — CR-038（后续候选项与延后项）

> 本轮建议范围见 MVP-SCOPE-CR-038.md；本表记录后续候选与阻塞前置。

## 后续候选（Deferred Ideas 转 backlog）

| ID | 内容 | 优先级 | 阻塞前置 | 重启条件 |
|---|---|---|---|---|
| DEF-038-01 | `${ENV.sw.*}` 占位符（用例直接引用 SW 端口） | P2 | 无 | 未来用例需 case_steps 操作 SW |
| DEF-038-02 | ngfw-pppoe factor-library | P2 | 无 | 未来需 PPPoE 测试设计因子 |
| DEF-038-03 | PPPoE Server 强制 VRF 隔离 | P2 | 无 | 未来拨号需 VRF 隔离 |
| DEF-038-04 | SW 配置回滚闭环（teardown 撤销） | P1 | CP7 验证暴露回滚需求 | CP7 回修/设计澄清 |

## 阻塞前置（本轮落地需先闭环）

| ID | 内容 | 阻塞 | 处置 |
|---|---|---|---|
| RA-038-001 | ptm-atomic CLI 是否暴露 PPPoE 拨号 op | R-F-005 | DQ-038-03 三选一：本仓库 op_mapper 映射到可用 op / 提外部依赖 / 降级手工步骤 |
| DQ-038-05 | PPPoE 地址池网段 + local-user 数据源契约 | R-F-019 | 待 CP2 确认（physical_pool 新字段 / devices.yaml / 独立配置） |
| SGA-01 | PPPoE 动态地址 vs 静态 IPAM | R-C-006 | 待 CP2 确认（SGQ-038-01） |
| SGA-04 | PPPoE Server 是否强制 VRF | R-F-003 | 待 CP2 确认（SGQ-038-03） |
| R-F-018 | 显式 SW 优先级 vs 自动透传 | R-F-018 | agent 默认处理（显式优先），互斥逻辑进 HLD，不阻塞 CP2 |

## 风险跟踪

| 风险 ID | 内容 | 状态 | 缓解 |
|---|---|---|---|
| RA-038-002 | H3C 真机命令与参考命令族差异 | open | dry-run diff 人工核对 + 真机独立授权 |
| RA-038-004 | 环回拓扑放宽 MVP 引入组合爆炸 | open | 首版覆盖参考场景 + 通用节点类型判定 |
| RA-038-007 | 真机下发 × 无 teardown 撤销 | open（风险接受） | 真机下发前人工核对 diff + 记录命令供手工回滚；CP7 暴露时升级 |

## 非本轮范围（明确不做）

- 固化单条 PPPoE 环回链路。
- ptm-atomic CLI 本体修改。
- 现有 GE 实例（GE1_1~4）改动。
