# BGP4+ 特性测试方案

> 版本: V1.0 | 日期: 2026-06-12 | 产品: NGFW-V6R01C03

---

## 1. 特性概述

BGP4+ 是 BGP 协议的 IPv6 扩展（RFC 4760），在 C03 版本中新增。BGP4+ 与 BGP 共用 bgpd 进程和全局配置，通过独立的 bgp6* REST API 端点管理 IPv6 邻居、网络宣告、路由引入和日志查询。

**核心功能**:
- BGP4+ 全局配置（共用 bgpconfig，独立 bgp6distance）
- IPv6 邻居管理（CRUD + MD5 认证）
- IPv6 网络宣告（CIDR 格式校验）
- 路由引入（static/ospfv3/connected + MED + 白名单过滤）
- BGP4+ 日志（邻居变化/路由更新/操作日志）

**排除**: BFD 探测、RIP 路由引入

---

## 2. 应用场景分析

### 2.1 场景总览

| 场景 ID | 场景名称 | 拓扑 | 类型 |
|:---:|---------|------|:--:|
| S01 | BGP4+ 全局配置下发 | node2 | primary |
| S04 | IBGP 邻居建立（直连） | node3 | primary |
| S05 | EBGP 邻居建立（直连） | node3 | primary |
| S06 | EBGP 多跳邻居建立 | node4 | extended |
| S07 | MD5 认证邻居 | node3 | primary |
| S08 | 邻居断联与恢复 | node3 | reliability |
| S10 | 引入直连 IPv6 路由 | node3 | primary |
| S11 | 引入静态 IPv6 路由 | node3 | primary |
| S12 | 引入 OSPFv3 路由 | node4 | extended |
| S14 | 网络宣告与通告 | node3 | primary |
| S16 | 路由选路（MED 优先） | node5 | primary |
| S19 | 等价路由转发 | node4 | extended |
| S20 | HA 主备切换 | node4 | reliability |
| S21 | 日志查看 | node2 | primary |
| S23 | 长稳运行 | node3 | reliability |
| S24 | 异构设备兼容 | node5 | extended |

### 2.2 测试拓扑

| topology_id | 构成 | 适用场景 |
|-------------|------|---------|
| node2_dut1_tg1_link3 | TG1 + DUT1 | 单机配置/日志 |
| node3_dut2_tg1_link3 | TG1 + DUT1 + DUT2 | 邻居建立/MD5/路由引入 |
| node4_dut2_tg1_sw1_link6 | TG1 + SW1 + DUT1 + DUT2 | 多跳/OSPFv3/HA |
| node4_dut2_tg1_sw2_link7 | TG1 + SW1 + SW2 + DUT1 + DUT2 | ECMP |
| node5_dut3_tg1_sw1_link6 | TG1 + SW1 + DUT1 + DUT2 + DUT3 | 多邻居选路/异构 |

---

## 3. 需求分析

| SR | 需求名称 | 覆盖状态 |
|-----|---------|:--:|
| SR-001 | RFC 4760 规范遵从 | ✅ 隐含覆盖 |
| SR-002 | BGP4+ 启用/禁用 | ✅ |
| SR-003 | AS号配置 | ✅ |
| SR-004 | 路由标识 Router ID | ✅ |
| SR-005 | IBGP/EBGP 优先级 | ✅ |
| SR-006a-d | 路由引入（静态/OSPFv3/直连/MED） | ✅ |
| SR-007 | 邻居管理 CRUD | ✅ |
| SR-008 | 网络宣告 CRUD | ✅ |
| SR-009a-d | 日志查看 | ✅ |
| SR-010 | 邻居状态机 | ✅ |
| SR-011 | 路由选路 | ✅ |
| SR-012 | 路由转发 | ✅ |

---

## 4. M 分析摘要

| M | 子模块 | PPDCS 主特征 | 测试点 |
|:--:|------|:---:|:--:|
| M1 | BPG配置 | D-Data + S-State | 18 |
| M2 | 路由通告 | P-Parameter + D-Data | 10 |
| M3 | 状态协商 | S-State | 5 |
| M4 | 路由计算 | P-Parameter | 4 |
| M5 | 路由转发 | P-Parameter + S-State | 4 |
| M6 | DFX | D-Data | 3 |

---

## 5. F 分析摘要

| 耦合关系 | 类型 | 强度 |
|---------|:--:|:--:|
| M1(BPG配置) ↔ M3(状态协商) | 顺序耦合 | strong |
| M2(路由通告) ↔ M4(路由计算) | 数据耦合 | strong |
| M3 ↔ M5(路由转发) | 顺序耦合 | strong |
| BGP4+ ↔ OSPFv3 | 数据耦合 | strong |
| BGP4+ ↔ 静态路由 | 数据耦合 | strong |
| BGP4+ ↔ HA | 容错耦合 | strong |

---

## 6. Q 分析摘要

| 质量维度 | 关联度 | 测试点 |
|---------|:---:|:--:|
| 可靠性 | 强相关 | 16 |
| 性能 | 强相关 | 7 |
| 安全性 | 强相关 | 3 |
| 兼容性 | 强相关 | 5 |
| 可维护性 | 强相关 | 3 |

---

## 7. 测试点整合

| 来源 | 测试点 |
|------|:--:|
| M 分析 | 47 |
| F 分析 | 22 |
| Q 分析 | 32 |
| **合计** | **101** |

---

## 8. 逻辑用例与物理用例

| 逻辑用例 | PPDCS 方法 | 物理用例 |
|---------|:---:|:--:|
| LC-BPG-001 全局使能与禁用 | D-Data | 9 |
| LC-BPG-002 优先级配置 | D-Data | 5 |
| LC-BPG-003 邻居 CRUD | D-Data | 6 |
| LC-BPG-004 MD5 认证邻居 | S-State | 8 |
| LC-BPG-005 接口类型邻居 | D-Data+S-State | 7 |
| LC-RT-001 路由引入 | P-Parameter | 8 |
| LC-RT-002 网络宣告 | D-Data | 6 |
| LC-ST-001 邻居状态机 | S-State | 14 |
| LC-RC-001 路由选路 | P-Parameter | 4 |
| LC-FW-001 ECMP 与负载均衡 | P-Parameter | 4 |
| LC-FW-002 路由冗余互备 | S-State | 2 |
| LC-FW-003 出接口转发 | P-Parameter | 4 |
| LC-DFX-001 日志与可维护性 | D-Data | 8 |
| LC-HA-001 HA VRRP 模式 | S-State | 4 |
| LC-HA-002 HA 开销控制 | S-State | 4 |
| LC-CMP-001 异构兼容 | P-Parameter | 7 |
| LC-REL-001 进程级可靠性 | P-Parameter | 2 |
| LC-REL-002 设备级可靠性 | P-Parameter | 3 |
| LC-REL-003 链路/邻居可靠性 | P-Parameter | 5 |
| LC-REL-004 数据面/协议可靠性 | P-Parameter | 8 |
| LC-REL-005 容量/性能边界 | P-Parameter | 7 |
| **合计 21** | | **125** |

---

## 9. 覆盖率报告

| 覆盖维度 | 覆盖率 |
|---------|:--:|
| 需求覆盖 (SR) | 17/17 (100%) |
| 场景步骤覆盖 | 85% |
| BGP Bug 回扫 | 21/21 (100%) |
| GAP 已关闭 | 18/18 (100%) |
