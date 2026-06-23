# BGP4+ D-Data 等价类与边界值分析 — LC-BPG-002：BGP4+ 优先级配置

> 生成时间: 2026-06-12 | 设计 Skill: data-design
> 上游输入: logic-cases.md §LC-BPG-002, ppdcs-annotation.md §M1, directory-structure.md
> PPDCS 主特征: D-Data
> 本文件仅覆盖 D-Data 维度的等价类/边界值分析。

---

## 一、输入对齐

| 输入来源 | 关键信息 |
|---------|---------|
| design-plan.md | 缺省，以用户直接指定的 factor 定义 + logic-cases.md 为设计输入 |
| design-planner-reasoning.md | 缺省 |
| logic-cases.md | LC-BPG-002 动作路径: PUT bgp6distance → GET验证; 因子: EBGP优先级/IBGP优先级 |
| ppdcs-annotation.md | M1 主轴 D-Data；本 LC 优先级因子为典型 D-Data |
| directory-structure.md | 四级=配置管理, 五级=BPG配置, SR-005 |
| 用户直接输入 | FAC-BGP6-EBGP-DIST(1~255,默认20), FAC-BGP6-IBGP-DIST(1~255,默认200) |

### 拓扑绑定旁路

LC-BPG-002 的优先级配置为全局参数，不涉及端口物化。

---

## 二、Factor Catalog

| factor_id | 数据项 | 值域摘要 | data_type | source_ref | fact_status | confirmation_gap_refs |
|-----------|-------|---------|-----------|------------|-------------|------------------------|
| FAC-BGP6-EBGP-DIST | EBGP管理距离 | 1~255, 默认20 | uint8 | LC-BPG-002, SR-005 | confirmed | — |
| FAC-BGP6-IBGP-DIST | IBGP管理距离 | 1~255, 默认200 | uint8 | LC-BPG-002, SR-005 | confirmed | — |

---

## 三、Factor 取值定义

### FAC-BGP6-EBGP-DIST（EBGP管理距离）

| 属性 | 值 |
|------|---|
| 数据类型 | uint8 |
| 值域 | [1, 255] |
| 默认值 | 20 |
| 有效典型值 | 20（默认） |
| 有效边界值 | 1（min-on）、255（max-on） |
| 无效边界值 | 0（min-off）、256（max-off） |
| 无效典型值 | -1（负数）、3.14（浮点）、"abc"（字符串） |

### FAC-BGP6-IBGP-DIST（IBGP管理距离）

| 属性 | 值 |
|------|---|
| 数据类型 | uint8 |
| 值域 | [1, 255] |
| 默认值 | 200 |
| 有效典型值 | 200（默认） |
| 有效边界值 | 1（min-on）、255（max-on） |
| 无效边界值 | 0（min-off）、256（max-off） |
| 无效典型值 | -1（负数）、3.14（浮点）、"abc"（字符串） |

**特殊约束**：IBGP 管理距离应大于 EBGP 管理距离（路由选路优先级），但 API 层不强制此约束（仅接受值时独立校验），选路行为的交叉验证属于 P-Parameter 维度，不在本文档范围内。

---

## 四、等价类与边界值分析

### FAC-BGP6-EBGP-DIST

| value_class_id | 取值/范围 | class_type | boundary_role | 预期结果 | source_refs | fact_status |
|:---|---------|:---:|:---:|---------|-------------|:---:|
| EVP-EBGP-01 | `20` | valid-typical | typical | 接受，默认值 | TP-M1-008, SR-005 | confirmed |
| EVP-EBGP-02 | `100` | valid-typical | typical | 接受 | inferred | confirmed |
| BVP-EBGP-LOW | `1` | valid-boundary | min-on | 接受 | TP-M1-008 | confirmed |
| BVP-EBGP-HIGH | `255` | valid-boundary | max-on | 接受 | TP-M1-008 | confirmed |
| IVP-EBGP-LOW-OFF | `0` | invalid-boundary | min-off | 拒绝 | TP-M1-008 | confirmed |
| IVP-EBGP-HIGH-OFF | `256` | invalid-boundary | max-off | 拒绝 | TP-M1-008 | confirmed |
| IVP-EBGP-01 | `-1` | invalid | negative | 拒绝 | inferred | confirmed |
| IVP-EBGP-02 | `3.14` | invalid | type-error | 拒绝 | inferred | confirmed |
| IVP-EBGP-03 | `"abc"` | invalid | type-error | 拒绝 | inferred | confirmed |
| IVP-EBGP-04 | `null` / 缺失 | invalid | empty | 拒绝或回退默认(20) | inferred | confirmed |

### FAC-BGP6-IBGP-DIST

| value_class_id | 取值/范围 | class_type | boundary_role | 预期结果 | source_refs | fact_status |
|:---|---------|:---:|:---:|---------|-------------|:---:|
| EVP-IBGP-01 | `200` | valid-typical | typical | 接受，默认值 | TP-M1-009, SR-005 | confirmed |
| EVP-IBGP-02 | `100` | valid-typical | typical | 接受 | inferred | confirmed |
| BVP-IBGP-LOW | `1` | valid-boundary | min-on | 接受 | TP-M1-009 | confirmed |
| BVP-IBGP-HIGH | `255` | valid-boundary | max-on | 接受 | TP-M1-009 | confirmed |
| IVP-IBGP-LOW-OFF | `0` | invalid-boundary | min-off | 拒绝 | TP-M1-009 | confirmed |
| IVP-IBGP-HIGH-OFF | `256` | invalid-boundary | max-off | 拒绝 | TP-M1-009 | confirmed |
| IVP-IBGP-01 | `-1` | invalid | negative | 拒绝 | inferred | confirmed |
| IVP-IBGP-02 | `3.14` | invalid | type-error | 拒绝 | inferred | confirmed |
| IVP-IBGP-03 | `"abc"` | invalid | type-error | 拒绝 | inferred | confirmed |
| IVP-IBGP-04 | `null` / 缺失 | invalid | empty | 拒绝或回退默认(200) | inferred | confirmed |

---

## 五、边界值表（三点法汇总）

| factor_id | min-off | min-on | typical | max-on | max-off |
|-----------|:-----:|:-----:|:-----:|:-----:|:-----:|
| FAC-BGP6-EBGP-DIST | 0 | 1 | 20 | 255 | 256 |
| FAC-BGP6-IBGP-DIST | 0 | 1 | 200 | 255 | 256 |

---

## 六、独立性检查

| 检查项 | 结论 | 证据 | 处理 |
|--------|:--:|------|------|
| EBGP-DIST 是否依赖 IBGP-DIST 的值 | 否 | API 接受值时独立校验各字段；无 IF/THEN 约束 | 保持 D-Data |
| IBGP-DIST 是否依赖 EBGP-DIST 的值 | 否 | 同上 | 保持 D-Data |
| 是否需跨因子交叉覆盖 | 否 | ppdcs-annotation M1 主轴 D-Data；选路优先级的交叉规则（IBGP>EBGP 语义）属于 P-Parameter（LC-RC-001），不在本 LC | 保持 D-Data |
| 是否存在可能应回退的信号 | 否 | reasoning 排除 Parameter/Combination | — |

**结论**：通过独立性检查，保持等价类+边界值策略。

---

## 七、Data Row 分配与 LC 叠加

| data_row_id | data_type | lc_step_ref | 分配值 | td_refs | 预期结果 | fact_status |
|:---|:---|:---|---------|:---|---------|:---|
| DR-001 | valid-typical | P1 | ebgp=20, ibgp=200 | TP-M1-008~009 | PUT 成功，GET 验证均为默认值 | confirmed |
| DR-002 | valid-typical | P1 | ebgp=100, ibgp=100 | inferred | PUT 成功 | confirmed |
| DR-003 | valid-boundary | P1 | ebgp=1, ibgp=200 | TP-M1-008 | 最小 EBGP 成功 | confirmed |
| DR-004 | valid-boundary | P1 | ebgp=255, ibgp=200 | TP-M1-008 | 最大 EBGP 成功 | confirmed |
| DR-005 | valid-boundary | P1 | ebgp=20, ibgp=1 | TP-M1-009 | 最小 IBGP 成功 | confirmed |
| DR-006 | valid-boundary | P1 | ebgp=20, ibgp=255 | TP-M1-009 | 最大 IBGP 成功 | confirmed |
| DR-007 | invalid-boundary | P1 | ebgp=0, ibgp=200 | TP-M1-008 | 拒绝 | confirmed |
| DR-008 | invalid-boundary | P1 | ebgp=256, ibgp=200 | TP-M1-008 | 拒绝 | confirmed |
| DR-009 | invalid-boundary | P1 | ebgp=20, ibgp=0 | TP-M1-009 | 拒绝 | confirmed |
| DR-010 | invalid-boundary | P1 | ebgp=20, ibgp=256 | TP-M1-009 | 拒绝 | confirmed |
| DR-011 | invalid | P1 | ebgp=-1, ibgp=200 | inferred | 拒绝 | confirmed |
| DR-012 | invalid | P1 | ebgp=20, ibgp=-1 | inferred | 拒绝 | confirmed |
| DR-013 | invalid | P1 | ebgp=null, ibgp=200 | inferred | 拒绝或取默认 | confirmed |

---

## 八、物理用例概要

物理用例（PC）由 `LC-BPG-002 + data_row → PC` 叠加生成，完整 16 列表输出见 `ppdcs/pc/BGP-配置管理-BPG配置-LC-BPG-002.md`。

| PC-ID | data_row | 测试场景 | 优先级 | fact_status |
|:---|:---|---------|:---:|:---|
| PC-BPG-DAT-012 | DR-001 | 默认值 (20/200) 配置 | P1 | confirmed |
| PC-BPG-DAT-013 | DR-003 | EBGP min=1 | P2 | confirmed |
| PC-BPG-DAT-014 | DR-004 | EBGP max=255 | P2 | confirmed |
| PC-BPG-DAT-015 | DR-005 | IBGP min=1 | P2 | confirmed |
| PC-BPG-DAT-016 | DR-006 | IBGP max=255 | P2 | confirmed |
| PC-BPG-DAT-017 | DR-007 | EBGP=0 拒绝 | P2 | confirmed |
| PC-BPG-DAT-018 | DR-008 | EBGP=256 拒绝 | P2 | confirmed |
| PC-BPG-DAT-019 | DR-009 | IBGP=0 拒绝 | P2 | confirmed |
| PC-BPG-DAT-020 | DR-010 | IBGP=256 拒绝 | P2 | confirmed |
| PC-BPG-DAT-021 | DR-011 | EBGP=-1 拒绝 | P3 | confirmed |
| PC-BPG-DAT-022 | DR-012 | IBGP=-1 拒绝 | P3 | confirmed |
| PC-BPG-DAT-023 | DR-013 | EBGP=null 拒绝 | P3 | confirmed |

---

> 关联文件: `ppdcs/pc/BGP-配置管理-BPG配置-LC-BPG-002.md`（物理用例输出）
