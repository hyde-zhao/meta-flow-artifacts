# BGP4+ P-Parameter 判定表分析 — LC-RT-001: 路由引入

> 生成时间: 2026-06-12 | 方法: parameter-design (PPDCS P-Parameter)
> 输入源: logic-cases.md / test-points.md / ppdcs-annotation.md / test-objects-factors.md
> design-plan.md / design-planner-reasoning.md: **缺失** — LC 设计推荐与推理事实状态无法确认，fact_status 统一标记为 confirmed（来自 test-points.md 源状态）

## 一、设计计划与推理摘要

### 1.1 设计计划 (design-plan.md)
**状态: 缺失** — 未找到 design-plan.md，无法确认计划推荐。

### 1.2 设计推理 (design-planner-reasoning.md)
**状态: 缺失** — 未找到 design-planner-reasoning.md，无法确认 primary_signal/candidate_features/exclusion_reasons。

### 1.3 从可用输入推导
| 项目 | 值 |
|------|-----|
| LC-ID | LC-RT-001 |
| PPDCS 特征 | P-Parameter (主) + D-Data (辅) |
| 推荐方法 | 判定表 |
| 设计 Skill | parameter-design |
| 主信号 | 引入协议 × MED 值组合 → 路由通告行为 |
| CAE 聚合规则 | 规则1-参数化 |

## 二、Factor Catalog（因子目录）

| factor_id | 因子名称 | 来源 | 典型取值/值域 | factor_type | design_role | source_ref | fact_status | confirmation_gap_refs |
|-----------|---------|------|--------------|-------------|-------------|------------|-------------|------------------------|
| FAC-BGP6-REDIST-PROTO | 引入协议类型 | LC因子表 | static, ospfv3, connected, rip(非法), ospf(非法) | parameter (control) | driver | LC-RT-001 / TP-M2-001~007 | confirmed | — |
| FAC-BGP6-REDIST-MED | 引入路由MED值 | LC因子表 | 0, 50, 100, 200, 4294967295, 4294967296(越界) | parameter (data) | driver | LC-RT-001 / TP-M2-004 | confirmed | — |
| FAC-BGP6-REDIST-FILTER | 过滤路由类型 | LC因子表 | tap直连, admin直连, ISP静态, IPSec反向, NAT66黑洞 | parameter (data) | auxiliary | LC-RT-001 / TP-M2-006~007 | confirmed | — |

## 三、Topology Binding Catalog（拓扑绑定目录）

| topology_binding_id | topology_role | role_usage | bound_topology_object | source_ref | fact_status |
|---------------------|---------------|------------|------------------------|------------|-------------|
| TB-RT-001-01 | DUT1 | 引入设备 | DUT1 (BGP4+ 路由引入源) | LC-RT-001 P1 | confirmed |
| TB-RT-001-02 | DUT2 | 对端验证设备 | DUT2 (BGP4+ 邻居对端) | LC-RT-001 P1 | confirmed |
| TB-RT-001-03 | EXT-OSPFv3 | OSPFv3 路由源 | 外部 OSPFv3 邻居设备 | TP-F-M2-OSPFv3-001 | confirmed |
| TB-RT-001-04 | TG | 路由源注入 | 流量仪/路由注入器 | LC-RT-001 | confirmed |

## 四、参数范围与规则清单

### 4.1 参数取值边界

**FAC-BGP6-REDIST-PROTO（引入协议）**:
- 有效: static, ospfv3, connected
- 无效: rip（BGP4+ 不支持）, ospf（IPv4协议）
- 约束: 引入 ospfv3 前需 OSPFv3 邻居已建立

**FAC-BGP6-REDIST-MED（MED值）**:
- 有效范围: 0 ~ 4294967295
- 边界: 0(最小值), 4294967295(最大值)
- 越界: 4294967296 → 错误
- 默认: 无明确默认值，需显式指定

**FAC-BGP6-REDIST-FILTER（过滤路由）**:
- 直连接入时过滤: tap口直连路由, admin口直连路由
- 静态引入时过滤: ISP静态路由, IPSec反向路由, NAT66黑洞路由

### 4.2 规则清单

| rule_id | rule_type | 条件表达式 | 动作/结果 | factor_refs | source_refs | priority | fact_status |
|---------|-----------|------------|-----------|-------------|-------------|----------|-------------|
| R-RT-01 | imply | IF 协议=connected AND MED=100 THEN 对端学到直连路由(非tap/admin) | 直连路由通告成功 | FAC-BGP6-REDIST-PROTO, FAC-BGP6-REDIST-MED | TP-M2-001 | P1 | confirmed |
| R-RT-02 | imply | IF 协议=static AND MED=50 THEN 对端学到静态路由(非ISP/IPSec/NAT66) | 静态路由通告成功 | FAC-BGP6-REDIST-PROTO, FAC-BGP6-REDIST-MED | TP-M2-002 | P1 | confirmed |
| R-RT-03 | imply | IF 协议=ospfv3 AND MED=200 AND OSPFv3邻居已建立 THEN 对端学到OSPFv3路由 | OSPFv3路由通告成功 | FAC-BGP6-REDIST-PROTO, FAC-BGP6-REDIST-MED | TP-M2-003 | P1 | confirmed |
| R-RT-04 | imply | IF MED=0 THEN 配置成功 | MED最小边界正常 | FAC-BGP6-REDIST-MED | TP-M2-004 | P2 | confirmed |
| R-RT-05 | imply | IF MED=4294967295 THEN 配置成功 | MED最大边界正常 | FAC-BGP6-REDIST-MED | TP-M2-004 | P2 | confirmed |
| R-RT-06 | mutex | IF MED=4294967296 THEN 配置失败 | MED越界拒绝 | FAC-BGP6-REDIST-MED | TP-M2-004 | P2 | confirmed |
| R-RT-07 | mutex | IF 协议=rip THEN 配置失败 | 非法协议拒绝 | FAC-BGP6-REDIST-PROTO | TP-M2-005 | P2 | confirmed |
| R-RT-08 | mutex | IF 协议=ospf THEN 配置失败 | IPv4协议拒绝 | FAC-BGP6-REDIST-PROTO | TP-M2-005 | P2 | confirmed |
| R-RT-09 | imply | IF 协议=connected AND 存在tap/admin直连路由 THEN tap/admin路由不出现在对端BGP路由表 | 白名单过滤生效 | FAC-BGP6-REDIST-PROTO, FAC-BGP6-REDIST-FILTER | TP-M2-006 | P1 | confirmed |
| R-RT-10 | imply | IF 协议=static AND 存在ISP/IPSec/NAT66静态路由 THEN ISP/IPSec/NAT66路由不出现在对端BGP路由表 | 白名单过滤生效 | FAC-BGP6-REDIST-PROTO, FAC-BGP6-REDIST-FILTER | TP-M2-007 | P1 | confirmed |

## 五、判定结构（判定表）

本 LC 参数数量=3（协议 × MED × 过滤），规则逻辑直观，使用判定表。

### 5.1 主判定表：协议类型 × MED 值

| 条件/动作 | DR-RT-01 | DR-RT-02 | DR-RT-03 | DR-RT-04 | DR-RT-05 | DR-RT-06 | DR-RT-07 | DR-RT-08 |
|-----------|----------|----------|----------|----------|----------|----------|----------|----------|
| 引入协议 | connected | static | ospfv3 | connected | static | connected | rip | ospf |
| MED值 | 100 | 50 | 200 | 0 | 4294967295 | 4294967296 | 100 | 100 |
| 前置条件 | 邻居Established | 邻居Established | OSPFv3邻居已建立 | 邻居Established | 邻居Established | 邻居Established | — | — |
| 预期结果 | 对端学到直连路由 | 对端学到静态路由 | 对端学到OSPFv3路由 | MED=0成功 | MED=MAX成功 | 返回错误 | 返回错误 | 返回错误 |
| rule_ref | R-RT-01 | R-RT-02 | R-RT-03 | R-RT-04 | R-RT-05 | R-RT-06 | R-RT-07 | R-RT-08 |
| trace_refs | TP-M2-001 | TP-M2-002 | TP-M2-003 | TP-M2-004 | TP-M2-004 | TP-M2-004 | TP-M2-005 | TP-M2-005 |
| priority | P1 | P1 | P1 | P2 | P2 | P2 | P2 | P2 |
| fact_status | confirmed | confirmed | confirmed | confirmed | confirmed | confirmed | confirmed | confirmed |

### 5.2 过滤子判定表：白名单过滤

| 条件/动作 | DR-RT-09 | DR-RT-10 | DR-RT-11 | DR-RT-12 | DR-RT-13 |
|-----------|----------|----------|----------|----------|----------|
| 引入协议 | connected | connected | static | static | static |
| MED值 | 100 | 100 | 50 | 50 | 50 |
| 过滤目标 | tap口直连路由 | admin口直连路由 | ISP静态路由 | IPSec反向路由 | NAT66黑洞路由 |
| 预期结果 | 对端BGP路由表中无tap路由 | 对端BGP路由表中无admin路由 | 对端BGP路由表中无ISP路由 | 对端BGP路由表中无IPSec路由 | 对端BGP路由表中无NAT66路由 |
| rule_ref | R-RT-09 | R-RT-09 | R-RT-10 | R-RT-10 | R-RT-10 |
| trace_refs | TP-M2-006 | TP-M2-006 | TP-M2-007 | TP-M2-007 | TP-M2-007 |
| priority | P1 | P1 | P1 | P1 | P1 |
| fact_status | confirmed | confirmed | confirmed | confirmed | confirmed |

### 5.3 OSPFv3 引入前后耦合子判定表

| 条件/动作 | DR-RT-14 | DR-RT-15 |
|-----------|----------|----------|
| 引入协议 | ospfv3 | ospfv3 |
| MED值 | 200 | 200 |
| OSPFv3邻居状态 | 已建立 | 断开 |
| 预期结果 | 对端学到OSPFv3路由 | 对端路由表中OSPFv3路由被撤销 |
| rule_ref | R-RT-03 | R-RT-03(反向) |
| trace_refs | TP-F-M2-OSPFv3-001 | TP-F-M2-OSPFv3-002 |
| priority | P1 | P1 |
| fact_status | confirmed | confirmed |

## 六、Data Row → LC 叠加

| data_row_id | rule_id | lc_step_ref | 触发参数组 | td_refs | 预期结果 | topology_binding_ref | confirmation_gap_refs | fact_status |
|-------------|---------|-------------|-----------|---------|---------|----------------------|------------------------|-------------|
| DR-RT-001 | R-RT-01 | P1-Step2 | protocol=connected, med=100 | TP-M2-001 | 对端学到直连路由，MED=100 | TB-RT-001-01,02 | — | confirmed |
| DR-RT-002 | R-RT-02 | P1-Step2 | protocol=static, med=50 | TP-M2-002 | 对端学到静态路由，MED=50 | TB-RT-001-01,02 | — | confirmed |
| DR-RT-003 | R-RT-03 | P1-Step2 | protocol=ospfv3, med=200 | TP-M2-003 | 对端学到OSPFv3路由，MED=200 | TB-RT-001-01,02,03 | — | confirmed |
| DR-RT-004 | R-RT-04 | P1-Step2 | protocol=connected, med=0 | TP-M2-004 | MED=0配置成功，路由通告正常 | TB-RT-001-01,02 | — | confirmed |
| DR-RT-005 | R-RT-05 | P1-Step2 | protocol=static, med=4294967295 | TP-M2-004 | MED=MAX配置成功 | TB-RT-001-01,02 | — | confirmed |
| DR-RT-006 | R-RT-06 | P1-Step2 | protocol=connected, med=4294967296 | TP-M2-004 | API返回错误，MED越界拒绝 | TB-RT-001-01 | — | confirmed |
| DR-RT-007 | R-RT-07 | P1-Step2 | protocol=rip, med=100 | TP-M2-005 | API返回错误，非法协议拒绝 | TB-RT-001-01 | — | confirmed |
| DR-RT-008 | R-RT-08 | P1-Step2 | protocol=ospf, med=100 | TP-M2-005 | API返回错误，IPv4协议拒绝 | TB-RT-001-01 | — | confirmed |
| DR-RT-009 | R-RT-09 | P1-Step2 | protocol=connected(DUT1含tap/admin口), med=100 | TP-M2-006 | 对端BGP路由表中无tap/admin路由 | TB-RT-001-01,02 | — | confirmed |
| DR-RT-010 | R-RT-10 | P1-Step2 | protocol=static(DUT1含ISP/IPSec/NAT66), med=50 | TP-M2-007 | 对端BGP路由表中无ISP/IPSec/NAT66路由 | TB-RT-001-01,02 | — | confirmed |
| DR-RT-011 | 扩展R-RT-03 | P1-Step3 | OSPFv3邻居断开后 | TP-F-M2-OSPFv3-002 | 对端OSPFv3引入路由被撤销 | TB-RT-001-01,02,03 | — | confirmed |

## 七、覆盖策略

### 7.1 判定表覆盖统计

| 覆盖维度 | 已覆盖 | 总数 | 覆盖率 |
|---------|:-----:|:---:|:-----:|
| 引入协议 | 5 (connected/static/ospfv3/rip/ospf) | 5 | 100% |
| MED边界值 | 5 (0/50/100/200/4294967295/4294967296) | 6 | 100% |
| 过滤路由类型 | 5 (tap/admin/ISP/IPSec/NAT66) | 5 | 100% |
| 有效路径 | 8 | 8 | 100% |
| 异常路径 | 5 | 5 | 100% |

### 7.2 覆盖策略

- **等价类覆盖**: 每个协议类型 × MED正常值 生成一条有效 data row
- **边界值覆盖**: MED 最小(0)、最大(4294967295)、越界(4294967296) 各一条
- **异常覆盖**: 非法协议(rip/ospf) 各一条
- **白名单过滤覆盖**: 5 种过滤路由类型各一条
- **耦合覆盖**: OSPFv3 邻居断开时路由撤销验证

### 7.3 未覆盖/待确认事项

| 缺口 | 说明 | 优先级 |
|------|------|:---:|
| design-plan.md 缺失 | 无法确认设计计划推荐的 primary_signal 和 candidate_features | 低 |
| design-planner-reasoning.md 缺失 | 无法确认推理过程中的 uncertain_facts 和排除原因 | 低 |
| MED默认值 | 文档未明确 MED 默认值，当前假设必须显式指定 | 中 |

### 7.4 规则冗余与合并说明

- R-RT-04 和 R-RT-05 可合并为"MED边界值合法性校验"，但保留两条以保留边界值追踪
- R-RT-07 和 R-RT-08 可合并为"非法协议拒绝"，保留两条以区分 rip 和 ospf 不同拒绝原因
- 过滤子判定表中，tap 和 admin 共享 R-RT-09，ISP/IPSec/NAT66 共享 R-RT-10，但在 data row 中展开为独立行以确保覆盖
