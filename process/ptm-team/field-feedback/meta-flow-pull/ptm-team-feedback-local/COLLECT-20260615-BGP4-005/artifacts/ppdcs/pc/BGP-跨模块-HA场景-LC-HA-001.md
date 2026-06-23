# BGP4+ HA VRRP 模式主备切换 — 物理用例

> 生成时间: 2026-06-12 | 设计Skill: state-design
> 逻辑用例: LC-HA-001 | 目录: BGP-跨模块-HA场景

---

## PC-HA-001-01：HA VRRP初始主备状态验证

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-HA-001-01 |
| `logic_case_id` | LC-HA-001 |
| `requirement_ids` | SR-007, SR-010 |
| `feature_tags` | BGP4+, HA, VRRP, 主备 |
| `case_title` | 验证VRRP主备就绪时主设备BGP邻居Established, 备设备BGP邻居Idle |
| `priority` | P0 |
| `preconditions` | DUT1/DUT2组成VRRP双机 (虚拟IPv6已配置), HA同步列表包含BGP4+ |
| `graph_ref` | SP-01 |
| `coverage_goal` | 主生命周期: 初始状态验证 |
| `trigger_data` | VRRP配置=有VRRP, DUT1=Master, DUT2=Backup |
| `topology_binding_refs` | node4_dut2_tg1_sw1_link6 |
| `trace_refs` | TP-M1-024, S20-OP1, S20-OP2 |
| `scenario_refs` | S20 |
| `scenario_chain_refs` | S20 normal_path 步骤1→2 |
| `action_source_refs` | S20-OP1, S20-OP2 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认VRRP双机HA就绪, DUT1为Master
    target: DUT1, DUT2
    atomic_op:
      op_id: get-ha-status
      args: {}
    expected_result: DUT1=Master(持有虚拟IPv6), DUT2=Backup
    trace_refs:
      - S20-OP1

  - step_id: STEP-002
    step_name: 主设备DUT1配置BGP4+邻居(通过虚拟IP)
    target: DUT1
    atomic_op:
      op_id: post-bgp6neighbor
      args:
        ip: "<TG_IPv6>"
        asNumber: 100
        interface: "GE0_1"
    expected_result: BGP邻居通过虚拟IP建立Established
    trace_refs:
      - S20-OP2

  - step_id: STEP-003
    step_name: 验证备设备DUT2 BGP配置已同步
    target: DUT2
    atomic_op:
      op_id: get-bgp6neighbor
      args: {}
    expected_result: 备设备BGP邻居配置与主设备一致, 但邻居状态=Idle (虚拟IP不在本机)
    trace_refs:
      - S20-OP2

  - step_id: STEP-004
    step_name: 验证主设备BGP邻居为Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args: {}
    expected_result: 邻居状态=Established, 通过虚拟IPv6通信
    trace_refs:
      - TP-M1-024
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认VRRP双机HA就绪 | DUT1, DUT2 | get-ha-status | DUT1=Master, DUT2=Backup |
| 2 | 主设备配置BGP4+邻居 | DUT1 | post-bgp6neighbor ip=<TG_IPv6>, asNumber=100 | BGP Established |
| 3 | 验证备设备配置同步 | DUT2 | get-bgp6neighbor | 配置一致, 状态=Idle |
| 4 | 验证主设备BGP Established | DUT1 | get-bgp6neighbor | 状态=Established |

### expected_results

1. VRRP双机HA就绪: DUT1=Master(持虚拟IP), DUT2=Backup
2. 主设备BGP邻居通过虚拟IP建立Established
3. 备设备BGP配置已同步但邻居为Idle (虚拟IP不在本机)

---

## PC-HA-001-02：主动主备切换 — 主降备

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-HA-001-02 |
| `logic_case_id` | LC-HA-001 |
| `requirement_ids` | SR-007, SR-010 |
| `feature_tags` | BGP4+, HA, VRRP, 主动切换 |
| `case_title` | 验证主动降低主设备VRRP优先级后虚拟IP浮动, 备设备BGP邻居重建Established |
| `priority` | P0 |
| `preconditions` | DUT1=Master(BGP Established), DUT2=Backup(BGP Idle), 虚拟IPv6在DUT1 |
| `graph_ref` | SP-02 |
| `coverage_goal` | 主动切换: T1 (S0→S1), 主降备路径 |
| `trigger_data` | 降低DUT1 VRRP优先级 (低于DUT2) |
| `topology_binding_refs` | node4_dut2_tg1_sw1_link6 |
| `trace_refs` | S20-OP3, LC-HA-001 P1 |
| `scenario_refs` | S20 |
| `scenario_chain_refs` | S20 normal_path 步骤3 |
| `action_source_refs` | S20-OP3 |
| `confirmation_gap_refs` | GAP-HA-002: 切换时间≤5s |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认初始状态: DUT1=Master(BGP Established), DUT2=Backup(BGP Idle)
    target: DUT1, DUT2
    atomic_op:
      op_id: get-bgp6neighbor
      args: {}
    expected_result: DUT1邻居=Established, DUT2邻居=Idle
    trace_refs:
      - TP-M1-024

  - step_id: STEP-002
    step_name: 主动降低DUT1 VRRP优先级
    target: DUT1
    atomic_op:
      op_id: set-vrrp-priority
      args:
        priority: "<低于DUT2的值>"
    expected_result: VRRP优先级已降低
    trace_refs:
      - S20-OP3

  - step_id: STEP-003
    step_name: 观察DUT1 BGP邻居状态变化
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args: {}
    expected_result: 虚拟IP从DUT1移除后BGP邻居断开→Idle
    trace_refs:
      - S20-OP3

  - step_id: STEP-004
    step_name: 观察DUT2 BGP邻居状态变化
    target: DUT2
    atomic_op:
      op_id: get-bgp6neighbor
      args: {}
    expected_result: 虚拟IP浮动到DUT2后BGP邻居重建→Established, 切换时间≤5s [待确认]
    trace_refs:
      - S20-OP3
    confirmation_gap_refs:
      - GAP-HA-002

  - step_id: STEP-005
    step_name: 验证DUT2 BGP邻居可正常交换路由
    target: DUT2
    atomic_op:
      op_id: get-bgp6route
      args: {}
    expected_result: DUT2通过BGP学到对端路由
    trace_refs:
      - S20-OP3
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认初始状态 | DUT1, DUT2 | get-bgp6neighbor | DUT1=Established, DUT2=Idle |
| 2 | 降低DUT1 VRRP优先级 | DUT1 | set-vrrp-priority | 优先级已降低 |
| 3 | 观察DUT1 BGP断开 | DUT1 | get-bgp6neighbor | 状态=Idle |
| 4 | 观察DUT2 BGP重建 | DUT2 | get-bgp6neighbor | 状态=Established, ≤5s [待确认] |
| 5 | 验证DUT2路由交换正常 | DUT2 | get-bgp6route | 学到对端路由 |

### expected_results

1. DUT1降优先级后虚拟IP浮动到DUT2
2. DUT1 BGP邻居断开 (Idle)
3. DUT2 BGP邻居通过虚拟IP重建 Established
4. 切换时间 ≤ 5s [待确认]
5. 路由恢复正常通告

---

## PC-HA-001-03：被动主备切换 — 主故障

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-HA-001-03 |
| `logic_case_id` | LC-HA-001 |
| `requirement_ids` | SR-007, SR-010 |
| `feature_tags` | BGP4+, HA, VRRP, 被动切换 |
| `case_title` | 验证主设备断电后备设备自动升主BGP邻居重建 |
| `priority` | P1 |
| `preconditions` | DUT1=Master(BGP Established), DUT2=Backup(BGP Idle) |
| `graph_ref` | SP-03 |
| `coverage_goal` | 被动切换: T2 (S1→S0) |
| `trigger_data` | DUT1断电 (模拟主设备故障) |
| `topology_binding_refs` | node4_dut2_tg1_sw1_link6 |
| `trace_refs` | S20-OP4, LC-HA-001 P2 |
| `scenario_refs` | S20 |
| `scenario_chain_refs` | S20 normal_path 步骤4 |
| `action_source_refs` | S20-OP4 |
| `confirmation_gap_refs` | GAP-HA-002: 切换时间≤5s |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认初始状态
    target: DUT1, DUT2
    atomic_op:
      op_id: get-bgp6neighbor
      args: {}
    expected_result: DUT1=Established, DUT2=Idle
    trace_refs:
      - TP-M1-024

  - step_id: STEP-002
    step_name: 模拟主设备DUT1断电
    target: DUT1
    atomic_op:
      op_id: power-off
      args:
        device: "DUT1"
    expected_result: DUT1断电, VRRP心跳超时
    trace_refs:
      - S20-OP4

  - step_id: STEP-003
    step_name: 观察DUT2自动升为Master并重建BGP邻居
    target: DUT2
    atomic_op:
      op_id: get-bgp6neighbor
      args: {}
    expected_result: DUT2 VRRP→Master, 虚拟IPv6绑定, BGP邻居重建→Established, 切换时间≤5s [待确认]
    trace_refs:
      - S20-OP4
    confirmation_gap_refs:
      - GAP-HA-002

  - step_id: STEP-004
    step_name: 验证DUT2路由表恢复正常
    target: DUT2
    atomic_op:
      op_id: get-bgp6route
      args: {}
    expected_result: BGP路由已学习, 流量恢复转发
    trace_refs:
      - S20-OP4
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认初始状态 | DUT1, DUT2 | get-bgp6neighbor | DUT1=Established, DUT2=Idle |
| 2 | 模拟DUT1断电 | DUT1 | power-off device=DUT1 | DUT1断电 |
| 3 | 观察DUT2升主重建BGP | DUT2 | get-bgp6neighbor | 状态=Established, ≤5s [待确认] |
| 4 | 验证路由恢复 | DUT2 | get-bgp6route | 路由已学习 |

### expected_results

1. DUT1断电后VRRP心跳超时
2. DUT2自动升为Master, 绑定虚拟IPv6
3. DUT2 BGP邻居通过虚拟IP重建 Established
4. 路由恢复正常, 流量切换完成

---

## PC-HA-001-05：切换时间边界 ≤5s [待确认]

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-HA-001-05 |
| `logic_case_id` | LC-HA-001 |
| `requirement_ids` | SR-010 |
| `feature_tags` | BGP4+, HA, VRRP, 性能边界 |
| `case_title` | 验证主动主备切换后BGP邻居重建时间 ≤5s [待确认] |
| `priority` | P1 |
| `preconditions` | DUT1=Master(BGP Established), DUT2=Backup(BGP Idle) |
| `graph_ref` | SP-07 |
| `coverage_goal` | 边界守卫: 切换时间 ≤5s |
| `trigger_data` | 精确计时: 从降低DUT1优先级到DUT2 BGP Established |
| `topology_binding_refs` | node4_dut2_tg1_sw1_link6 |
| `trace_refs` | LC-HA-001 P1 |
| `scenario_refs` | S20 |
| `scenario_chain_refs` | S20 normal_path 步骤3 |
| `action_source_refs` | — |
| `confirmation_gap_refs` | GAP-HA-002 |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 记录起始时间戳并主动降低DUT1 VRRP优先级
    target: DUT1
    atomic_op:
      op_id: set-vrrp-priority-with-timestamp
      args:
        priority: "<低于DUT2的值>"
    expected_result: 记录切换开始时间T0
    trace_refs:
      - LC-HA-001 P1

  - step_id: STEP-002
    step_name: 轮询DUT2 BGP邻居状态直到Established并记录时间
    target: DUT2
    atomic_op:
      op_id: get-bgp6neighbor-until-established
      args: {}
    expected_result: 记录DUT2 BGP Established时间T1, 计算 ΔT = T1 - T0
    trace_refs:
      - LC-HA-001 P1
    confirmation_gap_refs:
      - GAP-HA-002

  - step_id: STEP-003
    step_name: 验证切换时间 ≤5s
    target: DUT1, DUT2
    atomic_op:
      op_id: assert-timing
      args:
        delta: ΔT
        max: 5s
    expected_result: "[待确认] ΔT ≤ 5s"
    trace_refs:
      - LC-HA-001 P1
    confirmation_gap_refs:
      - GAP-HA-002
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 记录T0并触发切换 | DUT1 | set-vrrp-priority-with-timestamp | T0=切换开始 |
| 2 | 轮询直到Established | DUT2 | get-bgp6neighbor-until-established | 记录T1 |
| 3 | 验证ΔT ≤5s | DUT1, DUT2 | assert-timing delta=ΔT, max=5s | [待确认] ΔT ≤ 5s |

### expected_results

[待确认] 切换时间 = VRRP切换时间 + BGP邻居重建时间:
- VRRP快速切换: 通常 <1s
- BGP邻居重建: Connect → OpenSent → OpenConfirm → Established
- 总时间目标 ≤5s
