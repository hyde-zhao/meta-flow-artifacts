# BGP4+ HA 非VRRP 开销控制 — 状态图分析

> 生成时间: 2026-06-12 | 设计Skill: state-design
> 逻辑用例: LC-HA-002 | PPDCS 特征: S-State

---

## Design Context

| 字段 | 值 |
|------|-----|
| `recommended_feature` | S-State |
| `recommended_method` | 状态图法 (Chow's 0/1-switch) |
| `design_skill` | state-design |
| `primary_signal` | HA 角色切换导致路由开销值在 normal↔max 间迁移 |
| `candidate_features` | S-State, D-Data |
| `exclusion_reasons` | P-Process 不适用：核心是开销值状态转换而非操作流程；P-Parameter 不适用：四种路由类型只是数据叠加层，不改变状态拓扑 |
| `fact_status` | needs-confirmation (开销转换的原子性和切换时长待确认) |
| `test_object_refs` | bgp6neighbor + bgp6redist + bgp6network (非VRRP HA 下 BGP4+ 路由通告) |
| `factor_refs` | FAC-BGP6-HA-VRRP-MODE, FAC-BGP6-HA-ROLE, FAC-BGP6-HA-ROUTE-COST, FAC-BGP6-HA-COST-TRANS, FAC-BGP6-HA-ROUTE-TYPE |
| `scenario_refs` | S20 (BGP4+ HA 主备切换) — 非VRRP 子场景 |
| `scenario_chain_refs` | LC-HA-002 动作路径 P1→P4 |
| `confirmation_gap_refs` | GAP-HA2-001: 开销值转换是否原子完成；GAP-HA2-002: 切换后对端选路收敛时间 |

---

## State Model

### Mermaid 状态图

```mermaid
stateDiagram-v2
    [*] --> 主设备_正常开销

    state 主设备_正常开销 {
        [*] --> Master_Role
        Master_Role --> Cost_Normal: 四种路由类型开销=正常值
        note right of Cost_Normal
            MED = normal
            直连开销 = normal
            静态开销 = normal
            OSPFv3开销 = normal
            对端选路优先选主设备
        end note
    }

    state 备设备_最大开销 {
        [*] --> Backup_Role
        Backup_Role --> Cost_Max: 四种路由类型开销=4294967295
        note right of Cost_Max
            MED = 4294967295
            直连开销 = 4294967295
            静态开销 = 4294967295
            OSPFv3开销 = 4294967295
            对端选路避免选备设备
        end note
    }

    主设备_正常开销 --> 备设备_最大开销: T1 [主降备] HA角色: Master→Backup<br>开销值: normal→max (4种路由类型)
    备设备_最大开销 --> 主设备_正常开销: T2 [备升主] HA角色: Backup→Master<br>开销值: max→normal (4种路由类型)

    note right of T1
        主降备:
        ─ 网络宣告路由开销 → max
        ─ 直连引入路由开销 → max
        ─ 静态引入路由开销 → max
        ─ OSPFv3引入路由开销 → max
    end note

    note right of T2
        备升主:
        ─ 网络宣告路由开销 → normal
        ─ 直连引入路由开销 → normal
        ─ 静态引入路由开销 → normal
        ─ OSPFv3引入路由开销 → normal
        对端选路自动切换
    end note

    %% 异常
    主设备_正常开销 --> 主设备_正常开销: [非法] 开销转换不完整<br>部分路由类型未更新
    备设备_最大开销 --> 备设备_最大开销: [非法] 开销转换不完整
```

### 状态清单

| state_id | state_name | entry_conditions | exit_observations | trace_refs | confirmation_gap_refs | fact_status |
|----------|------------|------------------|-------------------|------------|-----------------------|-------------|
| S0 | 主设备_正常开销 | HA 角色=Master, 四种路由类型开销均为正常值 (normal) | 对端通过选路算法优先选中主设备路由；四种路由类型的 MED/开销均为正常业务值 | TP-F-M3-HA-002, LC-HA-002 P1 | — | confirmed |
| S1 | 备设备_最大开销 | HA 角色=Backup, 四种路由类型开销均为最大值 (4294967295) | 对端选路算法避免选中备设备路由（开销最大）；四种路由类型开销均为 4294967295 | TP-F-M3-HA-002, LC-HA-002 P1 | — | confirmed |

---

## Transition Table

| transition_id | from | to | event | guard | effect | trace_refs | confirmation_gap_refs | fact_status |
|---------------|------|----|-------|-------|--------|------------|-----------------------|-------------|
| T1 | S0 (主设备_正常开销) | S1 (备设备_最大开销) | 主降备: HA 角色 Master→Backup | HA 角色变更触发 | 四种路由类型开销全部变为最大值 (4294967295): 网络宣告/直连引入/静态引入/OSPFv3引入 | LC-HA-002 P2 | GAP-HA2-001: 转换是否原子 | needs-confirmation |
| T2 | S1 (备设备_最大开销) | S0 (主设备_正常开销) | 备升主: HA 角色 Backup→Master | HA 角色变更触发 | 四种路由类型开销全部恢复为正常值: 网络宣告/直连引入/静态引入/OSPFv3引入 | LC-HA-002 P3 | GAP-HA2-001: 转换是否原子 | needs-confirmation |
| T3 (路由类型×4) | S0→S0 (验证) | — | 初始状态验证 | HA角色=Master | 对端路由表验证四种路由类型的开销均为正常值 | LC-HA-002 P1 | — | confirmed |
| T4 (路由类型×4) | S1→S1 (验证) | — | 初始状态验证 | HA角色=Backup | 对端路由表验证四种路由类型的开销均为最大值 | LC-HA-002 P1 | — | confirmed |
| T5 (非法) | S0→S0 | 部分路由类型开销未更新 | 主降备时部分路由类型开销更新失败 | 四种路由类型开销不一致 (部分 normal 部分 max) | LC-HA-002 P4 | GAP-HA2-001 | needs-confirmation |
| T6 (非法) | S1→S1 | 部分路由类型开销未更新 | 备升主时部分路由类型开销更新失败 | 四种路由类型开销不一致 | LC-HA-002 P4 | GAP-HA2-001 | needs-confirmation |

---

## State Path Selection

### 迁移路径

| state_path_id | transition_sequence | path_type | guard_summary | scenario_chain_refs | trace_refs | confirmation_gap_refs | fact_status |
|---------------|--------------------|-----------|---------------|---------------------|------------|-----------------------|-------------|
| SP-01 | S0 (初始: 主正常开销, 备最大开销) | 主生命周期 | 双设备各自独立BGP邻居, 主设备开销=normal, 备设备开销=max | LC-HA-002 P1 | TP-F-M3-HA-002 | — | confirmed |
| SP-02 | T1: S0→S1 (主降备) | 主降备路径 | Master→Backup 触发开销 normal→max | LC-HA-002 P2 | — | GAP-HA2-001 | needs-confirmation |
| SP-03 | T2: S1→S0 (备升主) | 备升主路径 | Backup→Master 触发开销 max→normal | LC-HA-002 P3 | — | GAP-HA2-001 | needs-confirmation |
| SP-04 | T1→T2: S0→S1→S0 (主降备+备升主) | 全切换路径 | 先主降备(开销→max) → 再原备升主(开销→normal), 对端选路切换 | LC-HA-002 P1→P2→P3 | — | GAP-HA2-002 | needs-confirmation |
| SP-05 | T3 (网络宣告路由) | 路由类型覆盖 | Master 网络宣告路由开销=normal | LC-HA-002 P4 网络宣告 | — | — | confirmed |
| SP-06 | T3 (直连引入路由) | 路由类型覆盖 | Master 直连引入路由开销=normal | LC-HA-002 P4 直连引入 | — | — | confirmed |
| SP-07 | T3 (静态引入路由) | 路由类型覆盖 | Master 静态引入路由开销=normal | LC-HA-002 P4 静态引入 | — | — | confirmed |
| SP-08 | T3 (OSPFv3引入路由) | 路由类型覆盖 | Master OSPFv3引入路由开销=normal | LC-HA-002 P4 OSPFv3引入 | — | — | confirmed |
| SP-09 | T1→验证四种路由类型均为max | 路由类型覆盖 | Backup 下四种路由类型开销=4294967295 | LC-HA-002 P4 | — | — | confirmed |
| SP-10 | T2→验证四种路由类型均为normal | 路由类型覆盖 | Master 下四种路由类型开销=normal | LC-HA-002 P4 | — | — | confirmed |
| SP-11 | T5 (非法: 开销转换不完整) | 非法迁移 | 主降备时部分路由类型开销未更新至max | — | — | GAP-HA2-001 | needs-confirmation |

---

## Guard Conditions & Data Overlay

### 守卫条件数据映射

| state_path_id | transition_id | factor_ref | td_ref | value_set | guard_expectation | data_overlay_set | status |
|---------------|---------------|------------|--------|-----------|-------------------|------------------|--------|
| SP-01 | — | HA角色 | TD-HA2-001 | DUT1=Master, DUT2=Backup | pass | OVL-HA2-01 | ready |
| SP-01 | — | VRRP配置 | TD-HA2-002 | 无VRRP | pass | OVL-HA2-01 | ready |
| SP-02 | T1 | 开销值转换 | TD-HA2-003 | Master→Backup, cost: normal→max | pass (四种路由类型均变max) | OVL-HA2-02 | needs-confirmation |
| SP-03 | T2 | 开销值转换 | TD-HA2-004 | Backup→Master, cost: max→normal | pass (四种路由类型均变normal) | OVL-HA2-03 | needs-confirmation |
| SP-05 | T3 | 路由类型=网络宣告 | TD-HA2-RT01 | 网络宣告路由 | pass (开销=normal) | OVL-HA2-RT01 | ready |
| SP-06 | T3 | 路由类型=直连引入 | TD-HA2-RT02 | 直连引入路由 | pass (开销=normal) | OVL-HA2-RT02 | ready |
| SP-07 | T3 | 路由类型=静态引入 | TD-HA2-RT03 | 静态引入路由 | pass (开销=normal) | OVL-HA2-RT03 | ready |
| SP-08 | T3 | 路由类型=OSPFv3引入 | TD-HA2-RT04 | OSPFv3引入路由 | pass (开销=normal) | OVL-HA2-RT04 | ready |
| SP-09 | T1 | 路由类型=网络宣告 | TD-HA2-RT05 | 主降备后网络宣告开销=max | pass (4294967295) | OVL-HA2-02 | needs-confirmation |
| SP-09 | T1 | 路由类型=直连引入 | TD-HA2-RT06 | 主降备后直连引入开销=max | pass (4294967295) | OVL-HA2-02 | needs-confirmation |
| SP-09 | T1 | 路由类型=静态引入 | TD-HA2-RT07 | 主降备后静态引入开销=max | pass (4294967295) | OVL-HA2-02 | needs-confirmation |
| SP-09 | T1 | 路由类型=OSPFv3引入 | TD-HA2-RT08 | 主降备后OSPFv3引入开销=max | pass (4294967295) | OVL-HA2-02 | needs-confirmation |
| SP-10 | T2 | 路由类型×4 | TD-HA2-RT09~12 | 备升主后四种路由类型开销=normal | pass | OVL-HA2-03 | needs-confirmation |

### 数据叠加层定义

| overlay_id | 描述 | 关键参数 | 预期结果 |
|------------|------|---------|---------|
| OVL-HA2-01 | 初始状态 | 主=Master(开销normal), 备=Backup(开销max) | 对端选路优先选主 |
| OVL-HA2-02 | 主降备 | HA角色 Master→Backup | 四种路由开销 normal→4294967295 |
| OVL-HA2-03 | 备升主 | HA角色 Backup→Master | 四种路由开销 4294967295→normal |
| OVL-HA2-RT01~04 | 主设备四种路由类型初始验证 | 网络宣告/直连引入/静态引入/OSPFv3引入 | 开销均为normal |
| OVL-HA2-RT05~08 | 备设备四种路由类型验证 | 网络宣告/直连引入/静态引入/OSPFv3引入 | 开销均为4294967295 |

---

## PC Derivation Summary

| physical_case_id | logic_case_id | state_path_id | data_overlay_set | coverage_goal |
|-----------------|---------------|---------------|------------------|---------------|
| PC-HA-002-01 | LC-HA-002 | SP-01 | OVL-HA2-01 | 主生命周期: 初始双机开销验证 |
| PC-HA-002-02 | LC-HA-002 | SP-02 | OVL-HA2-02 | 主降备: 开销 normal→max |
| PC-HA-002-03 | LC-HA-002 | SP-03 | OVL-HA2-03 | 备升主: 开销 max→normal |
| PC-HA-002-04 | LC-HA-002 | SP-04 | OVL-HA2-02→OVL-HA2-03 | 全切换: 降备→升主 对端选路切换 |
| PC-HA-002-05 | LC-HA-002 | SP-05 | OVL-HA2-RT01 | 路由类型覆盖: 网络宣告 |
| PC-HA-002-06 | LC-HA-002 | SP-06 | OVL-HA2-RT02 | 路由类型覆盖: 直连引入 |
| PC-HA-002-07 | LC-HA-002 | SP-07 | OVL-HA2-RT03 | 路由类型覆盖: 静态引入 |
| PC-HA-002-08 | LC-HA-002 | SP-08 | OVL-HA2-RT04 | 路由类型覆盖: OSPFv3引入 |
| PC-HA-002-09 | LC-HA-002 | SP-09 | OVL-HA2-RT05~08 | 降备后四种路由类型均为max |
| PC-HA-002-10 | LC-HA-002 | SP-10 | OVL-HA2-RT09~12 | 升主后四种路由类型均为normal |
| PC-HA-002-11 | LC-HA-002 | SP-11 | — | 非法迁移: 开销转换不完整 [待确认] |

---

## Uncertain Facts / Confirmation Gaps

| gap_id | 描述 | 影响范围 | fact_status |
|--------|------|---------|-------------|
| GAP-HA2-001 | 开销值转换的原子性：四种路由类型从 normal→max (或反向) 是否同时完成？如果部分失败（如 OSPFv3 更新失败但直连已更新）系统如何处理？ | T1, T2, T5, T6 迁移守卫 | needs-confirmation |
| GAP-HA2-002 | 备升主后，对端选路从原主切换到新主的收敛时间是多少？是否需要等待 BGP UPDATE 消息传播？ | SP-04 全切换路径 | needs-confirmation |
| GAP-HA2-003 | 主备两台设备同时处于 Master 角色时的脑裂处理：路由开销值如何设定？是否会两端都发 normal 开销导致选路混乱？ | — | needs-confirmation |
