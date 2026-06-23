# BGP4+ D-Data 等价类与边界值分析 — LC-BPG-003：BGP4+ 邻居 CRUD

> 生成时间: 2026-06-12 | 设计 Skill: data-design
> 上游输入: logic-cases.md §LC-BPG-003, ppdcs-annotation.md §M1, directory-structure.md
> PPDCS 主特征: D-Data
> 本文件仅覆盖 D-Data 维度的等价类/边界值分析；C-Combination 维度由 combination-design 覆盖。

---

## 一、输入对齐

| 输入来源 | 关键信息 |
|---------|---------|
| design-plan.md | 缺省，以用户直接指定的 factor 定义 + logic-cases.md 为设计输入 |
| design-planner-reasoning.md | 缺省 |
| logic-cases.md | LC-BPG-003 动作路径 P1（CRUD）、P2（异常）；因子: 邻居IPv6/接口名/ebgpMultihop |
| ppdcs-annotation.md | M1 主轴 D-Data，辅 C-Combination（邻居创建组合） |
| directory-structure.md | 四级=配置管理, 五级=BPG配置, SR-007, SR-007a~g |
| 用户直接输入 | FAC-BGP6-NEIGHBOR-IP(IPv6单播,拒绝::1/ff00::/8/::), FAC-BGP6-INTERFACE(本机网卡名), FAC-BGP6-MULTIHOP(1~255,默认1) |

### 拓扑绑定旁路

LC-BPG-003 的 `topology_bindings` 未显式声明端口级绑定；接口名作为逻辑因子处理（接口存在性由预置条件保证，等价类只关注名字符串格式合法性）。

---

## 二、Factor Catalog

| factor_id | 数据项 | 值域摘要 | data_type | source_ref | fact_status | confirmation_gap_refs |
|-----------|-------|---------|-----------|------------|-------------|------------------------|
| FAC-BGP6-NEIGHBOR-IP | 邻居IPv6地址 | IPv6单播，拒绝::1 / ff00::/8 / :: | ipv6-address | LC-BPG-003, SR-007a, TP-M1-011 | confirmed | — |
| FAC-BGP6-INTERFACE | 发送接口名 | 本机网卡名字符串 | string | LC-BPG-003, SR-007c, TP-M1-014 | confirmed | — |
| FAC-BGP6-MULTIHOP | EBGP多跳数 | 1~255, 默认1 | uint8 | LC-BPG-003, SR-007d, TP-M1-015 | confirmed | — |

---

## 三、Factor 取值定义

### FAC-BGP6-NEIGHBOR-IP（邻居IPv6地址）

| 属性 | 值 |
|------|---|
| 数据类型 | ipv6-address |
| 值域 | 合法IPv6单播地址 |
| 拒绝项 | `::`（零地址/未指定） |
| | `::1`（回环地址） |
| | `ff00::/8`（组播地址段） |
| | IPv4 格式（如 `192.168.1.1`） |
| 有效典型值 | `2001:db8:1::2` |
| 有效典型值 | `fd00:192:168:1::1`（ULA） |
| 无效典型值 | `192.168.1.1`（IPv4） |
| | `::1`（回环） |
| | `ff02::1`（链路本地组播） |
| | `::`（零地址） |
| | `2001:db8::/32`（CIDR格式，非单地址） |
| | 非法十六进制格式 |

### FAC-BGP6-INTERFACE（发送接口名）

| 属性 | 值 |
|------|---|
| 数据类型 | string |
| 值域 | 本机存在且状态 UP 的网卡名 |
| 有效典型值 | `GE0_1`、`GE0_2` |
| 有效典型值 | `GE0_1.100`（子接口） |
| 无效典型值 | `""`（空串） |
| | `GE99_99`（不存在的接口） |
| | 超长字符串（>64字符） |
| | 含特殊字符（如 `GE/0*1`） |
| 默认值 | 无（必填） |

### FAC-BGP6-MULTIHOP（EBGP多跳数）

| 属性 | 值 |
|------|---|
| 数据类型 | uint8 |
| 值域 | [1, 255] |
| 默认值 | 1（即直连邻居，TTL=1） |
| 有效典型值 | 1（默认）、5 |
| 有效边界值 | 1（min-on）、255（max-on） |
| 无效边界值 | 0（min-off）、256（max-off） |
| 无效典型值 | -1、非整数 |

---

## 四、等价类与边界值分析

### FAC-BGP6-NEIGHBOR-IP

| value_class_id | 取值/范围 | class_type | boundary_role | 预期结果 | source_refs | fact_status |
|:---|---------|:---:|:---:|---------|-------------|:---:|
| EVP-NEIGH-01 | `2001:db8:1::2` | valid-typical | typical | 接受，CRUD 正常 | TP-M1-010, SR-007a | confirmed |
| EVP-NEIGH-02 | `fd00:192:168:1::1` | valid-typical | typical | 接受（ULA） | inferred | confirmed |
| EVP-NEIGH-03 | `2001:db8:2::2` | valid-typical | typical | 接受（第二个邻居） | LC-BPG-003 P1 | confirmed |
| IVP-NEIGH-01 | `::1` | invalid | loopback | 拒绝，回环地址 | TP-M1-011, SR-007a | confirmed |
| IVP-NEIGH-02 | `ff02::1` | invalid | multicast | 拒绝，组播地址(ff00::/8) | TP-M1-011, SR-007a | confirmed |
| IVP-NEIGH-03 | `::` | invalid | zero-address | 拒绝，零地址/未指定 | TP-M1-011, SR-007a | confirmed |
| IVP-NEIGH-04 | `192.168.1.1` | invalid | ipv4-format | 拒绝，IPv4 格式 | TP-M1-011 | confirmed |
| IVP-NEIGH-05 | `2001:db8:1::/64` | invalid | cidr-format | 拒绝，CIDR 格式（非单地址） | inferred | confirmed |
| IVP-NEIGH-06 | `2001:db8::1:::1` | invalid | malformed | 拒绝，非法格式 | inferred | confirmed |
| IVP-NEIGH-07 | `""` (空串) | invalid | empty | 拒绝，必填 | TP-M1-014 | confirmed |
| IVP-NEIGH-08 | `ff0e::1` | invalid | multicast-flag | 拒绝，标志位组播(ff00::/8) | SR-007a | confirmed |
| IVP-NEIGH-09 | `fe80::1` | valid-link-local | typical | [待确认] 链路本地地址是否允许 | inferred | needs-confirmation |

### FAC-BGP6-INTERFACE

| value_class_id | 取值/范围 | class_type | boundary_role | 预期结果 | source_refs | fact_status |
|:---|---------|:---:|:---:|---------|-------------|:---:|
| EVP-IFACE-01 | `GE0_1` | valid-typical | typical | 接受 | TP-M1-010 | confirmed |
| EVP-IFACE-02 | `GE0_2` | valid-typical | typical | 接受 | inferred | confirmed |
| EVP-IFACE-03 | `GE0_1.100` | valid-typical | typical | 接受（子接口） | inferred | confirmed |
| IVP-IFACE-01 | `""` (空串) | invalid | empty | 拒绝，必填 | TP-M1-014, SR-007c | confirmed |
| IVP-IFACE-02 | `GE99_99` | invalid | nonexistent | 拒绝，接口不存在 | LC-BPG-003 | confirmed |
| IVP-IFACE-03 | `A` (单字符) | invalid-boundary | min-off | [待确认] 单字符网卡名是否合法 | inferred | needs-confirmation |
| IVP-IFACE-04 | 超长字符串 (65字符) | invalid | overflow | [待确认] 接口名长度上限 | inferred | needs-confirmation |
| IVP-IFACE-05 | `GE/0*1` | invalid | special-char | 拒绝，含非法字符 | inferred | confirmed |
| IVP-IFACE-06 | `null` / 缺失 | invalid | empty | 拒绝，必填 | TP-M1-014 | confirmed |
| IVP-IFACE-07 | `          ` (纯空格) | invalid | whitespace | [待确认] 纯空白字符串行为 | inferred | needs-confirmation |

### FAC-BGP6-MULTIHOP

| value_class_id | 取值/范围 | class_type | boundary_role | 预期结果 | source_refs | fact_status |
|:---|---------|:---:|:---:|---------|-------------|:---:|
| EVP-HOP-01 | `1` | valid-typical | typical | 接受（默认，直连） | SR-007d | confirmed |
| EVP-HOP-02 | `5` | valid-typical | typical | 接受 | inferred | confirmed |
| BVP-HOP-LOW | `1` | valid-boundary | min-on | 接受 | TP-M1-015 implicit | confirmed |
| BVP-HOP-HIGH | `255` | valid-boundary | max-on | 接受 | TP-M1-015 implicit | confirmed |
| IVP-HOP-LOW-OFF | `0` | invalid-boundary | min-off | 拒绝 | TP-M1-015 | confirmed |
| IVP-HOP-HIGH-OFF | `256` | invalid-boundary | max-off | 拒绝 | TP-M1-015 | confirmed |
| IVP-HOP-01 | `-1` | invalid | negative | 拒绝 | inferred | confirmed |
| IVP-HOP-02 | `3.14` | invalid | type-error | 拒绝 | inferred | confirmed |
| IVP-HOP-03 | `null` / 缺失 | invalid | empty | 拒绝或取默认(1) | inferred | confirmed |

---

## 五、边界值表（三点法汇总）

| factor_id | min-off | min-on | typical | max-on | max-off |
|-----------|:-----:|:-----:|:-----:|:-----:|:-----:|
| FAC-BGP6-MULTIHOP | 0 | 1 | 5 | 255 | 256 |
| FAC-BGP6-NEIGHBOR-IP | :: | 2001:db8:1::1 | 2001:db8:1::2 | — | — |
| FAC-BGP6-INTERFACE | 空串 | GE0_1 | GE0_1 | — | — |

注：
- FAC-BGP6-NEIGHBOR-IP 为 IPv6 地址型，不适用数值三点法；`::` 充当禁止值（min-off 角色）。
- FAC-BGP6-INTERFACE 为字符串型，采用"空串/合法名/不存在名/超长/特殊字符"策略。

---

## 六、独立性检查

| 检查项 | 结论 | 证据 | 处理 |
|--------|:--:|------|------|
| NEIGHBOR-IP 是否依赖 INTERFACE | 否（D-Data 校验层面） | API 接收 POST 时独立校验各字段合法性；接口存在性由运行时检查 | 保持 D-Data |
| NEIGHBOR-IP 是否依赖 MULTIHOP | 否 | multihop 仅影响 TTL，不影响 IP 格式校验 | 保持 D-Data |
| INTERFACE 是否依赖 NEIGHBOR-IP | 否 | 接口名合法性独立于 IP 地址 | 保持 D-Data |
| 是否需跨因子交叉覆盖 | 部分需要（C-Combination 辅） | ppdcs-annotation M1 标注辅 C-Combination：邻居创建时 IP×AS×Interface×Multihop 多因子组合需 pairwise | D-Data 独立部分已完成；组合部分保留给 combination-design |
| 是否存在待确认交叉约束 | 否 | — | — |

**结论**：D-Data 维度通过独立性检查。辅 C-Combination 维度由 `combination-design` 另行覆盖。

---

## 七、Data Row 分配与 LC 叠加

| data_row_id | data_type | lc_step_ref | 分配值 | td_refs | 预期结果 | fact_status |
|:---|:---|:---|---------|:---|---------|:---|
| DR-001 | valid-typical | P1 | ip=2001:db8:1::2, iface=GE0_1, hop=1 | TP-M1-010 | CRUD 全流程成功 | confirmed |
| DR-002 | valid-typical | P1 | ip=fd00:192:168:1::1, iface=GE0_2, hop=5 | inferred | CRUD 成功 | confirmed |
| DR-003 | valid-boundary | P1 | ip=2001:db8:1::2, iface=GE0_1, hop=255 | TP-M1-015 | 最大 multihop 成功 | confirmed |
| DR-004 | invalid | P2 | ip=::1, iface=GE0_1, hop=1 | TP-M1-011 | 拒绝（回环） | confirmed |
| DR-005 | invalid | P2 | ip=ff02::1, iface=GE0_1, hop=1 | TP-M1-011 | 拒绝（组播） | confirmed |
| DR-006 | invalid | P2 | ip=::, iface=GE0_1, hop=1 | TP-M1-011 | 拒绝（零地址） | confirmed |
| DR-007 | invalid | P2 | ip=192.168.1.1, iface=GE0_1, hop=1 | TP-M1-011 | 拒绝（IPv4） | confirmed |
| DR-008 | invalid | P2 | ip=2001:db8:1::2, iface="", hop=1 | TP-M1-014 | 拒绝（空接口） | confirmed |
| DR-009 | invalid | P2 | ip=2001:db8:1::2, iface=GE0_1, hop=0 | TP-M1-015 | 拒绝 | confirmed |
| DR-010 | invalid | P2 | ip=2001:db8:1::2, iface=GE0_1, hop=256 | TP-M1-015 | 拒绝 | confirmed |
| DR-011 | invalid | P2 | ip=2001:db8:1::2, iface=GE99_99, hop=1 | LC-BPG-003 | 拒绝（接口不存在） | confirmed |
| DR-012 | invalid | P2 | ip=2001:db8:1::2, iface=GE0_1, hop=-1 | inferred | 拒绝 | confirmed |
| DR-013 | invalid | P2 | ip=2001:db8:1::2, iface=GE0_1, hop=null | inferred | 拒绝 | confirmed |
| DR-014 | invalid-gap | P2 | ip=fe80::1, iface=GE0_1, hop=1 | — | [待确认] link-local 地址行为 | needs-confirmation |
| DR-015 | invalid-gap | P2 | ip=2001:db8:1::2, iface=A, hop=1 | — | [待确认] 单字符接口名 | needs-confirmation |

---

## 八、物理用例概要

物理用例（PC）由 `LC-BPG-003 + data_row → PC` 叠加生成，完整 16 列表输出见 `ppdcs/pc/BGP-配置管理-BPG配置-LC-BPG-003.md`。

| PC-ID | data_row | 测试场景 | 优先级 | fact_status |
|:---|:---|---------|:---:|:---|
| PC-BPG-DAT-024 | DR-001~002 | 邻居 CRUD 正常流程 | P1 | confirmed |
| PC-BPG-DAT-025 | DR-003 | multihop max=255 | P2 | confirmed |
| PC-BPG-DAT-026 | DR-004 | 邻居 IP=::1 拒绝 | P2 | confirmed |
| PC-BPG-DAT-027 | DR-005 | 邻居 IP=ff02::1 拒绝 | P2 | confirmed |
| PC-BPG-DAT-028 | DR-006 | 邻居 IP=:: 拒绝 | P2 | confirmed |
| PC-BPG-DAT-029 | DR-007 | 邻居 IP=IPv4 格式拒绝 | P2 | confirmed |
| PC-BPG-DAT-030 | DR-008 | 接口=空串拒绝 | P2 | confirmed |
| PC-BPG-DAT-031 | DR-009 | multihop=0 拒绝 | P2 | confirmed |
| PC-BPG-DAT-032 | DR-010 | multihop=256 拒绝 | P2 | confirmed |
| PC-BPG-DAT-033 | DR-011 | 接口不存在拒绝 | P2 | confirmed |
| PC-BPG-DAT-034 | DR-012 | multihop=-1 拒绝 | P3 | confirmed |
| PC-BPG-DAT-035 | DR-013 | multihop=null 拒绝 | P3 | confirmed |
| PC-BPG-DAT-036 | DR-014 | link-local addr [待确认] | P3 | needs-confirmation |
| PC-BPG-DAT-037 | DR-015 | 单字符接口名 [待确认] | P3 | needs-confirmation |

---

## 九、待确认事项清单

| gap_ref | 事项 | 影响范围 | 来源 |
|---------|------|---------|------|
| LC-BPG-003-GAP-LL | 链路本地地址 fe80::/10 是否允许作为邻居 IP | NEIGHBOR-IP 有效值范围 | IVP-NEIGH-09 |
| LC-BPG-003-GAP-IFACE-1CHAR | 单字符接口名是否合法（如 "A"） | INTERFACE 最小长度 | IVP-IFACE-03 |
| LC-BPG-003-GAP-IFACE-LEN | 接口名字符串最大长度限制 | INTERFACE 边界值 | IVP-IFACE-04 |
| LC-BPG-003-GAP-IFACE-WS | 纯空白字符串是否被 trim 或拒绝 | INTERFACE 无效值行为 | IVP-IFACE-07 |

---

> 关联文件: `ppdcs/pc/BGP-配置管理-BPG配置-LC-BPG-003.md`（物理用例输出）
