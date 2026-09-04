# 发布切片 — CR-038（PPPoE 链路规划能力）

> 发布切片按依赖顺序排列，反映推荐实现路径（对应 REQUIREMENTS-CR-038.md 里程碑 M1~M6）。

## Slice 1：SW 显式节点映射（M1）

- **范围**：topo_mapper 支持 `node_type=SW` 显式匹配 + `role=pppoe-server` 识别 + pool_merge 归并 + 显式 SW 优先级（互斥逻辑进 HLD）。
- **需求**：R-F-001, R-F-002, R-F-011, R-F-018, R-NF-002, R-NF-004
- **Story**：STORY-038-01, STORY-038-02
- **验收**：SCN-038-POS-001/002, NEG-001/002, BND-001/002, REC-001
- **前置**：无

## Slice 2：PPPoE Server 配置（M2）

- **范围**：commands 三厂商 PPPoE 命令族 + switch_configurator 配置下发。
- **需求**：R-F-003, R-F-004, R-F-019, R-C-001, R-C-002, R-C-003
- **Story**：STORY-038-03, STORY-038-04
- **验收**：SCN-038-POS-003/004, NEG-003/004, AUTH-001
- **前置**：Slice 1

## Slice 3：PPPoE Client + SW 设备（M3）

- **范围**：PPPoE Client 拨号（ptm-atomic/op_mapper）+ devices.yaml sw 块 + device-reference SW 别名。
- **需求**：R-F-005, R-F-009, R-F-010, R-C-001
- **Story**：STORY-038-05, STORY-038-08, STORY-038-09
- **验收**：SCN-038-POS-005/008, PRE-001
- **前置**：Slice 1

## Slice 4：环回 env-file 导出（M4）

- **范围**：exporter 突破 MVP + SW 节点导出 + case_runner 兼容。
- **需求**：R-F-006, R-F-007, R-F-008, R-C-005, R-C-006
- **Story**：STORY-038-06, STORY-038-07
- **验收**：SCN-038-POS-006/007, NEG-005, BND-003/004
- **前置**：Slice 1

## Slice 5：多实例 TRex（M5）

- **范围**：physical_pool 消费多实例字段 + pool_merge 归并 + exporter 导出多实例 + interface_kind 透传。
- **需求**：R-F-013, R-F-014, R-F-015, R-F-016, R-F-017
- **Story**：STORY-038-13
- **验收**：SCN-038-POS-010/011
- **前置**：Slice 1

## Slice 6：验证与集成（M6）

- **范围**：trex 双向发流验证 + ptm-te 编排集成 + 拓扑建模文档。
- **需求**：R-F-012, 集成
- **Story**：STORY-038-10, STORY-038-11, STORY-038-12
- **验收**：SCN-038-POS-009, AUTH-002
- **前置**：Slice 2, Slice 3, Slice 4, Slice 5
