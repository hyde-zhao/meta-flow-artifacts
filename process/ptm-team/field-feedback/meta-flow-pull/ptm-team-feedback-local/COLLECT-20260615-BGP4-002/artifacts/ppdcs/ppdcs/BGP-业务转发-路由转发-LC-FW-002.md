# BGP4+ 路由冗余 — 静态路由与 BGP4+ 互备 — 状态图分析

> 生成时间: 2026-06-12 | 设计Skill: state-design
> 逻辑用例: LC-FW-002 | PPDCS 特征: S-State

---

## Design Context

| 字段 | 值 |
|------|-----|
| `recommended_feature` | S-State |
| `recommended_method` | 状态图法 (Chow's 0/1-switch) |
| `design_skill` | state-design |
| `primary_signal` | 静态路由在线/失效驱动转发路径在静态路由和 BGP4+ 间切换 |
| `candidate_features` | S-State |
| `exclusion_reasons` | P-Process 不适用：核心是转发状态切换而非操作流程步骤；P-Parameter 不适用：管理距离(AD)差异是状态的固有属性而非可变参数 |
| `fact_status` | needs-confirmation (中断时间≤5s 是否在所有场景下保证) |
| `test_object_refs` | bgp6redist + 静态路由表 (BGP4+ 路由冗余转发) |
| `factor_refs` | FAC-STATIC-AD, FAC-BGP-AD, FAC-ROUTE-STATE, FAC-FORWARD-SWITCH |
| `scenario_refs` | S10 (引入静态IPv6路由), S08 (邻居断联与恢复) |
| `scenario_chain_refs` | LC-FW-002 动作路径 P1→P3 |
| `confirmation_gap_refs` | GAP-FW2-001: 中断时间≤5s 保证机制；GAP-FW2-002: BGP 路由备份的预装机制 |

---

## State Model

### Mermaid 状态图

```mermaid
stateDiagram-v2
    [*] --> 静态路由在线

    state 静态路由在线 {
        [*] --> Static_Active
        Static_Active --> Static_AD1: AD=1 (最高优先级)
        note right of Static_AD1
            路由表: 静态路由 (AD=1)
            FIB: 下一跳指向静态路由
            流量: 走静态路由转发
            BGP路由在RIB中但AD低不生效
        end note
    }

    state BGP路由备份 {
        [*] --> BGP_Active
        BGP_Active --> BGP_AD200: IBGP AD=200 (或 EBGP AD=20)
        note right of BGP_AD200
            路由表: BGP路由 (AD=200)
            FIB: 下一跳指向BGP邻居
            流量: 走BGP路由转发
            静态路由已删除
        end note
    }

    静态路由在线 --> BGP路由备份: T1 [静态路由失效]<br>删除静态路由<br>→ 路由表切换到BGP路由<br>流量中断 ≤ 5s
    BGP路由备份 --> 静态路由在线: T2 [静态路由恢复]<br>重新添加静态路由<br>→ 路由表切回静态路由<br>AD=1 优先

    note right of T1
        切换序列:
        1. 删除静态路由
        2. RIB检测静态路由消失
        3. RIB安装次优BGP路由
        4. FIB更新下一跳
        5. 流量走BGP路由
        中断 ≤ 5s
    end note

    note right of T2
        恢复序列:
        1. 重新添加静态路由
        2. RIB检测静态路由(AD=1)
        3. RIB替换BGP路由为静态路由
        4. FIB更新下一跳
        5. 流量恢复走静态路由
    end note

    %% 边界场景
    静态路由在线 --> 静态路由在线: [边界] AD优先级验证<br>静态AD=1 > EBGP AD=20 > IBGP AD=200
    BGP路由备份 --> BGP路由备份: [边界] 静态路由持续失效<br>BGP邻居断开 → 流量中断
```

### 状态清单

| state_id | state_name | entry_conditions | exit_observations | trace_refs | confirmation_gap_refs | fact_status |
|----------|------------|------------------|-------------------|------------|-----------------------|-------------|
| S0 | 静态路由在线 | 静态路由已配置且生效，管理距离 AD=1；同目的 BGP 路由也存在于 RIB 中但 AD 更高不生效 | FIB 下一跳指向静态路由配置的下一跳；流量通过静态路由转发 | TP-F-M2-STATIC-PRI-001 | — | confirmed |
| S1 | BGP路由备份 | 静态路由被删除或失效；RIB 中仅剩 BGP 路由 (IBGP AD=200 或 EBGP AD=20) | FIB 下一跳指向 BGP 邻居；流量通过 BGP 路由转发 | TP-F-M2-STATIC-PRI-001, TP-Q-REL-014 | GAP-FW2-002: BGP路由是否已预装在FIB | needs-confirmation |
| S2 | 无路由 (流量中断) | 静态路由失效 + BGP 邻居断开 (所有路由源失效) | FIB 中无该目的路由；流量黑洞 | — | — | confirmed |

---

## Transition Table

| transition_id | from | to | event | guard | effect | trace_refs | confirmation_gap_refs | fact_status |
|---------------|------|----|-------|-------|--------|------------|-----------------------|-------------|
| T1 | S0 (静态路由在线) | S1 (BGP路由备份) | 删除静态路由 | 静态路由从配置中移除 | RIB 检测静态路由消失 → 安装 BGP 路由 (次优AD) → FIB 更新 → 流量切换到 BGP 转发 | LC-FW-002 P1 | GAP-FW2-001 | needs-confirmation |
| T2 | S1 (BGP路由备份) | S0 (静态路由在线) | 重新添加静态路由 | 静态路由恢复且下一跳可达 | RIB 检测静态路由(AD=1)优先于 BGP(AD=200) → 替换 FIB 条目 → 流量切回静态 | LC-FW-002 P2 | GAP-FW2-001 | needs-confirmation |
| T3 | S1 (BGP路由备份) | S2 (无路由) | BGP 邻居断开 | 静态路由仍失效 + BGP 邻居 Established→Idle | RIB 中目的路由全部撤销 → FIB 无路由 → 流量中断 | LC-FW-002 P3 (扩展) | — | confirmed |
| T4 (AD优先级) | S0 (验证) | S0 (验证) | 查看路由表 | 同目的存在静态(AD=1) + IBGP(AD=200) 两条路由 | 路由表优选静态路由 (AD最小优先) | LC-FW-002 P3 | — | confirmed |
| T5 (AD优先级) | S0 (验证) | S0 (验证) | 查看路由表 | 同目的存在静态(AD=1) + EBGP(AD=20) 两条路由 | 路由表优选静态路由 (静态AD=1 < EBGP AD=20) | LC-FW-002 P3 | — | confirmed |
| T6 (非法) | S0 (静态路由在线) | S2 (无路由) | 同时删除静态路由 + BGP邻居断开 | 所有路由源同时失效 | RIB 无任何路由 → FIB 无条目 → 流量黑洞 | — | — | confirmed |

---

## State Path Selection

### 迁移路径

| state_path_id | transition_sequence | path_type | guard_summary | scenario_chain_refs | trace_refs | confirmation_gap_refs | fact_status |
|---------------|--------------------|-----------|---------------|---------------------|------------|-----------------------|-------------|
| SP-01 | S0 (初始: 静态在线) | 主生命周期 | 静态路由AD=1 优先, BGP路由在RIB中备份 | LC-FW-002 P1 前段 | TP-F-M2-STATIC-PRI-001 | — | confirmed |
| SP-02 | T1: S0→S1 (静态失效→BGP回退) | 回退切换 | 删除静态路由 → 切BGP转发, 中断≤5s | LC-FW-002 P1 | TP-Q-REL-014 | GAP-FW2-001 | needs-confirmation |
| SP-03 | T2: S1→S0 (静态恢复→切回静态) | 恢复切换 | 重新添加静态路由 → 切回静态转发 | LC-FW-002 P2 | — | GAP-FW2-001 | needs-confirmation |
| SP-04 | T1→T2: S0→S1→S0 (完整切换循环) | 全生命周期 | 静态失效→BGP回退→静态恢复→切回 | LC-FW-002 P1+P2 | — | GAP-FW2-001 | needs-confirmation |
| SP-05 | T3: S1→S2 (BGP邻居断开) | 异常退化 | 静态失效期间BGP邻居也断开 → 无路由 | LC-FW-002 P3 (扩展) | — | — | confirmed |
| SP-06 | T4: AD优先级 (IBGP vs 静态) | 边界守卫 | 静态 AD=1 < IBGP AD=200 → 静态优先 | LC-FW-002 P3 | — | — | confirmed |
| SP-07 | T5: AD优先级 (EBGP vs 静态) | 边界守卫 | 静态 AD=1 < EBGP AD=20 → 静态优先 | LC-FW-002 P3 | — | — | confirmed |

---

## Guard Conditions & Data Overlay

### 守卫条件数据映射

| state_path_id | transition_id | factor_ref | td_ref | value_set | guard_expectation | data_overlay_set | status |
|---------------|---------------|------------|--------|-----------|-------------------|------------------|--------|
| SP-01 | — | 静态路由管理距离 | TD-FW2-001 | AD=1 | pass (路由表优选静态) | OVL-FW2-01 | ready |
| SP-01 | — | BGP路由管理距离 | TD-FW2-002 | IBGP AD=200 | pass (次优, 备份) | OVL-FW2-01 | ready |
| SP-02 | T1 | 路由状态 | TD-FW2-003 | 删除静态路由 | pass (切BGP, 中断≤5s) | OVL-FW2-02 | needs-confirmation |
| SP-03 | T2 | 路由状态 | TD-FW2-004 | 重新添加静态路由 | pass (切回静态) | OVL-FW2-03 | needs-confirmation |
| SP-06 | T4 | AD比较 | TD-FW2-005 | 静态AD=1 vs IBGP AD=200 | pass (静态优先) | OVL-FW2-04 | ready |
| SP-07 | T5 | AD比较 | TD-FW2-006 | 静态AD=1 vs EBGP AD=20 | pass (静态仍然优先) | OVL-FW2-05 | ready |
| SP-05 | T3 | BGP邻居状态 | TD-FW2-007 | 静态失效 + BGP邻居Idle | pass (无路由, 流量中断) | OVL-FW2-06 | ready |

### 数据叠加层定义

| overlay_id | 描述 | 关键参数 | 预期结果 |
|------------|------|---------|---------|
| OVL-FW2-01 | 初始状态 | 静态AD=1 + IBGP AD=200 | 路由表优选静态, BGP备份 |
| OVL-FW2-02 | 静态失效→BGP回退 | 删除静态路由 (IBGP场景) | 切BGP转发, 中断≤5s |
| OVL-FW2-03 | 静态恢复→切回 | 重新添加静态路由 | 切回静态转发 |
| OVL-FW2-04 | AD优先级: IBGP比较 | 静态AD=1 vs IBGP AD=200 | 静态优先 |
| OVL-FW2-05 | AD优先级: EBGP比较 | 静态AD=1 vs EBGP AD=20 | 静态仍然优先 |
| OVL-FW2-06 | 无路由异常 | 静态失效 + BGP断开 | 流量中断 |

---

## PC Derivation Summary

| physical_case_id | logic_case_id | state_path_id | data_overlay_set | coverage_goal |
|-----------------|---------------|---------------|------------------|---------------|
| PC-FW-002-01 | LC-FW-002 | SP-01 | OVL-FW2-01 | 主生命周期: 静态在线AD优先验证 |
| PC-FW-002-02 | LC-FW-002 | SP-02 | OVL-FW2-02 | 回退切换: 静态失效→BGP转发 |
| PC-FW-002-03 | LC-FW-002 | SP-03 | OVL-FW2-03 | 恢复切换: 静态恢复→切回静态 |
| PC-FW-002-04 | LC-FW-002 | SP-04 | OVL-FW2-02→OVL-FW2-03 | 全生命周期: 失效→回退→恢复 |
| PC-FW-002-05 | LC-FW-002 | SP-06 | OVL-FW2-04 | 边界守卫: AD优先级 IBGP比较 |
| PC-FW-002-06 | LC-FW-002 | SP-07 | OVL-FW2-05 | 边界守卫: AD优先级 EBGP比较 |
| PC-FW-002-07 | LC-FW-002 | SP-05 | OVL-FW2-06 | 异常退化: 静态失效+BGP断开 |

---

## Uncertain Facts / Confirmation Gaps

| gap_id | 描述 | 影响范围 | fact_status |
|--------|------|---------|-------------|
| GAP-FW2-001 | 中断时间 ≤5s 的保证机制：是否通过 BGP 路由预安装到 FIB (BGP PIC) 实现？如果没有预安装，RIB→FIB 安装延迟是否会导致中断超过 5s？ | T1, T2 迁移效果 | needs-confirmation |
| GAP-FW2-002 | BGP 路由在静态路由在线时是否已预装在 FIB（作为备份下一跳）？还是仅存在于 RIB 中，需要静态失效后才安装到 FIB？ | S1 进入条件 | needs-confirmation |
| GAP-FW2-003 | 当同时存在 EBGP(AD=20) 和 IBGP(AD=200) 路由作为静态路由的备份时，静态失效后会优先选择 EBGP 还是 IBGP？ | SP-02 守卫 | needs-confirmation |
