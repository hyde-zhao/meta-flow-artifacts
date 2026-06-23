# BGP4+ D-Data 等价类与边界值分析 — LC-BPG-001：BGP4+ 全局使能与禁用

> 生成时间: 2026-06-12 | 设计 Skill: data-design
> 上游输入: logic-cases.md §LC-BPG-001, ppdcs-annotation.md §M1, directory-structure.md
> PPDCS 主特征: D-Data | 辅特征: S-State
> 注意: 本文件仅覆盖 D-Data 维度的等价类/边界值分析；S-State 维度由 state-design 覆盖。

---

## 一、输入对齐

| 输入来源 | 关键信息 |
|---------|---------|
| design-plan.md | 缺省（process/plan/ 为空），以用户直接指定的 factor 定义 + logic-cases.md 为设计输入 |
| design-planner-reasoning.md | 缺省 |
| logic-cases.md | LC-BPG-001 动作路径 P1~P5, 因子-取值表, 覆盖 TP-M1-001~007, TP-M1-023, TP-Q-REL-003~004 |
| ppdcs-annotation.md | M1 主 D-Data, 辅 S-State / C-Combination |
| directory-structure.md | 四级=配置管理, 五级=BPG配置, SR-002~004 |
| 用户直接输入 | FAC-BGP-ENABLE(enum), FAC-BGP-ASN(1~4294967295), FAC-BGP-ROUTERID(合法IPv4≠0.0.0.0) |

### 拓扑绑定旁路

LC-BPG-001 未消费显式 `topology_bindings`；本 LC 的全局配置操作不涉及具体端口物化。

---

## 二、Factor Catalog

| factor_id | 数据项 | 值域摘要 | data_type | source_ref | fact_status | confirmation_gap_refs |
|-----------|-------|---------|-----------|------------|-------------|------------------------|
| FAC-BGP-ENABLE | BGP4+全局使能开关 | enum(true, false) | enum | LC-BPG-001, SR-002 | confirmed | — |
| FAC-BGP-ASN | AS号 | 1~4294967295 | uint32 | LC-BPG-001, SR-003, TP-M1-003~005 | confirmed | — |
| FAC-BGP-ROUTERID | Router ID | 合法IPv4(≠0.0.0.0) | ipv4 | LC-BPG-001, SR-004, TP-M1-006 | confirmed | — |

---

## 三、Factor 取值定义

### FAC-BGP-ENABLE（BGP4+全局使能）

| 属性 | 值 |
|------|---|
| 数据类型 | enum |
| 合法值 | `true`（使能）、`false`（禁用） |
| 默认值 | `false`（未使能） |
| 非法值 | 非布尔值（字符串、数字等） |
| 空值行为 | 缺省字段 → 保持当前值或拒绝 |

### FAC-BGP-ASN（AS号）

| 属性 | 值 |
|------|---|
| 数据类型 | uint32 |
| 值域 | [1, 4294967295] |
| 有效典型值 | 100（私有AS示例） |
| 有效边界值 | 1（min）、4294967295（max） |
| 无效边界值 | 0（min-off）、4294967296（max-off） |
| 无效典型值 | -1（负数）、非整数类型 |
| 默认值 | 无默认值（必填，创建时） |
| 精度/步长 | 1（整数） |

### FAC-BGP-ROUTERID（Router ID）

| 属性 | 值 |
|------|---|
| 数据类型 | ipv4-address |
| 值域 | 合法单播IPv4地址，排除 0.0.0.0 |
| 有效典型值 | `1.1.1.1` |
| 有效边界值 | 略（IPv4范围天然包含） |
| 无效边界值 | `0.0.0.0`（禁止值） |
| 无效典型值 | `224.0.0.1`（组播地址）、`255.255.255.255`（广播）、非法格式字符串 |
| 默认值 | 无默认值（必填） |
| 格式约束 | 点分十进制，四段 0~255 |

---

## 四、等价类与边界值分析

### FAC-BGP-ENABLE

| value_class_id | 取值/范围 | class_type | boundary_role | 预期结果 | source_refs | fact_status |
|:---|---------|:---:|:---:|---------|-------------|:---:|
| EVP-ENABLE-01 | `true` | valid-typical | typical | BGP4+ 全局使能成功 | TP-M1-001, SR-002 | confirmed |
| EVP-ENABLE-02 | `false` | valid-typical | typical | BGP4+ 全局禁用成功，邻居断开 | TP-M1-002, SR-002 | confirmed |
| IVP-ENABLE-01 | `"yes"` (字符串) | invalid | type-error | 参数类型错误，拒绝 | inferred | confirmed |
| IVP-ENABLE-02 | `1` (数字) | invalid | type-error | 参数类型错误，拒绝 | inferred | confirmed |
| IVP-ENABLE-03 | `null` / 缺失 | invalid | empty | 必填校验，拒绝或取默认值 | inferred | confirmed |
| IVP-ENABLE-04 | `"True"` (大小写) | invalid | format-error | 布尔值大小写/格式错误，拒绝 | inferred | needs-confirmation |

### FAC-BGP-ASN

| value_class_id | 取值/范围 | class_type | boundary_role | 预期结果 | source_refs | fact_status |
|:---|---------|:---:|:---:|---------|-------------|:---:|
| EVP-ASN-01 | `100` | valid-typical | typical | 接受，正常配置 | TP-M1-001 | confirmed |
| BVP-ASN-LOW | `1` | valid-boundary | min-on | 接受，最小ASN | TP-M1-003 | confirmed |
| BVP-ASN-HIGH | `4294967295` | valid-boundary | max-on | 接受，最大ASN | TP-M1-004 | confirmed |
| IVP-ASN-LOW-OFF | `0` | invalid-boundary | min-off | 拒绝，ASN 不能为 0 | TP-M1-005 | confirmed |
| IVP-ASN-HIGH-OFF | `4294967296` | invalid-boundary | max-off | 拒绝，超出 uint32 范围 | TP-M1-005 | confirmed |
| IVP-ASN-01 | `-1` | invalid | negative | 拒绝，负数 | inferred | confirmed |
| IVP-ASN-02 | `3.14` (浮点) | invalid | type-error | 拒绝，非整数 | inferred | confirmed |
| IVP-ASN-03 | `"abc"` (字符串) | invalid | type-error | 拒绝，类型错误 | inferred | confirmed |

### FAC-BGP-ROUTERID

| value_class_id | 取值/范围 | class_type | boundary_role | 预期结果 | source_refs | fact_status |
|:---|---------|:---:|:---:|---------|-------------|:---:|
| EVP-RID-01 | `1.1.1.1` | valid-typical | typical | 接受 | TP-M1-001 | confirmed |
| EVP-RID-02 | `192.168.1.1` | valid-typical | typical | 接受 | inferred | confirmed |
| IVP-RID-01 | `0.0.0.0` | invalid | forbidden-value | 拒绝 | TP-M1-006, SR-004 | confirmed |
| IVP-RID-02 | `224.0.0.1` | invalid | multicast | 拒绝（组播地址） | LC-BPG-001 | confirmed |
| IVP-RID-03 | `255.255.255.255` | invalid | broadcast | 拒绝（广播地址） | inferred | confirmed |
| IVP-RID-04 | `127.0.0.1` | invalid | loopback | [待确认] 回环地址是否可作 Router ID | inferred | needs-confirmation |
| IVP-RID-05 | `"abc.def.ghi.jkl"` | invalid | format-error | 拒绝，非法格式 | inferred | confirmed |
| IVP-RID-06 | `1.1.1.256` | invalid | segment-overflow | 拒绝，段值溢出 | inferred | confirmed |
| IVP-RID-07 | `1.1.1` | invalid | format-error | 拒绝，不足四段 | inferred | confirmed |

---

## 五、边界值表（三点法汇总）

| factor_id | min-off | min-on | typical | max-on | max-off |
|-----------|:-----:|:-----:|:-----:|:-----:|:-----:|
| FAC-BGP-ASN | 0 | 1 | 100 | 4294967295 | 4294967296 |
| FAC-BGP-ROUTERID | 0.0.0.0 | 1.1.1.1 | 1.1.1.1 | — | — |

注：
- FAC-BGP-ENABLE 为 enum 型，不适用数值三点法，采用"合法枚举+非法枚举+空值"策略。
- FAC-BGP-ROUTERID 为 IPv4 地址型，min-on/max-on 概念不适用；`0.0.0.0` 作为禁止值充当 min-off 角色。

---

## 六、独立性检查

| 检查项 | 结论 | 证据 | 处理 |
|--------|:--:|------|------|
| enable 是否依赖 ASN 的值 | 否 | enable=true/false 均为独立操作；ASN 有效范围不因 enable 状态改变 | 保持 D-Data |
| ASN 是否依赖 Router ID 的值 | 否 | ASN 与 Router ID 各自独立校验；无 IF/THEN 规则依赖 | 保持 D-Data |
| Router ID 是否依赖 ASN 的值 | 否 | Router ID 合法性校验独立于 ASN | 保持 D-Data |
| 是否需交叉覆盖 | 否（D-Data 层面） | reasoning 排除 Combination 作为主轴；M1 辅 C-Combination 仅适用于邻居创建（LC-BPG-003），非本 LC | 保持 D-Data |
| 是否存在待确认交叉约束 | 否 | 无 | — |
| 是否存在可能应回退 Parameter/Combination 的信号 | 否 | ppdcs-annotation 确认 M1 主轴 D-Data；本 LC 三个因子可独立验证 | — |

**结论**：LC-BPG-001 的 D-Data 维度通过独立性检查，保持等价类+边界值策略。

---

## 七、Data Row 分配与 LC 叠加

| data_row_id | data_type | lc_step_ref | 分配值 | td_refs | 预期结果 | fact_status |
|:---|:---|:---|---------|:---|---------|:---|
| DR-001 | valid-typical | P1-Step1 | enable=true, asn=100, rid=1.1.1.1 | TP-M1-001 | PUT 成功，三层一致 | confirmed |
| DR-002 | valid-typical | P1-Step3 | enable=false, asn=100, rid=1.1.1.1 | TP-M1-002 | PUT 成功，邻居断开 | confirmed |
| DR-003 | valid-typical | P1-Step5 | enable=true, asn=100, rid=1.1.1.1 | TP-M1-001 | 重新使能恢复 | confirmed |
| DR-004 | valid-boundary | P1 | enable=true, asn=1, rid=1.1.1.1 | TP-M1-003 | 最小 ASN 成功 | confirmed |
| DR-005 | valid-boundary | P1 | enable=true, asn=4294967295, rid=1.1.1.1 | TP-M1-004 | 最大 ASN 成功 | confirmed |
| DR-006 | invalid-boundary | P2 | asn=0, rid=1.1.1.1 | TP-M1-005 | 拒绝 | confirmed |
| DR-007 | invalid-boundary | P2 | asn=4294967296, rid=1.1.1.1 | TP-M1-005 | 拒绝 | confirmed |
| DR-008 | invalid | P2 | rid=0.0.0.0, asn=100 | TP-M1-006 | 拒绝 | confirmed |
| DR-009 | invalid | P2 | rid=224.0.0.1, asn=100 | LC-BPG-001 | 拒绝 | confirmed |
| DR-010 | invalid | P2 | asn=-1, rid=1.1.1.1 | inferred | 拒绝 | confirmed |
| DR-011 | invalid | P2 | enable=null, asn=100 | inferred | 拒绝或取默认 | confirmed |
| DR-012 | invalid | P2 | rid="abc", asn=100 | inferred | 拒绝 | confirmed |
| DR-013 | invalid-gap | P2 | rid=127.0.0.1 | — | [待确认] | needs-confirmation |

**叠加规则**：LC-BPG-001 的动作路径 P1（使能→禁用→恢复）作为序列框架，DR-001~003 按序列注入；无效值一次一个隔离（每 data row 仅一个因子取无效值，其余取有效值）。

---

## 八、物理用例概要

物理用例（PC）由 `LC-BPG-001 + data_row → PC` 叠加生成，完整 16 列表输出见 `ppdcs/pc/BGP-配置管理-BPG配置-LC-BPG-001.md`。

| PC-ID | data_row | 测试场景 | 优先级 | fact_status |
|:---|:---|---------|:---:|:---|
| PC-BPG-DAT-001 | DR-001~003 | 使能→禁用→恢复（典型值） | P1 | confirmed |
| PC-BPG-DAT-002 | DR-004 | 最小 ASN=1 使能 | P2 | confirmed |
| PC-BPG-DAT-003 | DR-005 | 最大 ASN=4294967295 使能 | P2 | confirmed |
| PC-BPG-DAT-004 | DR-006 | ASN=0 拒绝 | P2 | confirmed |
| PC-BPG-DAT-005 | DR-007 | ASN=4294967296 拒绝 | P2 | confirmed |
| PC-BPG-DAT-006 | DR-008 | Router ID=0.0.0.0 拒绝 | P2 | confirmed |
| PC-BPG-DAT-007 | DR-009 | Router ID=224.0.0.1 拒绝 | P2 | confirmed |
| PC-BPG-DAT-008 | DR-010 | ASN=-1 拒绝 | P3 | confirmed |
| PC-BPG-DAT-009 | DR-011 | enable=null 拒绝 | P3 | confirmed |
| PC-BPG-DAT-010 | DR-012 | Router ID 格式错误拒绝 | P3 | confirmed |
| PC-BPG-DAT-011 | DR-013 | Router ID=127.0.0.1 [待确认] | P3 | needs-confirmation |

---

## 九、待确认事项清单

| gap_ref | 事项 | 影响范围 | 来源 |
|---------|------|---------|------|
| LC-BPG-001-GAP-RID-LOOPBACK | Router ID 是否允许回环地址 (127.0.0.1) | RID 无效值分类 | IVP-RID-04 |
| LC-BPG-001-GAP-ENABLE-CASE | enable 布尔值大小写敏感性 (True/False 是否接受) | enable 无效值分类 | IVP-ENABLE-04 |

---

> 关联文件: `ppdcs/pc/BGP-配置管理-BPG配置-LC-BPG-001.md`（物理用例输出）
