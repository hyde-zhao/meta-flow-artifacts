---
checkpoint_id: "CP7"
checkpoint_name: "验证完成检查"
type: "auto"
status: "PASS_WITH_RISK"
story_id: "STORY-038-08"
story_slug: "devices-sw-schema"
cr_id: "CR-038"
wave: 1
design_evidence_type: "waived"
validation_mode: "mixed"
executed_by: "meta-qa"
executed_at: "2026-08-15"
---

# CP7 验证完成检查 — STORY-038-08 devices.yaml SW 设备块

## Entry Criteria

| 条目 | 结果 |
|---|---|
| Story `status=ready-for-verification` | PASS |
| CP6 编码完成门 PASS | PASS |
| waived 设计证据已确认 | PASS（Story 卡片含 Waived 重访条件 + 实现证据） |

## 验证对象清单

| 对象 | 文件 | 核对结果 |
|------|------|---------|
| 顶部 Schema 字段说明新增 `switch` 块 | devices.yaml.example | 4 字段 host/role/device_type/hardware_platform 说明齐全 |
| `device_groups` 示例 4 `sw-pppoe-server` | devices.yaml.example | `switch.host=10.113.55.139` 平铺，`role=pppoe-server`，`device_type=ruijie-B`，`hardware_platform=RG-SW` |

## 验证追踪矩阵

| 场景 | 结果 |
|------|------|
| SCN-038-POS-008（SW 块归并） | PASS（dry-run 对照 `_merge_sw_pc` 命中 switch.host 回填） |
| 验收标准「模板含 node_type: SW + role」 | 语义差（见下），实现口径为 switch 块 + 平铺 host，无 node_type 字段 |

## 设计契约验证

- 块键名 `switch`（非 `sw`）：匹配 `_merge_sw_pc` 的 `block_key={'SW':'switch','PC':'client'}`。
- 平铺 `host`（非 `management.host`）：与 firewall.host/tg.host 对称。
- 无凭据明文：switch 块仅元数据，无 ssh/web/telnet 连接块、无 password。
- **语义差**：R-F-009 需求原文「device_groups.sw 块（host/port/user/password 占位 + brand）」+ S08 验收标准「模板含 node_type: SW」反映 pool 侧 schema，与实现 devices.yaml `switch` 块（host/role/device_type/hardware_platform）字段集/块键不一致。实现按 S02 口径正确落地，差异留痕待 CR 回写。

## 分层验证结果

- review：模板 switch 块 4 字段 + 平铺 host 核对通过。
- dry-run 对照：真实 `pool_merge._merge_sw_pc` + 真实模板命中 `switch.host=10.113.55.139` 回填 device_type/hardware_platform，source='merged'。
- 真机：N/A。

## 功能 / 异常 / 回归 / 集成

- 功能：SW 设备块作为 layer1 模板示例，供 pool_merge 归并。
- 异常：N/A（模板示例，无校验逻辑）。
- 回归：device_groups 既有 3 示例未破坏（仅新增第 4 示例 + schema 说明）。
- 集成：与 S02 `_merge_sw_pc` 匹配逻辑对齐。

## 非功能

- 无凭据明文；无 schema 校验新增（符合 waived 边界）。

## 缺陷 / 问题

| 项 | 说明 |
|----|------|
| 命名口径开放项（LCQ-CR038-DOC-SW-BLOCK-NAMING） | R-F-009 原文/验收措辞（pool 侧）vs 实现（switch 块）语义差，需 host-orchestrator 走 CR 回写 |
| Return Packet 缺失 | 用 Story 卡片实现证据 + CP6 检查作等价证据 |

## 剩余风险

- 命名口径回写（scope 决策项）——R-F-009 字段集（host/port/user/password/brand）与实现（host/role/device_type/hardware_platform）实质差异，需确认需求原文是否修订。

## 阶段决策

**PASS_WITH_RISK** —— 实现按 S02 口径正确落地，dry-run 归并对照通过，无凭据；命名口径语义差汇入 CP8（走 CR 回写）。路由：host-orchestrator。
