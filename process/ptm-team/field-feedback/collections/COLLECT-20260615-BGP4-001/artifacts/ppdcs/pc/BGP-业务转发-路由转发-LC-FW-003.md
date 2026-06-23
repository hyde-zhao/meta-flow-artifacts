# BGP4+ 不同出接口类型路由转发 — 物理用例

> 生成时间: 2026-06-12 | 设计Skill: parameter-design
> 逻辑用例: LC-FW-003 | 目录: BGP-业务转发-路由转发

---

## PC-FW3-PAR-001：物理口+IBGP

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW3-PAR-001 |
| `logic_case_id` | LC-FW-003 |
| `requirement_ids` | SR-FW-003 |
| `feature_tags` | BGP4+, 出接口, 物理口, IBGP |
| `case_title` | 验证物理口作为出接口时IBGP路由转发正常 |
| `priority` | P1 |
| `preconditions` | DUT1/DUT2通过物理口GE0_1连接, IBGP Established |
| `graph_ref` | SP-01 |
| `coverage_goal` | 出接口基线: 物理口 + IBGP |
| `trigger_data` | 出接口类型=物理口, 路由学习=IBGP |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-FW3-001 |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 normal_path |
| `action_source_refs` | DR-FW3-001 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认物理口GE0_1状态up
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        interface: "GE0_1"
    expected_result: 接口物理状态=up, IPv6地址已配置
    trace_refs:
      - DR-FW3-001

  - step_id: STEP-002
    step_name: 确认IBGP邻居通过物理口Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, interface=GE0_1
    trace_refs:
      - DR-FW3-001

  - step_id: STEP-003
    step_name: DUT1学习IBGP路由，出接口为GE0_1
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:f31::/64"
    expected_result: 路由下一跳出接口=GE0_1
    trace_refs:
      - DR-FW3-001

  - step_id: STEP-004
    step_name: TG1验证通过物理口的业务转发
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:f31::1"
        duration: 30
    expected_result: 流量转发正常, 无丢包
    trace_refs:
      - DR-FW3-001
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认物理口状态up | DUT1 | fw_config_interface | GE0_1 up |
| 2 | 确认IBGP Established | DUT1 | get-bgp6neighbor | 状态=Established, interface=GE0_1 |
| 3 | 验证路由出接口 | DUT1 | fw_verify_bgp6_route | 出接口=GE0_1 |
| 4 | TG1流量验证 | TG1 | tg_start_traffic_stream | 转发正常, 无丢包 |

### expected_results

1. 物理口作为 BGP 出接口路由转发正常
2. IBGP 路由正确指向物理口
3. 业务流量无丢包

---

## PC-FW3-PAR-002：物理口+EBGP

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW3-PAR-002 |
| `logic_case_id` | LC-FW-003 |
| `requirement_ids` | SR-FW-003 |
| `feature_tags` | BGP4+, 出接口, 物理口, EBGP |
| `case_title` | 验证物理口作为出接口时EBGP路由转发正常 |
| `priority` | P1 |
| `preconditions` | DUT1/DUT3通过物理口GE0_1连接, EBGP Established |
| `graph_ref` | SP-01 |
| `coverage_goal` | 出接口基线: 物理口 + EBGP |
| `trigger_data` | 出接口类型=物理口, 路由学习=EBGP |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-FW3-002 |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 normal_path |
| `action_source_refs` | DR-FW3-002 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认物理口GE0_1状态up
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        interface: "GE0_1"
    expected_result: 接口物理状态=up, IPv6地址已配置
    trace_refs:
      - DR-FW3-002

  - step_id: STEP-002
    step_name: 确认EBGP邻居Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:3::2"
    expected_result: 邻居状态=Established, AS不同, interface=GE0_1
    trace_refs:
      - DR-FW3-002

  - step_id: STEP-003
    step_name: DUT1学习EBGP路由，出接口为GE0_1
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:f32::/64"
    expected_result: 路由下一跳出接口=GE0_1, 来源=EBGP
    trace_refs:
      - DR-FW3-002

  - step_id: STEP-004
    step_name: TG1验证通过物理口的EBGP业务转发
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:f32::1"
        duration: 30
    expected_result: 流量转发正常, 无丢包
    trace_refs:
      - DR-FW3-002
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认物理口状态up | DUT1 | fw_config_interface | GE0_1 up |
| 2 | 确认EBGP Established | DUT1 | get-bgp6neighbor | EBGP Established |
| 3 | 验证路由出接口 | DUT1 | fw_verify_bgp6_route | 出接口=GE0_1, 来源=EBGP |
| 4 | TG1流量验证 | TG1 | tg_start_traffic_stream | 转发正常 |

### expected_results

1. 物理口作为 EBGP 出接口路由转发正常
2. EBGP 路由正确指向物理口
3. 业务流量无丢包

---

## PC-FW3-PAR-003：子接口+IBGP

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW3-PAR-003 |
| `logic_case_id` | LC-FW-003 |
| `requirement_ids` | SR-FW-003 |
| `feature_tags` | BGP4+, 出接口, 子接口, IBGP |
| `case_title` | 验证子接口作为出接口时IBGP路由转发正常 |
| `priority` | P1 |
| `preconditions` | DUT1/DUT2通过子接口GE0_1.100连接, IBGP Established |
| `graph_ref` | SP-01 |
| `coverage_goal` | 出接口类型: 子接口 + IBGP |
| `trigger_data` | 出接口类型=子接口, 路由学习=IBGP |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-FW3-003 |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 subif_path |
| `action_source_refs` | DR-FW3-003 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 配置子接口GE0_1.100, VLAN 100
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        interface: "GE0_1.100"
        type: sub_interface
        vlan: 100
        ipv6: "2001:db8:1:100::1/64"
    expected_result: 子接口创建成功, 状态up
    trace_refs:
      - DR-FW3-003

  - step_id: STEP-002
    step_name: 通过子接口创建IBGP邻居
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1:100::2"
        asNumber: 100
        interface: "GE0_1.100"
    expected_result: IBGP邻居Established, interface=GE0_1.100
    trace_refs:
      - DR-FW3-003

  - step_id: STEP-003
    step_name: DUT1学习IBGP路由，出接口为子接口
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:f33::/64"
    expected_result: 路由下一跳出接口=GE0_1.100
    trace_refs:
      - DR-FW3-003

  - step_id: STEP-004
    step_name: TG1验证通过子接口的业务转发
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:f33::1"
        duration: 30
    expected_result: 流量转发正常, VLAN Tag正确
    trace_refs:
      - DR-FW3-003
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 配置子接口GE0_1.100 | DUT1 | fw_config_interface type=sub_interface, vlan=100 | 子接口up |
| 2 | 通过子接口创建IBGP邻居 | DUT1 | fw_create_bgp6_neighbor interface=GE0_1.100 | IBGP Established |
| 3 | 验证路由出接口 | DUT1 | fw_verify_bgp6_route | 出接口=GE0_1.100 |
| 4 | TG1流量验证 | TG1 | tg_start_traffic_stream | 转发正常, VLAN正确 |

### expected_results

1. 子接口作为 BGP 出接口路由转发正常
2. 流量携带正确的 VLAN Tag
3. 子接口隔离正常

---

## PC-FW3-PAR-004：子接口+EBGP

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW3-PAR-004 |
| `logic_case_id` | LC-FW-003 |
| `requirement_ids` | SR-FW-003 |
| `feature_tags` | BGP4+, 出接口, 子接口, EBGP |
| `case_title` | 验证子接口作为出接口时EBGP路由转发正常 |
| `priority` | P1 |
| `preconditions` | DUT1/DUT3通过子接口GE0_1.200连接, EBGP Established |
| `graph_ref` | SP-01 |
| `coverage_goal` | 出接口类型: 子接口 + EBGP |
| `trigger_data` | 出接口类型=子接口, 路由学习=EBGP |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-FW3-004 |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 subif_path |
| `action_source_refs` | DR-FW3-004 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 配置子接口GE0_1.200, VLAN 200
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        interface: "GE0_1.200"
        type: sub_interface
        vlan: 200
        ipv6: "2001:db8:3:200::1/64"
    expected_result: 子接口创建成功, 状态up
    trace_refs:
      - DR-FW3-004

  - step_id: STEP-002
    step_name: 通过子接口创建EBGP邻居
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:3:200::2"
        asNumber: 200
        interface: "GE0_1.200"
    expected_result: EBGP邻居Established, interface=GE0_1.200
    trace_refs:
      - DR-FW3-004

  - step_id: STEP-003
    step_name: DUT1学习EBGP路由，出接口为子接口
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:f34::/64"
    expected_result: 路由下一跳出接口=GE0_1.200, 来源=EBGP
    trace_refs:
      - DR-FW3-004

  - step_id: STEP-004
    step_name: TG1验证通过子接口的EBGP业务转发
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:f34::1"
        duration: 30
    expected_result: 流量转发正常
    trace_refs:
      - DR-FW3-004
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 配置子接口GE0_1.200 | DUT1 | fw_config_interface type=sub_interface, vlan=200 | 子接口up |
| 2 | 通过子接口创建EBGP邻居 | DUT1 | fw_create_bgp6_neighbor interface=GE0_1.200 | EBGP Established |
| 3 | 验证路由出接口 | DUT1 | fw_verify_bgp6_route | 出接口=GE0_1.200 |
| 4 | TG1流量验证 | TG1 | tg_start_traffic_stream | 转发正常 |

### expected_results

1. 子接口作为 EBGP 出接口路由转发正常
2. EBGP 邻居通过子接口建立和通信正常
3. 业务流量无丢包

---

## PC-FW3-PAR-005：聚合口+IBGP+成员切换

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW3-PAR-005 |
| `logic_case_id` | LC-FW-003 |
| `requirement_ids` | SR-FW-003 |
| `feature_tags` | BGP4+, 出接口, 聚合口, IBGP, 成员切换 |
| `case_title` | 验证聚合口作为出接口时IBGP路由转发及成员口切换不影响转发 |
| `priority` | P1 |
| `preconditions` | DUT1/DUT2通过聚合口Eth-Trunk1(GE0_1+GE0_2)连接, IBGP Established |
| `graph_ref` | SP-01 |
| `coverage_goal` | 出接口类型: 聚合口 + IBGP + 成员切换 |
| `trigger_data` | 出接口类型=聚合口, 路由学习=IBGP, 成员切换 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-FW3-005 |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 lag_path |
| `action_source_refs` | DR-FW3-005 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 配置聚合口Eth-Trunk1, 成员GE0_1+GE0_2
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        interface: "Eth-Trunk1"
        type: lag
        members: ["GE0_1", "GE0_2"]
        ipv6: "2001:db8:1::1/64"
    expected_result: 聚合口创建成功, 2成员在线
    trace_refs:
      - DR-FW3-005

  - step_id: STEP-002
    step_name: 通过聚合口创建IBGP邻居
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "Eth-Trunk1"
    expected_result: IBGP Established, interface=Eth-Trunk1
    trace_refs:
      - DR-FW3-005

  - step_id: STEP-003
    step_name: TG1发起持续流量
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:f35::1"
        duration: 120
    expected_result: 流量持续发送
    trace_refs:
      - DR-FW3-005

  - step_id: STEP-004
    step_name: 移除成员GE0_1, 验证流量切换到GE0_2
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        interface: "Eth-Trunk1"
        members: ["GE0_2"]
    expected_result: 聚合口成员变为仅GE0_2, 流量不中断
    trace_refs:
      - DR-FW3-005

  - step_id: STEP-005
    step_name: 验证BGP邻居仍Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established
    trace_refs:
      - DR-FW3-005

  - step_id: STEP-006
    step_name: 恢复成员GE0_1
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        interface: "Eth-Trunk1"
        members: ["GE0_1", "GE0_2"]
    expected_result: 聚合口恢复2成员, 流量正常
    trace_refs:
      - DR-FW3-005
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 配置聚合口Eth-Trunk1 | DUT1 | fw_config_interface type=lag, members=[GE0_1, GE0_2] | 聚合口up |
| 2 | 通过聚合口创建IBGP邻居 | DUT1 | fw_create_bgp6_neighbor interface=Eth-Trunk1 | IBGP Established |
| 3 | TG1发起持续流量 | TG1 | tg_start_traffic_stream | 流量持续 |
| 4 | 移除成员GE0_1 | DUT1 | fw_config_interface members=[GE0_2] | 流量不中断 |
| 5 | 验证BGP邻居仍Established | DUT1 | get-bgp6neighbor | Established |
| 6 | 恢复成员GE0_1 | DUT1 | fw_config_interface members=[GE0_1, GE0_2] | 恢复正常 |

### expected_results

1. 聚合口作为 BGP 出接口路由转发正常
2. 聚合口成员切换时 BGP 邻居不中断
3. 流量在成员口间无感切换

---

## PC-FW3-PAR-006：聚合口+EBGP

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW3-PAR-006 |
| `logic_case_id` | LC-FW-003 |
| `requirement_ids` | SR-FW-003 |
| `feature_tags` | BGP4+, 出接口, 聚合口, EBGP |
| `case_title` | 验证聚合口作为出接口时EBGP路由转发正常 |
| `priority` | P1 |
| `preconditions` | DUT1/DUT3通过聚合口Eth-Trunk2(GE0_3+GE0_4)连接, EBGP Established |
| `graph_ref` | SP-01 |
| `coverage_goal` | 出接口类型: 聚合口 + EBGP |
| `trigger_data` | 出接口类型=聚合口, 路由学习=EBGP |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-FW3-006 |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 lag_path |
| `action_source_refs` | DR-FW3-006 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 配置聚合口Eth-Trunk2
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        interface: "Eth-Trunk2"
        type: lag
        members: ["GE0_3", "GE0_4"]
        ipv6: "2001:db8:3::1/64"
    expected_result: 聚合口创建成功
    trace_refs:
      - DR-FW3-006

  - step_id: STEP-002
    step_name: 通过聚合口创建EBGP邻居
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:3::2"
        asNumber: 200
        interface: "Eth-Trunk2"
    expected_result: EBGP Established
    trace_refs:
      - DR-FW3-006

  - step_id: STEP-003
    step_name: 验证路由出接口为聚合口
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:f36::/64"
    expected_result: 路由下一跳出接口=Eth-Trunk2, 来源=EBGP
    trace_refs:
      - DR-FW3-006

  - step_id: STEP-004
    step_name: TG1流量验证
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:f36::1"
        duration: 30
    expected_result: 流量转发正常
    trace_refs:
      - DR-FW3-006
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 配置聚合口Eth-Trunk2 | DUT1 | fw_config_interface type=lag | 聚合口up |
| 2 | 通过聚合口创建EBGP邻居 | DUT1 | fw_create_bgp6_neighbor interface=Eth-Trunk2 | EBGP Established |
| 3 | 验证路由出接口 | DUT1 | fw_verify_bgp6_route | 出接口=Eth-Trunk2 |
| 4 | TG1流量验证 | TG1 | tg_start_traffic_stream | 转发正常 |

### expected_results

1. 聚合口作为 EBGP 出接口路由转发正常
2. EBGP 邻居通过聚合口建立和通信正常
3. 业务流量无丢包

---

## PC-FW3-PAR-007：隧道口+IBGP

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW3-PAR-007 |
| `logic_case_id` | LC-FW-003 |
| `requirement_ids` | SR-FW-003 |
| `feature_tags` | BGP4+, 出接口, 隧道口, IBGP |
| `case_title` | 验证隧道口作为出接口时IBGP路由转发正常 |
| `priority` | P1 |
| `preconditions` | DUT1/DUT2通过GRE/IPsec隧道连接, IBGP通过隧道Established |
| `graph_ref` | SP-01 |
| `coverage_goal` | 出接口类型: 隧道口 + IBGP |
| `trigger_data` | 出接口类型=隧道口, 路由学习=IBGP |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-FW3-007 |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 tunnel_path |
| `action_source_refs` | DR-FW3-007 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 配置隧道接口Tunnel1
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        interface: "Tunnel1"
        type: tunnel
        tunnel_mode: gre
        source: "2001:db8:1::1"
        destination: "2001:db8:1::2"
        ipv6: "2001:db8:tun1::1/64"
    expected_result: 隧道口创建成功, 状态up
    trace_refs:
      - DR-FW3-007

  - step_id: STEP-002
    step_name: 通过隧道口创建IBGP邻居
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:tun1::2"
        asNumber: 100
        interface: "Tunnel1"
    expected_result: IBGP Established, interface=Tunnel1
    trace_refs:
      - DR-FW3-007

  - step_id: STEP-003
    step_name: 验证路由出接口为隧道口
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:f37::/64"
    expected_result: 路由下一跳出接口=Tunnel1
    trace_refs:
      - DR-FW3-007

  - step_id: STEP-004
    step_name: TG1验证通过隧道口的业务转发
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:f37::1"
        duration: 30
    expected_result: 流量经隧道封装后正常转发
    trace_refs:
      - DR-FW3-007
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 配置隧道口Tunnel1 | DUT1 | fw_config_interface type=tunnel, mode=gre | 隧道up |
| 2 | 通过隧道口创建IBGP邻居 | DUT1 | fw_create_bgp6_neighbor interface=Tunnel1 | IBGP Established |
| 3 | 验证路由出接口 | DUT1 | fw_verify_bgp6_route | 出接口=Tunnel1 |
| 4 | TG1流量验证 | TG1 | tg_start_traffic_stream | 经隧道转发正常 |

### expected_results

1. 隧道口作为 BGP 出接口路由转发正常
2. BGP 报文经隧道封装传输
3. 业务流量经过隧道正常转发

---

## PC-FW3-PAR-008：隧道口+EBGP

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW3-PAR-008 |
| `logic_case_id` | LC-FW-003 |
| `requirement_ids` | SR-FW-003 |
| `feature_tags` | BGP4+, 出接口, 隧道口, EBGP |
| `case_title` | 验证隧道口作为出接口时EBGP路由转发正常 |
| `priority` | P1 |
| `preconditions` | DUT1/DUT3通过GRE隧道连接, EBGP通过隧道Established |
| `graph_ref` | SP-01 |
| `coverage_goal` | 出接口类型: 隧道口 + EBGP |
| `trigger_data` | 出接口类型=隧道口, 路由学习=EBGP |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-FW3-008 |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 tunnel_path |
| `action_source_refs` | DR-FW3-008 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 配置隧道接口Tunnel2
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        interface: "Tunnel2"
        type: tunnel
        tunnel_mode: gre
        source: "2001:db8:1::1"
        destination: "2001:db8:3::2"
        ipv6: "2001:db8:tun2::1/64"
    expected_result: 隧道口创建成功, 状态up
    trace_refs:
      - DR-FW3-008

  - step_id: STEP-002
    step_name: 通过隧道口创建EBGP邻居
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:tun2::2"
        asNumber: 200
        interface: "Tunnel2"
    expected_result: EBGP Established
    trace_refs:
      - DR-FW3-008

  - step_id: STEP-003
    step_name: 验证路由出接口为隧道口
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:f38::/64"
    expected_result: 路由下一跳出接口=Tunnel2, 来源=EBGP
    trace_refs:
      - DR-FW3-008

  - step_id: STEP-004
    step_name: TG1流量验证
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:f38::1"
        duration: 30
    expected_result: 流量经隧道封装后正常转发
    trace_refs:
      - DR-FW3-008
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 配置隧道口Tunnel2 | DUT1 | fw_config_interface type=tunnel | 隧道up |
| 2 | 通过隧道口创建EBGP邻居 | DUT1 | fw_create_bgp6_neighbor interface=Tunnel2 | EBGP Established |
| 3 | 验证路由出接口 | DUT1 | fw_verify_bgp6_route | 出接口=Tunnel2 |
| 4 | TG1流量验证 | TG1 | tg_start_traffic_stream | 经隧道转发正常 |

### expected_results

1. 隧道口作为 EBGP 出接口路由转发正常
2. EBGP 邻居通过隧道建立和通信正常
3. 业务流量经过隧道正常转发

---

## PC-FW3-PAR-009：MTU=1280 边界

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW3-PAR-009 |
| `logic_case_id` | LC-FW-003 |
| `requirement_ids` | SR-FW-003 |
| `feature_tags` | BGP4+, MTU边界, 1280, IPv6最小MTU |
| `case_title` | 验证MTU=1280（IPv6最小MTU）时BGP4+路由转发正常 |
| `priority` | P2 |
| `preconditions` | DUT1/DUT2 IBGP Established, 接口MTU=1280 |
| `graph_ref` | SP-01 |
| `coverage_goal` | MTU边界: IPv6最小MTU 1280 |
| `trigger_data` | MTU=1280 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-FW3-009 |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 boundary_path |
| `action_source_refs` | DR-FW3-009 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 配置接口MTU=1280
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        interface: "GE0_1"
        mtu: 1280
    expected_result: MTU=1280配置成功
    trace_refs:
      - DR-FW3-009

  - step_id: STEP-002
    step_name: 确认IBGP邻居Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established
    trace_refs:
      - DR-FW3-009

  - step_id: STEP-003
    step_name: TG1发起1278字节载荷流量
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:f39::1"
        payload_size: 1278
        duration: 30
    expected_result: 流量转发正常, 不分片
    trace_refs:
      - DR-FW3-009

  - step_id: STEP-004
    step_name: TG1发起1280字节满载流量
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:f39::1"
        payload_size: 1280
        duration: 30
    expected_result: 流量转发正常, MTU满载
    trace_refs:
      - DR-FW3-009
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 配置MTU=1280 | DUT1 | fw_config_interface mtu=1280 | 配置成功 |
| 2 | 确认IBGP Established | DUT1 | get-bgp6neighbor | Established |
| 3 | 1278字节流量验证 | TG1 | tg_start_traffic_stream payload=1278 | 正常转发 |
| 4 | 1280字节满载验证 | TG1 | tg_start_traffic_stream payload=1280 | 正常转发 |

### expected_results

1. MTU=1280（IPv6 最小 MTU）时 BGP4+ 邻居正常建立
2. 1280 字节满载报文正常转发不分片
3. 无 PMTU 问题

---

## PC-FW3-PAR-010：MTU=1279 异常

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW3-PAR-010 |
| `logic_case_id` | LC-FW-003 |
| `requirement_ids` | SR-FW-003 |
| `feature_tags` | BGP4+, MTU越界, 1279, 异常处理 |
| `case_title` | 验证MTU=1279（低于IPv6最小MTU）时BGP4+行为异常 |
| `priority` | P2 |
| `preconditions` | DUT1/DUT2 IBGP邻居, 接口MTU尝试配置为1279 |
| `graph_ref` | SP-01 |
| `coverage_goal` | MTU越界: 低于IPv6最小MTU 1279 |
| `trigger_data` | MTU=1279 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-FW3-010 |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 abnormal_path |
| `action_source_refs` | DR-FW3-010 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 尝试配置接口MTU=1279
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        interface: "GE0_1"
        mtu: 1279
    expected_result: API返回error或自动调整为1280
    trace_refs:
      - DR-FW3-010

  - step_id: STEP-002
    step_name: 确认最终MTU值
    target: DUT1
    atomic_op:
      op_id: get-interface
      args:
        interface: "GE0_1"
    expected_result: 检查MTU未被设为1279 (拒绝或调整)
    trace_refs:
      - DR-FW3-010

  - step_id: STEP-003
    step_name: 确认IBGP邻居行为
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态不受影响 (若设备自动调整为1280)
    trace_refs:
      - DR-FW3-010
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 尝试配置MTU=1279 | DUT1 | fw_config_interface mtu=1279 | API error或自动调整 |
| 2 | 确认最终MTU值 | DUT1 | get-interface GE0_1 | MTU≠1279 |
| 3 | 确认IBGP邻居行为 | DUT1 | get-bgp6neighbor | 不受影响 |

### expected_results

1. MTU=1279 低于 IPv6 最小 MTU(1280) 被拒绝或自动调整
2. 设备不会接受低于 1280 的 MTU 值用于 IPv6 接口
3. 现有 BGP 邻居不受影响

---

## PC-FW3-PAR-011：聚合口故障恢复

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW3-PAR-011 |
| `logic_case_id` | LC-FW-003 |
| `requirement_ids` | SR-FW-003 |
| `feature_tags` | BGP4+, 聚合口, 故障恢复, 高可用 |
| `case_title` | 验证聚合口全部成员故障后恢复时BGP邻居重建和路由恢复 |
| `priority` | P2 |
| `preconditions` | DUT1/DUT2通过聚合口Eth-Trunk1 IBGP Established |
| `graph_ref` | SP-01 |
| `coverage_goal` | 故障恢复: 聚合口全部成员故障后恢复 |
| `trigger_data` | 聚合口全部成员down后恢复 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-FW3-011 |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 lag_failure_recovery_path |
| `action_source_refs` | DR-FW3-011 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认聚合口IBGP Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居Established, interface=Eth-Trunk1
    trace_refs:
      - DR-FW3-011

  - step_id: STEP-002
    step_name: 同时down聚合口所有成员
    target: DUT1
    atomic_op:
      op_id: set-interface-down
      args:
        interface: "GE0_1"
    expected_result: GE0_1 down
    trace_refs:
      - DR-FW3-011

  - step_id: STEP-003
    step_name: down最后一个成员GE0_2
    target: DUT1
    atomic_op:
      op_id: set-interface-down
      args:
        interface: "GE0_2"
    expected_result: GE0_2 down, 聚合口down, BGP邻居断开
    trace_refs:
      - DR-FW3-011

  - step_id: STEP-004
    step_name: 恢复聚合口成员GE0_1
    target: DUT1
    atomic_op:
      op_id: set-interface-up
      args:
        interface: "GE0_1"
    expected_result: GE0_1 up, 聚合口up
    trace_refs:
      - DR-FW3-011

  - step_id: STEP-005
    step_name: 验证BGP邻居重建Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居自动重建Established
    trace_refs:
      - DR-FW3-011

  - step_id: STEP-006
    step_name: TG1验证业务转发恢复
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:f311::1"
        duration: 30
    expected_result: 流量转发恢复正常
    trace_refs:
      - DR-FW3-011
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认聚合口IBGP Established | DUT1 | get-bgp6neighbor | Established, interface=Eth-Trunk1 |
| 2 | down成员GE0_1 | DUT1 | set-interface-down GE0_1 | GE0_1 down |
| 3 | down成员GE0_2 | DUT1 | set-interface-down GE0_2 | 聚合口down, BGP断开 |
| 4 | 恢复成员GE0_1 | DUT1 | set-interface-up GE0_1 | 聚合口up |
| 5 | 验证BGP邻居重建 | DUT1 | get-bgp6neighbor | 自动重建Established |
| 6 | TG1验证业务恢复 | TG1 | tg_start_traffic_stream | 转发正常 |

### expected_results

1. 聚合口全部成员故障后 BGP 邻居断开
2. 成员恢复后聚合口 up，BGP 邻居自动重建
3. 业务转发恢复正常

---

## PC-FW3-PAR-012：隧道故障耦合

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW3-PAR-012 |
| `logic_case_id` | LC-FW-003 |
| `requirement_ids` | SR-FW-003 |
| `feature_tags` | BGP4+, 隧道口, 故障耦合, 底层链路 |
| `case_title` | 验证隧道底层物理链路故障时BGP邻居断开和恢复 |
| `priority` | P2 |
| `preconditions` | DUT1/DUT3通过隧道Tunnel1 EBGP Established, 隧道底层为物理口GE0_1 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 故障耦合: 隧道底层链路故障影响BGP |
| `trigger_data` | 隧道底层物理口down |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-FW3-012 |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 tunnel_failure_path |
| `action_source_refs` | DR-FW3-012 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认隧道EBGP Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:tun2::2"
    expected_result: 邻居Established, interface=Tunnel2
    trace_refs:
      - DR-FW3-012

  - step_id: STEP-002
    step_name: down隧道底层物理口GE0_1
    target: DUT1
    atomic_op:
      op_id: set-interface-down
      args:
        interface: "GE0_1"
    expected_result: GE0_1 down, 隧道down
    trace_refs:
      - DR-FW3-012

  - step_id: STEP-003
    step_name: 验证隧道口状态down
    target: DUT1
    atomic_op:
      op_id: get-interface
      args:
        interface: "Tunnel2"
    expected_result: 隧道状态=down
    trace_refs:
      - DR-FW3-012

  - step_id: STEP-004
    step_name: 验证BGP邻居断开
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:tun2::2"
    expected_result: 邻居状态≠Established
    trace_refs:
      - DR-FW3-012

  - step_id: STEP-005
    step_name: 恢复底层物理口
    target: DUT1
    atomic_op:
      op_id: set-interface-up
      args:
        interface: "GE0_1"
    expected_result: GE0_1 up, 隧道自动恢复
    trace_refs:
      - DR-FW3-012

  - step_id: STEP-006
    step_name: 验证BGP邻居自动重建
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:tun2::2"
    expected_result: 邻居自动重建Established
    trace_refs:
      - DR-FW3-012
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认隧道EBGP Established | DUT1 | get-bgp6neighbor | Established, interface=Tunnel2 |
| 2 | down底层物理口 | DUT1 | set-interface-down GE0_1 | GE0_1 down |
| 3 | 验证隧道down | DUT1 | get-interface Tunnel2 | 状态=down |
| 4 | 验证BGP邻居断开 | DUT1 | get-bgp6neighbor | ≠Established |
| 5 | 恢复底层物理口 | DUT1 | set-interface-up GE0_1 | GE0_1 up |
| 6 | 验证BGP自动重建 | DUT1 | get-bgp6neighbor | 自动重建Established |

### expected_results

1. 隧道底层物理链路故障导致隧道 down，BGP 邻居断开
2. 底层链路恢复后隧道自动 up
3. BGP 邻居自动重建 Established
