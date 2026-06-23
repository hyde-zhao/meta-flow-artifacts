# BGP4+ D-Data 等价类与边界值分析 — LC-RT-002：BGP4+ 网络宣告

> 生成时间: 2026-06-12 | 设计 Skill: data-design
> 上游输入: logic-cases.md §LC-RT-002, ppdcs-annotation.md §M2, directory-structure.md
> PPDCS 主特征: D-Data
> 本文件仅覆盖 D-Data 维度的等价类/边界值分析（CIDR 格式 + 掩码类型）；P-Parameter 维度（M2 主轴）由 parameter-design 覆盖。

---

## 一、输入对齐

| 输入来源 | 关键信息 |
|---------|---------|
| design-plan.md | 缺省，以用户直接指定的 factor 定义 + logic-cases.md 为设计输入 |
| design-planner-reasoning.md | 缺省 |
| logic-cases.md | LC-RT-002 动作路径: POST→GET→PUT→DELETE，因子: IPv6 CIDR/掩码类型/出接口类型/接口开销值 |
| ppdcs-annotation.md | M2 主轴 P-Parameter，辅 D-Data（CIDR格式可独立验证） |
| directory-structure.md | 四级=协议协商, 五级=路由通告, SR-008, SR-008a |
| 用户直接输入 | FAC-BGP6-NETWORK-CIDR(IPv6 CIDR,拒绝::/0/ff00::/8/::1/128/::), FAC-BGP6-NETWORK-MASK(标准/非标准包含/非标准不包含) |

### 拓扑绑定旁路

LC-RT-002 的出接口类型和接口开销值属于控制因子，在 D-Data 分析中仅作为绑定信息保留；等价类/边界值分析聚焦于 CIDR 格式和掩码类型两个数据因子。

---

## 二、Factor Catalog

| factor_id | 数据项 | 值域摘要 | data_type | source_ref | fact_status | confirmation_gap_refs |
|-----------|-------|---------|-----------|------------|-------------|------------------------|
| FAC-BGP6-NETWORK-CIDR | IPv6 CIDR 网络地址 | IPv6 CIDR，拒绝 ::/0 / ff00::/8 / ::1/128 / :: | cidr | LC-RT-002, SR-008a, TP-M2-008~009 | confirmed | — |
| FAC-BGP6-NETWORK-MASK | 掩码类型 | enum(标准, 非标准包含, 非标准不包含) | enum | LC-RT-002, TP-M2-010 | confirmed | — |

---

## 三、Factor 取值定义

### FAC-BGP6-NETWORK-CIDR（IPv6 CIDR 网络地址）

| 属性 | 值 |
|------|---|
| 数据类型 | ipv6-cidr |
| 值域 | 合法 IPv6 CIDR 格式，前缀地址为单播 IPv6 |
| 有效格式 | `<ipv6_address>/<prefix_length>`，prefix 范围 1~128 |
| 拒绝项 | `::/0`（全零默认路由，不允许宣告） |
| | `ff00::/8`（组播地址段） |
| | `::1/128`（回环地址） |
| | `::`（零地址/未指定，无前缀长度或 /0） |
| 有效典型值 | `2001:db8:100::/48` |
| 有效典型值 | `2001:db8:200::/64` |
| 无效典型值 | `192.168.1.0/24`（IPv4 CIDR） |
| | `::/0`（默认路由） |
| | `ff00::/64`（组播） |
| | `::1/128`（回环） |
| | 非法 IPv6 格式 |

### FAC-BGP6-NETWORK-MASK（掩码类型）

| 属性 | 值 |
|------|---|
| 数据类型 | enum |
| 合法值 | `标准`：接口实配地址的前缀长度等于宣告前缀长度（如接口 /48，宣告 /48 → 完全匹配） |
| | `非标准包含`：接口实配地址的前缀长度 < 宣告前缀长度（如接口 /48，宣告 /40 → 宣告范围覆盖接口地址但不精确匹配） |
| | `非标准不包含`：接口实配地址的前缀长度 > 宣告前缀长度（如接口 /48，宣告 /56 → 宣告范围不等于接口地址所在的网络） |
| 行为差异 | 标准：路由正常宣告，对端学到；非标准包含：对端学到（范围覆盖）；非标准不包含：对端学不到 |
| 非法值 | 空值、其他枚举值 |

**掩码语义详解**：掩码类型关注接口实配地址与宣告 CIDR 的关系。设 DUT 接口配置 `2001:db8:100::1/48`：
- 宣告 `2001:db8:100::/48` → **标准**（前缀长度相等）
- 宣告 `2001:db8:100::/40` → **非标准包含**（/40 网络包含 /48 子网，宣告范围覆盖实际地址）
- 宣告 `2001:db8:100::/56` → **非标准不包含**（/56 是 /48 的子网，但接口地址不在 /56 内，宣告不等于实际网络）

---

## 四、等价类与边界值分析

### FAC-BGP6-NETWORK-CIDR

| value_class_id | 取值/范围 | class_type | boundary_role | 预期结果 | source_refs | fact_status |
|:---|---------|:---:|:---:|---------|-------------|:---:|
| EVP-CIDR-01 | `2001:db8:100::/48` | valid-typical | typical | 接受，对端学到路由 | TP-M2-008 | confirmed |
| EVP-CIDR-02 | `2001:db8:200::/64` | valid-typical | typical | 接受 | inferred | confirmed |
| EVP-CIDR-03 | `2001:db8:100::/128` | valid-boundary | max-prefix | 接受（主机路由宣告） | inferred | confirmed |
| EVP-CIDR-04 | `2001:db8:100::/1` | valid-boundary | min-prefix | [待确认] 极短前缀是否允许 | inferred | needs-confirmation |
| IVP-CIDR-01 | `::/0` | invalid | default-route | 拒绝 | TP-M2-009, SR-008a | confirmed |
| IVP-CIDR-02 | `ff00::/64` | invalid | multicast | 拒绝（组播 ff00::/8） | TP-M2-009, SR-008a | confirmed |
| IVP-CIDR-03 | `::1/128` | invalid | loopback | 拒绝 | TP-M2-009, SR-008a | confirmed |
| IVP-CIDR-04 | `::` | invalid | zero-address | 拒绝 | TP-M2-009, SR-008a | confirmed |
| IVP-CIDR-05 | `192.168.1.0/24` | invalid | ipv4-cidr | 拒绝，IPv4 格式 | inferred | confirmed |
| IVP-CIDR-06 | `2001:db8:100::` (无前缀) | invalid | missing-prefix | 拒绝，缺少前缀长度 | inferred | confirmed |
| IVP-CIDR-07 | `2001:db8:100::/129` | invalid | prefix-overflow | 拒绝，前缀溢出 (max=128) | inferred | confirmed |
| IVP-CIDR-08 | `2001:db8:100::/0` | invalid | zero-prefix | 拒绝 | inferred | confirmed |
| IVP-CIDR-09 | `2001:db8:100::/abc` | invalid | nan-prefix | 拒绝，前缀非数字 | inferred | confirmed |
| IVP-CIDR-10 | `""` (空串) | invalid | empty | 拒绝 | inferred | confirmed |
| IVP-CIDR-11 | `fe80::1/64` | invalid | link-local | [待确认] 链路本地地址是否允许宣告 | inferred | needs-confirmation |

### FAC-BGP6-NETWORK-MASK

| value_class_id | 取值/范围 | class_type | boundary_role | 预期结果 | source_refs | fact_status |
|:---|:---|:---:|:---:|---------|-------------|:---:|
| EVP-MASK-01 | `标准` | valid-typical | typical | 路由正常宣告，对端学到 | TP-M2-008, TP-M2-010 | confirmed |
| EVP-MASK-02 | `非标准包含` | valid-typical | typical | 对端学到（范围覆盖） | TP-M2-010 | confirmed |
| EVP-MASK-03 | `非标准不包含` | valid-typical | typical | 对端学不到 | TP-M2-010 | confirmed |
| IVP-MASK-01 | `""` (空串) | invalid | empty | 拒绝 | inferred | confirmed |
| IVP-MASK-02 | `其他枚举` | invalid | unknown-enum | 拒绝 | inferred | confirmed |

---

## 五、边界值表

### FAC-BGP6-NETWORK-CIDR（前缀长度边界）

| factor_id | min-off | min-on | typical | max-on | max-off |
|-----------|:-----:|:-----:|:-----:|:-----:|:-----:|
| NETWORK-CIDR prefix | 0 | 1 | 48 | 128 | 129 |

### FAC-BGP6-NETWORK-MASK

| boundary_role | 取值 | 说明 |
|:---|---------|------|
| typical | 标准 | 接口前缀=宣告前缀，正常宣告 |
| valid-alt | 非标准包含 | 宣告前缀短于接口前缀（范围覆盖），对端学到 |
| valid-alt | 非标准不包含 | 宣告前缀长于接口前缀（子网不匹配），对端学不到 |

注：掩码类型为 enum 语义型，不适用数值三点法，采用"D-Data 枚举型：合法枚举全部覆盖+非法枚举+空值"策略。

---

## 六、独立性检查

| 检查项 | 结论 | 证据 | 处理 |
|--------|:--:|------|------|
| CIDR 格式是否依赖掩码类型 | 否 | CIDR 格式校验（地址合法性+前缀范围）独立于掩码类型语义 | 保持 D-Data |
| 掩码类型是否依赖 CIDR 值 | 语义上有对应关系，但作为独立枚举值不依赖 | 掩码类型的三个取值（标准/包含/不包含）独立有效；具体判断需运行时比较接口实配地址与宣告CIDR | 保持 D-Data |
| 是否需跨因子交叉覆盖 | 部分（掩码类型需与 CIDR 值配合验证） | 每种掩码类型需配一个具体的 CIDR 宣告值来验证行为 | D-Data 中对每种掩码类型分配不同 CIDR data_row 即可，不需要 pairwise |
| 是否存在可能应回退的信号 | 否 | 本 LC 的 D-Data 聚焦 CIDR 合法性和掩码语义独立性；P-Parameter 覆盖引入协议×MED 组合，不在本文件范围 | — |

**结论**：通过独立性检查，保持等价类+边界值策略。掩码类型的三种取值各自独立验证。

---

## 七、Data Row 分配与 LC 叠加

| data_row_id | data_type | lc_step_ref | 分配值 | td_refs | 预期结果 | fact_status |
|:---|:---|:---|---------|:---|---------|:---|
| DR-001 | valid-typical | P1-POST | cidr=2001:db8:100::/48, mask=标准 | TP-M2-008 | 宣告成功，对端学到 | confirmed |
| DR-002 | valid-typical | P1-GET | cidr=2001:db8:100::/48, mask=标准 | TP-M2-008 | GET 验证宣告条目 | confirmed |
| DR-003 | valid-typical | P1-PUT | cidr=2001:db8:100::/48, mask=标准 | — | PUT 修改成功 | confirmed |
| DR-004 | valid-typical | P3 | cidr=2001:db8:100::/48, mask=非标准包含(/40) | TP-M2-010 | 对端学到（范围覆盖） | confirmed |
| DR-005 | valid-typical | P3 | cidr=2001:db8:100::/48, mask=非标准不包含(/56) | TP-M2-010 | 对端学不到 | confirmed |
| DR-006 | valid-boundary | P1 | cidr=2001:db8:100::/128, mask=标准 | inferred | 主机路由宣告成功 | confirmed |
| DR-007 | valid-boundary | P1 | cidr=2001:db8:100::/1, mask=非标准包含 | inferred | [待确认] 极短前缀 | needs-confirmation |
| DR-008 | invalid | P2 | cidr=::/0, mask=标准 | TP-M2-009 | 拒绝 | confirmed |
| DR-009 | invalid | P2 | cidr=ff00::/64, mask=标准 | TP-M2-009 | 拒绝 | confirmed |
| DR-010 | invalid | P2 | cidr=::1/128, mask=标准 | TP-M2-009 | 拒绝 | confirmed |
| DR-011 | invalid | P2 | cidr=::, mask=标准 | TP-M2-009 | 拒绝 | confirmed |
| DR-012 | invalid | P2 | cidr=192.168.1.0/24, mask=标准 | inferred | 拒绝（IPv4） | confirmed |
| DR-013 | invalid | P2 | cidr=2001:db8:100::/129, mask=标准 | inferred | 拒绝（前缀溢出） | confirmed |
| DR-014 | invalid | P2 | cidr=2001:db8:100::, mask=标准 | inferred | 拒绝（缺前缀） | confirmed |
| DR-015 | invalid | P2 | cidr=2001:db8:100::/abc, mask=标准 | inferred | 拒绝（NaN前缀） | confirmed |
| DR-016 | invalid | P2 | cidr=2001:db8:100::, mask="" | inferred | 拒绝（空掩码类型） | confirmed |
| DR-017 | invalid-gap | P2 | cidr=fe80::1/64, mask=标准 | — | [待确认] link-local | needs-confirmation |

---

## 八、物理用例概要

物理用例（PC）由 `LC-RT-002 + data_row → PC` 叠加生成，完整 16 列表输出见 `ppdcs/pc/BGP-协议协商-路由通告-LC-RT-002.md`。

| PC-ID | data_row | 测试场景 | 优先级 | fact_status |
|:---|:---|---------|:---:|:---|
| PC-RT-DAT-001 | DR-001~003 | 标准掩码 CRUD | P1 | confirmed |
| PC-RT-DAT-002 | DR-004 | 非标准包含掩码宣告 | P1 | confirmed |
| PC-RT-DAT-003 | DR-005 | 非标准不包含掩码宣告 | P1 | confirmed |
| PC-RT-DAT-004 | DR-006 | 主机路由 /128 宣告 | P2 | confirmed |
| PC-RT-DAT-005 | DR-007 | 极短前缀 /1 [待确认] | P3 | needs-confirmation |
| PC-RT-DAT-006 | DR-008 | ::/0 拒绝 | P2 | confirmed |
| PC-RT-DAT-007 | DR-009 | ff00::/64 拒绝 | P2 | confirmed |
| PC-RT-DAT-008 | DR-010 | ::1/128 拒绝 | P2 | confirmed |
| PC-RT-DAT-009 | DR-011 | :: 拒绝 | P2 | confirmed |
| PC-RT-DAT-010 | DR-012 | IPv4 CIDR 拒绝 | P2 | confirmed |
| PC-RT-DAT-011 | DR-013 | /129 前缀溢出拒绝 | P2 | confirmed |
| PC-RT-DAT-012 | DR-014 | 缺前缀拒绝 | P2 | confirmed |
| PC-RT-DAT-013 | DR-015 | NaN 前缀拒绝 | P3 | confirmed |
| PC-RT-DAT-014 | DR-016 | 空掩码类型拒绝 | P3 | confirmed |
| PC-RT-DAT-015 | DR-017 | link-local [待确认] | P3 | needs-confirmation |

---

## 九、待确认事项清单

| gap_ref | 事项 | 影响范围 | 来源 |
|---------|------|---------|------|
| LC-RT-002-GAP-SHORT-PREFIX | 极短前缀 /1 是否允许宣告 | CIDR 前缀有效范围 | EVP-CIDR-04 |
| LC-RT-002-GAP-LINK-LOCAL | 链路本地地址 fe80::/10 是否允许宣告 | CIDR 无效值分类 | IVP-CIDR-11 |

---

> 关联文件: `ppdcs/pc/BGP-协议协商-路由通告-LC-RT-002.md`（物理用例输出）
