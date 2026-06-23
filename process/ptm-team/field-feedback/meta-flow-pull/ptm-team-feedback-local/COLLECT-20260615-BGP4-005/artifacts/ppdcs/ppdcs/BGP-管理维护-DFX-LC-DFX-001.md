# BGP4+ D-Data 等价类与边界值分析 — LC-DFX-001：BGP4+ 日志与可维护性

> 生成时间: 2026-06-12 | 设计 Skill: data-design
> 上游输入: logic-cases.md §LC-DFX-001, ppdcs-annotation.md §M6, directory-structure.md
> PPDCS 主特征: D-Data
> M6 全部因子为 D-Data，无辅特征。

---

## 一、输入对齐

| 输入来源 | 关键信息 |
|---------|---------|
| design-plan.md | 缺省，以用户直接指定的 factor 定义 + logic-cases.md 为设计输入 |
| design-planner-reasoning.md | 缺省 |
| logic-cases.md | LC-DFX-001 动作路径 P1~P12，因子: 日志类型/分页参数 |
| ppdcs-annotation.md | M6 主轴 D-Data：日志类型独立查询，分页参数独立验证 |
| directory-structure.md | 四级=管理维护, 五级=DFX, SR-009a~d |
| 用户直接输入 | FAC-BGP6-LOG-TYPE(邻居变化/路由添加/路由删除/认证), FAC-BGP6-LOG-PAGE(page≥1,size≥1,默认10) |

### 拓扑绑定旁路

LC-DFX-001 的日志查询为只读 GET 操作，不涉及端口物化。

---

## 二、Factor Catalog

| factor_id | 数据项 | 值域摘要 | data_type | source_ref | fact_status | confirmation_gap_refs |
|-----------|-------|---------|-----------|------------|-------------|------------------------|
| FAC-BGP6-LOG-TYPE | 日志类型 | enum(邻居变化, 路由添加, 路由删除, 认证) | enum | LC-DFX-001, SR-009a~c | confirmed | — |
| FAC-BGP6-LOG-PAGE | 分页参数 | page≥1, size≥1, 默认 size=10 | composite(uint32, uint32) | LC-DFX-001, SR-009d, TP-M6-003 | confirmed | — |

---

## 三、Factor 取值定义

### FAC-BGP6-LOG-TYPE（日志类型）

| 属性 | 值 |
|------|---|
| 数据类型 | enum |
| 合法值 | `邻居变化`：邻居状态 Idle↔Established 变化事件 |
| | `路由添加`：BGP4+ 路由表新增 IPv6 路由 |
| | `路由删除`：BGP4+ 路由表撤销/删除 IPv6 路由 |
| | `认证`：MD5 认证成功/失败事件 |
| 默认值 | 无（查询时按类型过滤，不指定则返回全部类型） |
| 非法值 | 空串、未定义枚举值（如 "debug"、"system"） |

### FAC-BGP6-LOG-PAGE（分页参数）

| 属性 | 值 |
|------|---|
| 数据类型 | composite: {page: uint32, size: uint32} |
| 值域 | page ≥ 1, size ≥ 1 |
| 默认值 | page=1, size=10 |
| 有效典型值 | page=1, size=10（默认） |
| 有效边界值 | page=1, size=1（最小有效分页） |
| | page=1, size=100（大量条目） |
| 无效边界值 | page=0, size=10（page 小于最小） |
| | page=1, size=0（size 小于最小） |
| 无效典型值 | page=-1, size=10（负数） |
| | page=1, size=-1（负数） |
| | page=99999, size=10（page 远超实际页数，返回空列表） |
| | page=1, size=99999（size 超大） |

---

## 四、等价类与边界值分析

### FAC-BGP6-LOG-TYPE

| value_class_id | 取值/范围 | class_type | boundary_role | 预期结果 | source_refs | fact_status |
|:---|---------|:---:|:---:|---------|-------------|:---:|
| EVP-LOG-01 | `邻居变化` | valid-typical | typical | 返回邻居变化日志列表 | TP-M6-001, SR-009a | confirmed |
| EVP-LOG-02 | `路由添加` | valid-typical | typical | 返回路由添加日志列表 | TP-M6-002, SR-009b | confirmed |
| EVP-LOG-03 | `路由删除` | valid-typical | typical | 返回路由删除日志列表 | TP-M6-002, SR-009c | confirmed |
| EVP-LOG-04 | `认证` | valid-typical | typical | 返回认证日志列表 | SR-009a | confirmed |
| IVP-LOG-01 | `""` (空串) | invalid | empty | 拒绝或返回全部类型 | inferred | confirmed |
| IVP-LOG-02 | `debug` | invalid | unknown-enum | 拒绝 | inferred | confirmed |
| IVP-LOG-03 | `system` | invalid | unknown-enum | 拒绝 | inferred | confirmed |
| IVP-LOG-04 | `null` / 缺失 | invalid | empty | 返回全部类型或拒绝 | inferred | needs-confirmation |

### FAC-BGP6-LOG-PAGE

| value_class_id | 取值/范围 | class_type | boundary_role | 预期结果 | source_refs | fact_status |
|:---|---------|:---:|:---:|---------|-------------|:---:|
| EVP-PAGE-01 | page=1, size=10 | valid-typical | typical | 返回第一页 10 条 | TP-M6-003, SR-009d | confirmed |
| EVP-PAGE-02 | page=2, size=10 | valid-typical | typical | 返回第二页 | inferred | confirmed |
| BVP-PAGE-SIZE-01 | page=1, size=1 | valid-boundary | min-on | 返回 1 条 | LC-DFX-001 | confirmed |
| BVP-PAGE-PAGE-01 | page=1, size=10 | valid-boundary | min-on | 第一页（page 最小值） | TP-M6-003 | confirmed |
| BVP-PAGE-HIGH | page=99999, size=10 | valid-boundary | overflow-page | 返回空列表（超出范围不报错） | LC-DFX-001 P12 | confirmed |
| IVP-PAGE-01 | page=0, size=10 | invalid-boundary | min-off | 拒绝 | inferred | confirmed |
| IVP-PAGE-02 | page=1, size=0 | invalid-boundary | min-off | 拒绝 | inferred | confirmed |
| IVP-PAGE-03 | page=-1, size=10 | invalid | negative | 拒绝 | inferred | confirmed |
| IVP-PAGE-04 | page=1, size=-1 | invalid | negative | 拒绝 | inferred | confirmed |
| IVP-PAGE-05 | page=1, size=99999 | invalid | overflow-size | [待确认] 超大 size 是否被截断或拒绝 | inferred | needs-confirmation |
| IVP-PAGE-06 | page=1.5, size=10 | invalid | type-error | 拒绝（非整数） | inferred | confirmed |
| IVP-PAGE-07 | page="abc", size=10 | invalid | type-error | 拒绝 | inferred | confirmed |
| IVP-PAGE-08 | page=null, size=10 | invalid | empty | 拒绝或取默认 1 | inferred | confirmed |
| IVP-PAGE-09 | page=1, size=null | invalid | empty | 拒绝或取默认 10 | inferred | confirmed |

---

## 五、边界值表（三点法汇总）

| factor_id | min-off | min-on | typical | max-on | max-off |
|-----------|:-----:|:-----:|:-----:|:-----:|:-----:|
| FAC-BGP6-LOG-PAGE.page | 0 | 1 | 1 | 99999 | — |
| FAC-BGP6-LOG-PAGE.size | 0 | 1 | 10 | — | 99999 |

注：
- page 的 max-on 为实际日志条目数对应的最大页码（动态），`99999` 为远超实际页数的溢出场景，返回空列表而不报错。
- size 的 max-on 取决于系统配置的最大 page size，缺此规格信息时标记 `max-size` 为 `[待确认]`。
- FAC-BGP6-LOG-TYPE 为 enum 型，不适用三点法，采用"合法枚举全部覆盖+非法枚举+空值"策略。

---

## 六、独立性检查

| 检查项 | 结论 | 证据 | 处理 |
|--------|:--:|------|------|
| LOG-TYPE 是否依赖 LOG-PAGE | 否 | 日志类型过滤和分页是两个独立的查询维度；按类型过滤后再分页，或分页后再过滤类型，逻辑独立 | 保持 D-Data |
| LOG-PAGE 是否依赖 LOG-TYPE | 否 | 分页参数(page, size)的合法性校验不依赖日志类型取值 | 保持 D-Data |
| 是否需跨因子交叉覆盖 | 否 | M6 明确标注主轴 D-Data，两因子互不依赖 | 保持 D-Data |
| 是否存在可能应回退的信号 | 否 | reasoning 无提示 | — |

**结论**：通过独立性检查。LOG-TYPE 和 LOG-PAGE 完全独立，各自等价类+边界值即可。

---

## 七、Data Row 分配与 LC 叠加

| data_row_id | data_type | lc_step_ref | 分配值 | td_refs | 预期结果 | fact_status |
|:---|:---|:---|---------|:---|---------|:---|
| DR-001 | valid-typical | P1 | type=邻居变化, page=1,size=10 | TP-M6-001 | 返回邻居变化日志 | confirmed |
| DR-002 | valid-typical | P1 | type=路由添加, page=1,size=10 | TP-M6-002 | 返回路由添加日志 | confirmed |
| DR-003 | valid-typical | P1 | type=路由删除, page=1,size=10 | TP-M6-002 | 返回路由删除日志 | confirmed |
| DR-004 | valid-typical | P1 | type=认证, page=1,size=10 | SR-009a | 返回认证日志 | confirmed |
| DR-005 | valid-boundary | P1 | type=邻居变化, page=1,size=1 | LC-DFX-001 | 返回 1 条（最小 size） | confirmed |
| DR-006 | valid-boundary | P12 | type=邻居变化, page=100,size=10 | LC-DFX-001 P12 | 最后一页正确 | confirmed |
| DR-007 | valid-boundary | P12 | type=邻居变化, page=101,size=10 | LC-DFX-001 P12 | 返回空列表 | confirmed |
| DR-008 | valid-boundary | P12 | type=邻居变化, page=99999,size=10 | LC-DFX-001 P12 | 返回空列表（溢出不报错） | confirmed |
| DR-009 | invalid-boundary | P1 | type=邻居变化, page=0,size=10 | inferred | 拒绝 | confirmed |
| DR-010 | invalid-boundary | P1 | type=邻居变化, page=1,size=0 | inferred | 拒绝 | confirmed |
| DR-011 | invalid | P1 | type=邻居变化, page=-1,size=10 | inferred | 拒绝 | confirmed |
| DR-012 | invalid | P1 | type=邻居变化, page=1,size=-1 | inferred | 拒绝 | confirmed |
| DR-013 | invalid | P1 | type="", page=1,size=10 | inferred | 拒绝或返回全部 | confirmed |
| DR-014 | invalid | P1 | type=debug, page=1,size=10 | inferred | 拒绝 | confirmed |
| DR-015 | invalid-gap | P1 | type=邻居变化, page=1,size=99999 | — | [待确认] | needs-confirmation |
| DR-016 | invalid-gap | P1 | type=邻居变化, page=null,size=10 | — | [待确认] | needs-confirmation |

---

## 八、物理用例概要

物理用例（PC）由 `LC-DFX-001 + data_row → PC` 叠加生成，完整 16 列表输出见 `ppdcs/pc/BGP-管理维护-DFX-LC-DFX-001.md`。

| PC-ID | data_row | 测试场景 | 优先级 | fact_status |
|:---|:---|---------|:---:|:---|
| PC-DFX-DAT-001 | DR-001 | 邻居变化日志查询（默认分页） | P1 | confirmed |
| PC-DFX-DAT-002 | DR-002 | 路由添加日志查询 | P1 | confirmed |
| PC-DFX-DAT-003 | DR-003 | 路由删除日志查询 | P1 | confirmed |
| PC-DFX-DAT-004 | DR-004 | 认证日志查询 | P1 | confirmed |
| PC-DFX-DAT-005 | DR-005 | 分页 size=1（最小） | P2 | confirmed |
| PC-DFX-DAT-006 | DR-006 | 分页最后一页（page=100） | P2 | confirmed |
| PC-DFX-DAT-007 | DR-007 | 分页超范围（page=101） | P2 | confirmed |
| PC-DFX-DAT-008 | DR-008 | 分页 page=99999 溢出 | P2 | confirmed |
| PC-DFX-DAT-009 | DR-009 | page=0 拒绝 | P2 | confirmed |
| PC-DFX-DAT-010 | DR-010 | size=0 拒绝 | P2 | confirmed |
| PC-DFX-DAT-011 | DR-011 | page=-1 拒绝 | P3 | confirmed |
| PC-DFX-DAT-012 | DR-012 | size=-1 拒绝 | P3 | confirmed |
| PC-DFX-DAT-013 | DR-013 | 空日志类型 | P3 | confirmed |
| PC-DFX-DAT-014 | DR-014 | 非法日志类型 | P3 | confirmed |
| PC-DFX-DAT-015 | DR-015 | size=99999 [待确认] | P3 | needs-confirmation |
| PC-DFX-DAT-016 | DR-016 | page=null [待确认] | P3 | needs-confirmation |

---

## 九、待确认事项清单

| gap_ref | 事项 | 影响范围 | 来源 |
|---------|------|---------|------|
| LC-DFX-001-GAP-SIZE-MAX | 分页 size 最大值是多少（当前推断为 99999 可能被截断） | LOG-PAGE 边界 | IVP-PAGE-05 |
| LC-DFX-001-GAP-NULL-BEHAVIOR | null/missing 参数行为：取默认值还是拒绝 | LOG-PAGE/L-TYPE 空值行为 | IVP-PAGE-08, IVP-LOG-04 |

---

> 关联文件: `ppdcs/pc/BGP-管理维护-DFX-LC-DFX-001.md`（物理用例输出）
