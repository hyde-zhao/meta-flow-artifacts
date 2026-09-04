# Story Map — CR-038（PPPoE 链路规划能力）

> 来源：USE-CASES-CR-038.md / REQUIREMENTS-CR-038.md
> 用户活动 → 任务 → Story（候选），供 meta-se 拆解与 CP4/CP5 消费。

## 用户活动与任务

| 活动 | 任务 | Story 候选 | 来源 UC / REQ | 优先级 |
|---|---|---|---|---|
| A1：规划 PPPoE 链路拓扑 | A1-T1：SW 显式节点映射 | STORY-038-01 SW 节点显式映射（含显式 SW 优先级） | UC-038-01 / R-F-001, R-F-002, R-F-018 | P0 |
| | A1-T2：SW 节点归并与台账 | STORY-038-02 SW 节点归并（pool_merge） | UC-038-01 / R-F-011 | P1 |
| A2：生成 PPPoE 配置 | A2-T1：PPPoE Server 命令族 | STORY-038-03 PPPoE Server 命令族（commands，含数据源契约） | UC-038-02 / R-F-003, R-F-004, R-F-019 | P0 |
| | A2-T2：PPPoE Server 配置下发 | STORY-038-04 PPPoE Server 下发（switch_configurator） | UC-038-02 / R-F-003 | P0 |
| | A2-T3：PPPoE Client 拨号 | STORY-038-05 PPPoE Client（ptm-atomic/op_mapper） | UC-038-03 / R-F-005 | P0 |
| A3：导出环回 env-file | A3-T1：突破 MVP 校验 | STORY-038-06 exporter 突破 MVP + 环回 | UC-038-04 / R-F-006, R-F-007 | P0 |
| | A3-T2：case_runner SW 兼容 | STORY-038-07 case_runner env-file SW 契约 | UC-038-04 / R-F-008 | P1 |
| | A3-T3：多实例 TRex 消费与导出 | STORY-038-13 多实例 TRex（physical_pool + pool_merge + exporter） | UC-038-04 / R-F-013, R-F-014, R-F-015, R-F-016, R-F-017 | P0 |
| A4：配置 SW 设备台账 | A4-T1：SW 设备 schema | STORY-038-08 devices.yaml sw 块 | UC-038-05 / R-F-009 | P1 |
| | A4-T2：SW 平台别名 | STORY-038-09 device-reference SW 别名 | UC-038-05 / R-F-010 | P1 |
| A5：验证 PPPoE 链路 | A5-T1：双向发流 0 丢包 | STORY-038-10 trex 双向发流验证 | UC-038-06 / R-F-012 | P0 |
| A6：集成编排 | A6-T1：ptm-te 编排集成 | STORY-038-11 ptm-te.md 编排 PPPoE 场景 | UC-038-01~06 / 集成 | P1 |
| | A6-T2：拓扑建模文档 | STORY-038-12 topology-collection Mock→SW 建模 | UC-038-01 / 建模 | P1 |

## Story 汇总

| Story | 优先级 | 来源 | 关键 REQ |
|---|---|---|---|
| STORY-038-01 | P0 | UC-038-01 | R-F-001, R-F-002, R-F-018 |
| STORY-038-02 | P1 | UC-038-01 | R-F-011 |
| STORY-038-03 | P0 | UC-038-02 | R-F-003, R-F-004, R-F-019 |
| STORY-038-04 | P0 | UC-038-02 | R-F-003 |
| STORY-038-05 | P0 | UC-038-03 | R-F-005 |
| STORY-038-06 | P0 | UC-038-04 | R-F-006, R-F-007 |
| STORY-038-07 | P1 | UC-038-04 | R-F-008 |
| STORY-038-08 | P1 | UC-038-05 | R-F-009 |
| STORY-038-09 | P1 | UC-038-05 | R-F-010 |
| STORY-038-10 | P0 | UC-038-06 | R-F-012 |
| STORY-038-11 | P1 | 集成 | 编排 |
| STORY-038-12 | P1 | 建模 | 拓扑建模 |
| STORY-038-13 | P0 | UC-038-04 | R-F-013, R-F-014, R-F-015, R-F-016, R-F-017 |

## Epic 建议

| Epic | 范围 | Story |
|---|---|---|
| E-038-1：SW 映射与台账 | SW 节点显式映射（含显式 SW 优先级）+ 归并 | STORY-038-01, STORY-038-02 |
| E-038-2：PPPoE 配置 | Server（含数据源契约）+ Client 配置生成与下发 | STORY-038-03, STORY-038-04, STORY-038-05 |
| E-038-3：环回 env-file | exporter 突破 MVP + case_runner 兼容 + 多实例 TRex | STORY-038-06, STORY-038-07, STORY-038-13 |
| E-038-4：SW 设备管理 | devices.yaml + device-reference | STORY-038-08, STORY-038-09 |
| E-038-5：验证与集成 | trex 验证 + 编排 + 建模 | STORY-038-10, STORY-038-11, STORY-038-12 |
