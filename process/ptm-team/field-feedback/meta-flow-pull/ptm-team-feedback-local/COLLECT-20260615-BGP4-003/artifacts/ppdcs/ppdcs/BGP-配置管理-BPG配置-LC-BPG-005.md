# BGP4+ D-Data 等价类与边界值分析 — LC-BPG-005：基于不同接口类型建立邻居

> 生成时间: 2026-06-12 | 设计 Skill: data-design
> 上游输入: logic-cases.md §LC-BPG-005, ppdcs-annotation.md §M1, directory-structure.md
> PPDCS 主特征: D-Data + S-State
> 本文件仅覆盖 D-Data 维度的等价类/边界值分析（接口类型枚举 + MTU）；S-State 维度由 state-design 覆盖。

---

## 一、输入对齐

| 输入来源 | 关键信息 |
|---------|---------|
| design-plan.md | 缺省，以用户直接指定的 factor 定义 + logic-cases.md 为设计输入 |
| design-planner-reasoning.md | 缺省 |
| logic-cases.md | LC-BPG-005 P1~P6，因子: 接口类型/IPv6地址/MTU |
| ppdcs-annotation.md | M1 主轴 D-Data，辅 S-State |
| directory-structure.md | 四级=配置管理, 五级=BPG配置, SR-007, SR-007c |
| 用户直接输入 | 接口类型(物理口/子接口/聚合口/隧道口/BVI口), MTU(1500/1280/1279) |

### 拓扑绑定旁路

接口类型枚举为逻辑因子；具体接口名（GE0_1、GE0_1.100、bond0、tunnel0、bvi1）作为 PC 物化时的拓扑绑定，不参与等价类/边界值分析。

---

## 二、Factor Catalog

| factor_id | 数据项 | 值域摘要 | data_type | source_ref | fact_status | confirmation_gap_refs |
|-----------|-------|---------|-----------|------------|-------------|------------------------|
| FAC-BGP6-IF-TYPE | 接口类型 | enum(物理口, 子接口, 聚合口, 隧道口, BVI口) | enum | LC-BPG-005, TP-M1-019~021, BGP用例54-57 | confirmed | — |
| FAC-BGP6-IF-MTU | 接口MTU | {1500, 1280, 1279} | uint16 | LC-BPG-005, TP-M1-022, BGP用例67 | confirmed | — |

---

## 三、Factor 取值定义

### FAC-BGP6-IF-TYPE（接口类型）

| 属性 | 值 |
|------|---|
| 数据类型 | enum |
| 合法值 | `物理口`（如 GE0_1）、`子接口`（如 GE0_1.100）、`聚合口`（如 bond0）、`隧道口`（如 tunnel0）、`BVI口`（如 bvi1） |
| 非法值 | 其他不支持的接口类型（如 Loopback、Vlanif、Tunnel6in4） |
| 空值 | 空串或缺失 → 拒绝 |

### FAC-BGP6-IF-MTU（接口MTU）

| 属性 | 值 |
|------|---|
| 数据类型 | uint16 |
| 有效典型值 | `1500`（以太网标准MTU） |
| 有效边界值 | `1280`（IPv6 最小 MTU） |
| 无效边界值 | `1279`（< IPv6 最小 MTU，预期 MSS 异常） |
| 无效典型值 | `0`、`-1`、非整数 |
| 默认值 | 无（由具体接口类型决定） |

**IPv6 MTU 约束**：RFC 2460 规定 IPv6 链路最小 MTU = 1280，低于此值的接口可能导致 MSS 协商异常、分片问题或邻居不可达。

---

## 四、等价类与边界值分析

### FAC-BGP6-IF-TYPE

| value_class_id | 取值/范围 | class_type | boundary_role | 预期结果 | source_refs | fact_status |
|:---|---------|:---:|:---:|---------|-------------|:---:|
| EVP-IFTYPE-01 | `物理口` | valid-typical | typical | 邻居 Established，正常转发 | LC-BPG-005 P1, BGP用例54-57 | confirmed |
| EVP-IFTYPE-02 | `子接口` | valid-typical | typical | 邻居 Established，VLAN 隔离 | LC-BPG-005 P2, TP-M1-019 | confirmed |
| EVP-IFTYPE-03 | `聚合口` | valid-typical | typical | 邻居 Established，成员口切换保持 | LC-BPG-005 P3, TP-M1-020 | confirmed |
| EVP-IFTYPE-04 | `隧道口` | valid-typical | typical | 邻居 Established，隧道封装正常 | LC-BPG-005 P4, TP-M1-021 | confirmed |
| EVP-IFTYPE-05 | `BVI口` | valid-typical | typical | 邻居 Established，桥接转发正常 | LC-BPG-005 P5 | confirmed |
| IVP-IFTYPE-01 | `Loopback` | invalid | unsupported-type | [待确认] 环回口是否支持建立BGP邻居 | inferred | needs-confirmation |
| IVP-IFTYPE-02 | `Vlanif` | invalid | unsupported-type | [待确认] Vlanif 是否支持 | inferred | needs-confirmation |
| IVP-IFTYPE-03 | `""` (空串) | invalid | empty | 拒绝 | inferred | confirmed |
| IVP-IFTYPE-04 | `未知类型` | invalid | unknown-enum | 拒绝 | inferred | confirmed |

### FAC-BGP6-IF-MTU

| value_class_id | 取值/范围 | class_type | boundary_role | 预期结果 | source_refs | fact_status |
|:---|---------|:---:|:---:|---------|-------------|:---:|
| EVP-MTU-01 | `1500` | valid-typical | typical | 邻居正常 | LC-BPG-005, TP-M1-022 | confirmed |
| BVP-MTU-LOW | `1280` | valid-boundary | min-on | 邻居正常（IPv6 最小 MTU） | TP-M1-022, RFC 2460 | confirmed |
| IVP-MTU-LOW-OFF | `1279` | invalid-boundary | min-off | MSS 异常/邻居可能异常 | TP-M1-022, BGP用例67 | confirmed |
| IVP-MTU-01 | `0` | invalid | zero | 拒绝或邻居不可达 | inferred | confirmed |
| IVP-MTU-02 | `-1` | invalid | negative | 拒绝 | inferred | confirmed |
| IVP-MTU-03 | `68` | invalid-boundary | ipv4-min | [待确认] 低于 IPv4 最小(68)但设备可能接受 | inferred | needs-confirmation |
| IVP-MTU-04 | `9000` | valid-typical | jumbo | [待确认] 巨帧 MTU 是否支持 | inferred | needs-confirmation |

---

## 五、边界值表（三点法汇总）

| factor_id | min-off | min-on | typical | max-on | max-off |
|-----------|:-----:|:-----:|:-----:|:-----:|:-----:|
| FAC-BGP6-IF-MTU | 1279 | 1280 | 1500 | — | — |

注：
- FAC-BGP6-IF-TYPE 为 enum 型，不适用数值三点法，采用"合法枚举全部覆盖+非法枚举+空值"策略。
- FAC-BGP6-IF-MTU 的 max-on/max-off 取决于设备支持的 MTU 上限（如 9000 巨帧），本 LC 仅关注 IPv6 最小 MTU 边界（1280），上限边界属于设备能力规格，不在本文档范围内。

---

## 六、独立性检查

| 检查项 | 结论 | 证据 | 处理 |
|--------|:--:|------|------|
| 接口类型是否依赖 MTU | 否 | 所有 5 种接口类型均可独立配置 MTU；MTU 的 IPv6 最小值约束对所有接口类型有效 | 保持 D-Data |
| MTU 是否依赖接口类型 | 否 | MTU 值域不因接口类型变化；同一组 MTU 边界值可用于任意接口类型 | 保持 D-Data |
| 是否需跨因子交叉覆盖 | 否 | 各接口类型的 MTU 行为一致（1280 邻接正常/1279 异常），无需 pairwise | 保持 D-Data |
| 是否存在可能应回退的信号 | 否 | S-State 维度仅涉及接口状态 up/down 对邻居的影响（LC-ST-001），不改变本 LC 的 D-Data 因子独立性 | — |

**结论**：通过独立性检查，保持等价类+边界值策略。

---

## 七、Data Row 分配与 LC 叠加

| data_row_id | data_type | lc_step_ref | 分配值 | td_refs | 预期结果 | fact_status |
|:---|:---|:---|---------|:---|---------|:---|
| DR-001 | valid-typical | P1 | type=物理口, mtu=1500 | TP-M1-019 | Established | confirmed |
| DR-002 | valid-typical | P2 | type=子接口, mtu=1500 | TP-M1-019 | Established | confirmed |
| DR-003 | valid-typical | P3 | type=聚合口, mtu=1500 | TP-M1-020 | Established | confirmed |
| DR-004 | valid-typical | P4 | type=隧道口, mtu=1500 | TP-M1-021 | Established | confirmed |
| DR-005 | valid-typical | P5 | type=BVI口, mtu=1500 | LC-BPG-005 P5 | Established | confirmed |
| DR-006 | valid-boundary | P6 | type=物理口, mtu=1280 | TP-M1-022 | 邻居正常（IPv6最小MTU） | confirmed |
| DR-007 | invalid-boundary | P6 | type=物理口, mtu=1279 | TP-M1-022 | MSS 异常 | confirmed |
| DR-008 | invalid-boundary | P6 | type=子接口, mtu=1279 | TP-M1-022 | MSS 异常 | confirmed |
| DR-009 | invalid-boundary | P6 | type=隧道口, mtu=1279 | — | MSS 异常（隧道封装+小MTU） | confirmed |
| DR-010 | invalid | P6 | type=物理口, mtu=0 | inferred | 拒绝或不可达 | confirmed |
| DR-011 | invalid | P6 | type=物理口, mtu=-1 | inferred | 拒绝 | confirmed |
| DR-012 | invalid | — | type="", mtu=1500 | inferred | 拒绝 | confirmed |
| DR-013 | invalid | — | type=未知, mtu=1500 | inferred | 拒绝 | confirmed |
| DR-014 | invalid-gap | P6 | type=物理口, mtu=68 | — | [待确认] 远低于 IPv6 最小 | needs-confirmation |
| DR-015 | invalid-gap | P6 | type=物理口, mtu=9000 | — | [待确认] 巨帧支持 | needs-confirmation |

---

## 八、物理用例概要

物理用例（PC）由 `LC-BPG-005 + data_row → PC` 叠加生成，完整 16 列表输出见 `ppdcs/pc/BGP-配置管理-BPG配置-LC-BPG-005.md`。

| PC-ID | data_row | 测试场景 | 优先级 | fact_status |
|:---|:---|---------|:---:|:---|
| PC-BPG-DAT-038 | DR-001 | 物理口邻居建立 | P1 | confirmed |
| PC-BPG-DAT-039 | DR-002 | 子接口邻居建立 | P1 | confirmed |
| PC-BPG-DAT-040 | DR-003 | 聚合口邻居建立 | P1 | confirmed |
| PC-BPG-DAT-041 | DR-004 | 隧道口邻居建立 | P1 | confirmed |
| PC-BPG-DAT-042 | DR-005 | BVI口邻居建立 | P1 | confirmed |
| PC-BPG-DAT-043 | DR-006 | MTU=1280 邻居正常 | P2 | confirmed |
| PC-BPG-DAT-044 | DR-007 | MTU=1279 MSS异常（物理口） | P2 | confirmed |
| PC-BPG-DAT-045 | DR-008 | MTU=1279 MSS异常（子接口） | P2 | confirmed |
| PC-BPG-DAT-046 | DR-009 | MTU=1279 隧道口异常 | P2 | confirmed |
| PC-BPG-DAT-047 | DR-010 | MTU=0 拒绝 | P2 | confirmed |
| PC-BPG-DAT-048 | DR-011 | MTU=-1 拒绝 | P3 | confirmed |
| PC-BPG-DAT-049 | DR-012 | 接口类型=空拒绝 | P3 | confirmed |
| PC-BPG-DAT-050 | DR-013 | 接口类型=未知拒绝 | P3 | confirmed |
| PC-BPG-DAT-051 | DR-014 | MTU=68 [待确认] | P3 | needs-confirmation |
| PC-BPG-DAT-052 | DR-015 | MTU=9000 [待确认] | P3 | needs-confirmation |

---

## 九、待确认事项清单

| gap_ref | 事项 | 影响范围 | 来源 |
|---------|------|---------|------|
| LC-BPG-005-GAP-LOOPBACK | Loopback 接口是否支持建立 BGP4+ 邻居 | IF-TYPE 枚举 | IVP-IFTYPE-01 |
| LC-BPG-005-GAP-VLANIF | Vlanif 接口是否支持建立 BGP4+ 邻居 | IF-TYPE 枚举 | IVP-IFTYPE-02 |
| LC-BPG-005-GAP-MTU-68 | 远低于 IPv6 最小 MTU 的 68 值设备行为 | MTU 无效边界 | IVP-MTU-03 |
| LC-BPG-005-GAP-MTU-JUMBO | 巨帧 MTU (9000) 是否支持 | MTU 有效上限 | IVP-MTU-04 |

---

> 关联文件: `ppdcs/pc/BGP-配置管理-BPG配置-LC-BPG-005.md`（物理用例输出）
