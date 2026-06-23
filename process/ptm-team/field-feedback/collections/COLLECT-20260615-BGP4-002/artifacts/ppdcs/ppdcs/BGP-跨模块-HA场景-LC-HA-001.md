# BGP4+ HA VRRP 模式主备切换 — 状态图分析

> 生成时间: 2026-06-12 | 设计Skill: state-design
> 逻辑用例: LC-HA-001 | PPDCS 特征: S-State

---

## Design Context

| 字段 | 值 |
|------|-----|
| `recommended_feature` | S-State |
| `recommended_method` | 状态图法 (Chow's 0/1-switch) |
| `design_skill` | state-design |
| `primary_signal` | VRRP 主备角色切换导致 BGP 邻居状态迁移 |
| `candidate_features` | S-State |
| `exclusion_reasons` | P-Process 不适用：主备切换事件驱动的 BGP 邻居状态变化是核心，非步骤流程编排；P-Parameter 不适用：切换方向(主动/被动)不改变状态拓扑 |
| `fact_status` | needs-confirmation (HA配置不同步细节、备设备修改拒绝机制待确认) |
| `test_object_refs` | bgp6neighbor (VRRP 环境下 BGP4+ 邻居) |
| `factor_refs` | FAC-BGP6-HA-VRRP-MODE, FAC-BGP6-HA-ROLE |
| `scenario_refs` | S20 (BGP4+ HA 主备切换) |
| `scenario_chain_refs` | S20 normal_path + abnormal_path |
| `confirmation_gap_refs` | GAP-HA-001: 备设备独立修改配置的拒绝机制；GAP-HA-002: 切换期间路由黑洞时长 |

---

## State Model

### Mermaid 状态图

```mermaid
stateDiagram-v2
    [*] --> 主设备_BGP_Established

    state 主设备_BGP_Established {
        [*] --> VRRP_Master
        VRRP_Master --> BGP_Up: 虚拟IP在本机
        note right of BGP_Up: BGP邻居 Established<br>通过虚拟IP通信
    }

    state 备设备_BGP_Idle {
        [*] --> VRRP_Backup
        VRRP_Backup --> BGP_Down: 虚拟IP不在本机
        note right of BGP_Down: BGP邻居 Idle<br>等待虚拟IP浮动
    }

    主设备_BGP_Established --> 备设备_BGP_Idle: T1 [主动切换] 降优先级<br>→ 虚拟IP浮动到备机<br>→ 主BGP down + 备BGP重建
    备设备_BGP_Idle --> 主设备_BGP_Established: T2 [被动切换] 主设备故障<br>→ 虚拟IP浮动到备机<br>→ 备BGP重建为Established

    note right of T1
        主动切换:
        1. 降低DUT1优先级
        2. 虚拟IP从DUT1移除
        3. DUT1 BGP邻居断开→Idle
        4. 虚拟IP浮动到DUT2
        5. DUT2 BGP邻居重建→Established
        切换时间 ≤ 5s
    end note

    note right of T2
        被动切换:
        1. DUT1断电/故障
        2. VRRP心跳超时
        3. DUT2升为Master
        4. 虚拟IP浮动到DUT2
        5. DUT2 BGP邻居重建
    end note

    %% 异常路径
    主设备_BGP_Established --> 主设备_BGP_Established: [非法] HA配置不同步<br>→ 备机配置滞后
    备设备_BGP_Idle --> 备设备_BGP_Idle: [非法] 备机独立修改配置<br>→ 操作被拒绝/告警
```

### 状态清单

| state_id | state_name | entry_conditions | exit_observations | trace_refs | confirmation_gap_refs | fact_status |
|----------|------------|------------------|-------------------|------------|-----------------------|-------------|
| S0 | 主设备_BGP_Established | VRRP Master 持有虚拟 IPv6 + BGP 邻居 Established | 虚拟 IP 在本机，BGP 邻居正常交换路由；对端通过虚拟 IP 通信 | TP-M1-024 | — | confirmed |
| S1 | 备设备_BGP_Idle | VRRP Backup 不持有虚拟 IPv6 + BGP 邻居 Idle | 虚拟 IP 不在本机，BGP 邻居处于 Idle；配置已同步但未激活 | S20-OP2 | — | confirmed |
| S2 | HA配置不同步 | 主设备修改 BGP4+ 配置但 HA 同步失败 | 备设备配置与主设备不一致 | S20 abnormal_path (HA同步列表中无BGP) | GAP-HA-001 | needs-confirmation |

---

## Transition Table

| transition_id | from | to | event | guard | effect | trace_refs | confirmation_gap_refs | fact_status |
|---------------|------|----|-------|-------|--------|------------|-----------------------|-------------|
| T1 | S0 (主设备) | S1 (备设备) | 主动主备切换 (降优先级) | VRRP优先级被手动降低, 虚拟IP浮动触发 | DUT1 VRRP→Backup, 虚拟IP移除, BGP邻居→Idle; DUT2 VRRP→Master, 虚拟IP绑定, BGP邻居重建→Established | S20-OP3, LC-HA-001 P1 | — | confirmed |
| T2 | S1 (备设备) | S0 (主设备) | 被动主备切换 (主设备故障) | DUT1 断电/故障, VRRP心跳超时 | DUT2 VRRP→Master, 虚拟IP浮动到DUT2, BGP邻居重建→Established; DUT1 BGP→Idle | S20-OP4, LC-HA-001 P2 | — | confirmed |
| T3 | S1 (备设备) | S0 (主设备) | 备升主 (恢复切换) | 原主设备恢复后, 或手动提升备设备优先级 | 同T2, 角色反转 | LC-HA-001 P2 (恢复) | — | confirmed |
| T4 (非法) | S0 (主设备) | S2 (HA配置不同步) | 主设备修改BGP配置→HA同步失败 | HA同步机制缺陷 (Bug#220943 已修复) | 备设备BGP配置不更新, 切换后行为异常 | S20 abnormal_path | GAP-HA-001 | needs-confirmation |
| T5 (非法) | S1 (备设备) | S1 (备设备) | 备设备独立修改BGP配置 | 备设备角色=Backup, 尝试直接修改BGP4+配置 | 操作被拒绝 或 产生配置冲突告警, HA同步列表显示不一致 | LC-HA-001 P4 | GAP-HA-001 | needs-confirmation |

---

## State Path Selection

### 迁移路径

| state_path_id | transition_sequence | path_type | guard_summary | scenario_chain_refs | trace_refs | confirmation_gap_refs | fact_status |
|---------------|--------------------|-----------|---------------|---------------------|------------|-----------------------|-------------|
| SP-01 | S0 (DUT1主+DUT2备, 初始) | 主生命周期 | DUT1=VRRP Master + BGP Established, DUT2=VRRP Backup + BGP Idle | S20 步骤1→2 | TP-M1-024 | — | confirmed |
| SP-02 | T1: S0→S1 (DUT1主动降备) | 主动切换 | 降DUT1优先级 → 虚拟IP浮动 → DUT1 BGP down → DUT2 BGP重建 | S20 步骤3, LC-HA-001 P1 | S20-OP3 | — | confirmed |
| SP-03 | T2: S1→S0 (DUT1被动故障) | 被动切换 | DUT1断电 → VRRP超时 → DUT2升主 → BGP重建 | S20 步骤4, LC-HA-001 P2 | S20-OP4 | — | confirmed |
| SP-04 | T1→T3: S0→S1→S0 (主动切换+回切) | 恢复路径 | DUT1降备→DUT2升主→DUT1恢复→DUT1升主 | LC-HA-001 P1+P2恢复 | — | — | confirmed |
| SP-05 | T4: S0→S2 (HA配置不同步) | 异常路径 | 主设备修改BGP配置 → HA同步失败 → 备设备配置不一致 | S20 abnormal_path, LC-HA-001 P3 | Bug#220943 | GAP-HA-001 | needs-confirmation |
| SP-06 | T5: S1→S1 (备设备修改被拒) | 非法迁移 | 备设备直接修改BGP配置 → 操作被拒绝或告警 | LC-HA-001 P4 | — | GAP-HA-001 | needs-confirmation |
| SP-07 | T1→切换时间≤5s | 边界守卫 | 主动切换后 BGP 邻居重建时间 ≤5s | LC-HA-001 P1 | — | GAP-HA-002 | needs-confirmation |
| SP-08 | T2→切换时间≤5s | 边界守卫 | 被动切换后 BGP 邻居重建时间 ≤5s | LC-HA-001 P2 | — | GAP-HA-002 | needs-confirmation |

---

## Guard Conditions & Data Overlay

### 守卫条件数据映射

| state_path_id | transition_id | factor_ref | td_ref | value_set | guard_expectation | data_overlay_set | status |
|---------------|---------------|------------|--------|-----------|-------------------|------------------|--------|
| SP-01 | — | VRRP配置 | TD-HA-001 | 有VRRP, 虚拟IPv6 | pass (双机HA就绪) | OVL-HA-01 | ready |
| SP-01 | — | HA角色 | TD-HA-002 | DUT1=Master, DUT2=Backup | pass (角色正确) | OVL-HA-01 | ready |
| SP-02 | T1 | HA切换方向=主降备 | TD-HA-003 | 降低DUT1优先级 | pass (DUT2升主, BGP重建) | OVL-HA-02 | ready |
| SP-03 | T2 | HA切换方向=备升主 | TD-HA-004 | DUT1断电 | pass (DUT2自动升主, BGP重建) | OVL-HA-03 | ready |
| SP-05 | T4 | HA配置同步 | TD-HA-005 | 主设备修改配置, 同步失败 | fail (备设备配置不一致) | OVL-HA-05 | needs-confirmation |
| SP-06 | T5 | 备设备修改 | TD-HA-006 | DUT2(Backup) PUT bgp6neighbor | fail (操作被拒绝) | OVL-HA-06 | needs-confirmation |
| SP-07 | T1 | 切换时间 | TD-HA-007 | ≤5s | pass (快速切换) | OVL-HA-02 | needs-confirmation |
| SP-08 | T2 | 切换时间 | TD-HA-008 | ≤5s | pass (快速切换) | OVL-HA-03 | needs-confirmation |

### 数据叠加层定义

| overlay_id | 描述 | 关键参数 | 预期结果 |
|------------|------|---------|---------|
| OVL-HA-01 | 初始状态验证 | VRRP配置+DUT1主+DUT2备 | DUT1 BGP Established, DUT2 BGP Idle |
| OVL-HA-02 | 主动切换(主降备) | 降DUT1优先级 | DUT1 BGP→Idle, DUT2 BGP→Established, ≤5s |
| OVL-HA-03 | 被动切换(主故障) | DUT1断电 | DUT2 BGP→Established, ≤5s |
| OVL-HA-04 | 恢复切换 | DUT1恢复/提升优先级 | DUT1 BGP→Established, DUT2 BGP→Idle |
| OVL-HA-05 | HA配置不同步 | 主修改配置, 同步失败 | 备配置不一致, 切换后异常 |
| OVL-HA-06 | 备设备修改被拒 | Backup角色PUT配置 | 操作被拒绝或告警 |

---

## PC Derivation Summary

| physical_case_id | logic_case_id | state_path_id | data_overlay_set | coverage_goal |
|-----------------|---------------|---------------|------------------|---------------|
| PC-HA-001-01 | LC-HA-001 | SP-01 | OVL-HA-01 | 主生命周期: 初始主备状态验证 |
| PC-HA-001-02 | LC-HA-001 | SP-02 | OVL-HA-02 | 主动切换: 主降备路径 |
| PC-HA-001-03 | LC-HA-001 | SP-03 | OVL-HA-03 | 被动切换: 主故障路径 |
| PC-HA-001-04 | LC-HA-001 | SP-04 | OVL-HA-04 | 恢复路径: 主动切换+回切 |
| PC-HA-001-05 | LC-HA-001 | SP-07 | OVL-HA-02 | 边界守卫: 主动切换时间≤5s |
| PC-HA-001-06 | LC-HA-001 | SP-08 | OVL-HA-03 | 边界守卫: 被动切换时间≤5s |
| PC-HA-001-07 | LC-HA-001 | SP-05 | OVL-HA-05 | 异常路径: HA配置不同步 [待确认] |
| PC-HA-001-08 | LC-HA-001 | SP-06 | OVL-HA-06 | 非法迁移: 备设备独立修改 [待确认] |

---

## Uncertain Facts / Confirmation Gaps

| gap_id | 描述 | 影响范围 | fact_status |
|--------|------|---------|-------------|
| GAP-HA-001 | 备设备独立修改 BGP4+ 配置的拒绝机制：是 API 层拒绝还是 HA 框架层拒绝？拒绝的错误码/告警形式是什么？ | T5 迁移路径, SP-06 | needs-confirmation |
| GAP-HA-002 | 主备切换期间路由黑洞持续时间：是否严格 ≤5s？VRRP 快速切换 + BGP Graceful Restart 是否支持？ | SP-07, SP-08 | needs-confirmation |
| GAP-HA-003 | HA 配置同步失败后，备设备 BGP 配置是保持旧版本还是清空？Bug#220943 已在 C02 修复，C03 需回归验证 | SP-05 | needs-confirmation |
