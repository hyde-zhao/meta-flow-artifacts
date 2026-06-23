# BGP4+ 基于不同接口类型建立邻居 — 物理用例

> 生成时间: 2026-06-12 | 设计Skill: data-design
> 逻辑用例: LC-BPG-005 | 目录: BGP-配置管理-BPG配置

---

## PC-BPG-DAT-038：物理口邻居建立

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-038 |
| `logic_case_id` | LC-BPG-005 |
| `requirement_ids` | SR-007, SR-007c |
| `feature_tags` | BGP4+, 邻居管理, 接口类型, 物理口 |
| `case_title` | 验证BGP4+邻居通过物理口建立成功 |
| `priority` | P1 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能 (ASN=100, RouterID=1.1.1.1); 物理口 GE0_1 IPv6 配置互通; MTU=1500 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 主生命周期: 物理口邻居建立 Established |
| `trigger_data` | type=物理口, ip=2001:db8:1::2, iface=GE0_1, hop=1, mtu=1500 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-019 |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 normal_path "物理口" 步骤1→2 |
| `action_source_refs` | S04-OP1 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: POST创建邻居 (物理口 GE0_1, MTU=1500)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "GE0_1"
        ebgpMultihop: 1
    expected_result: API返回success, 邻居创建成功
    trace_refs:
      - TP-M1-019

  - step_id: STEP-002
    step_name: GET验证邻居状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, 接口=GE0_1
    trace_refs:
      - TP-M1-019

  - step_id: STEP-003
    step_name: 清理: DELETE删除邻居
    target: DUT1
    atomic_op:
      op_id: fw_delete_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: API返回success
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | POST创建邻居 (物理口 GE0_1) | DUT1 | fw_create_bgp6_neighbor ip=2001:db8:1::2, asNumber=100, interface=GE0_1, ebgpMultihop=1 | API返回success |
| 2 | GET验证邻居状态 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 邻居 Established |
| 3 | 清理: DELETE删除邻居 | DUT1 | fw_delete_bgp6_neighbor id=2001:db8:1::2 | API返回success |

### expected_results

1. 物理口 (GE0_1) 邻居创建成功
2. 邻居状态达到 Established
3. 邻居可正常交换 BGP UPDATE 消息

---

## PC-BPG-DAT-039：子接口邻居建立

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-039 |
| `logic_case_id` | LC-BPG-005 |
| `requirement_ids` | SR-007, SR-007c |
| `feature_tags` | BGP4+, 邻居管理, 接口类型, 子接口 |
| `case_title` | 验证BGP4+邻居通过子接口建立成功 |
| `priority` | P1 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能 (ASN=100, RouterID=1.1.1.1); 子接口 GE0_1.100 已创建且 IPv6 配置互通; MTU=1500 |
| `graph_ref` | SP-02 |
| `coverage_goal` | 接口类型覆盖: 子接口邻居建立 Established |
| `trigger_data` | type=子接口, ip=2001:db8:1::2, iface=GE0_1.100, hop=1, mtu=1500 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-019 |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 normal_path "子接口" 步骤1→2 |
| `action_source_refs` | S04-OP2 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: POST创建邻居 (子接口 GE0_1.100, MTU=1500)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "GE0_1.100"
        ebgpMultihop: 1
    expected_result: API返回success, 子接口邻居创建成功
    trace_refs:
      - TP-M1-019

  - step_id: STEP-002
    step_name: GET验证邻居状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, 接口=GE0_1.100
    trace_refs:
      - TP-M1-019

  - step_id: STEP-003
    step_name: 清理: DELETE删除邻居
    target: DUT1
    atomic_op:
      op_id: fw_delete_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: API返回success
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | POST创建邻居 (子接口 GE0_1.100) | DUT1 | fw_create_bgp6_neighbor ip=2001:db8:1::2, asNumber=100, interface=GE0_1.100, ebgpMultihop=1 | API返回success |
| 2 | GET验证邻居状态 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 邻居 Established |
| 3 | 清理: DELETE删除邻居 | DUT1 | fw_delete_bgp6_neighbor id=2001:db8:1::2 | API返回success |

### expected_results

1. 子接口 (GE0_1.100) 邻居创建成功
2. 邻居状态达到 Established，VLAN 隔离生效
3. 邻居可正常交换 BGP UPDATE 消息

---

## PC-BPG-DAT-040：聚合口邻居建立

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-040 |
| `logic_case_id` | LC-BPG-005 |
| `requirement_ids` | SR-007, SR-007c |
| `feature_tags` | BGP4+, 邻居管理, 接口类型, 聚合口 |
| `case_title` | 验证BGP4+邻居通过聚合口建立成功 |
| `priority` | P1 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能 (ASN=100, RouterID=1.1.1.1); 聚合口 bond0 已创建且 IPv6 配置互通; 至少一个成员口 UP; MTU=1500 |
| `graph_ref` | SP-03 |
| `coverage_goal` | 接口类型覆盖: 聚合口邻居建立 Established |
| `trigger_data` | type=聚合口, ip=2001:db8:1::2, iface=bond0, hop=1, mtu=1500 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-020 |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 normal_path "聚合口" 步骤1→2 |
| `action_source_refs` | S04-OP3 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: POST创建邻居 (聚合口 bond0, MTU=1500)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "bond0"
        ebgpMultihop: 1
    expected_result: API返回success, 聚合口邻居创建成功
    trace_refs:
      - TP-M1-020

  - step_id: STEP-002
    step_name: GET验证邻居状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, 接口=bond0
    trace_refs:
      - TP-M1-020

  - step_id: STEP-003
    step_name: 清理: DELETE删除邻居
    target: DUT1
    atomic_op:
      op_id: fw_delete_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: API返回success
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | POST创建邻居 (聚合口 bond0) | DUT1 | fw_create_bgp6_neighbor ip=2001:db8:1::2, asNumber=100, interface=bond0, ebgpMultihop=1 | API返回success |
| 2 | GET验证邻居状态 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 邻居 Established |
| 3 | 清理: DELETE删除邻居 | DUT1 | fw_delete_bgp6_neighbor id=2001:db8:1::2 | API返回success |

### expected_results

1. 聚合口 (bond0) 邻居创建成功
2. 邻居状态达到 Established
3. 聚合口成员链路冗余正常

---

## PC-BPG-DAT-041：隧道口邻居建立

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-041 |
| `logic_case_id` | LC-BPG-005 |
| `requirement_ids` | SR-007, SR-007c |
| `feature_tags` | BGP4+, 邻居管理, 接口类型, 隧道口 |
| `case_title` | 验证BGP4+邻居通过隧道口建立成功 |
| `priority` | P1 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能 (ASN=100, RouterID=1.1.1.1); 隧道口 tunnel0 已创建且 IPv6 配置互通; 隧道源/目的可达; MTU=1500 |
| `graph_ref` | SP-04 |
| `coverage_goal` | 接口类型覆盖: 隧道口邻居建立 Established |
| `trigger_data` | type=隧道口, ip=2001:db8:1::2, iface=tunnel0, hop=1, mtu=1500 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-021 |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 normal_path "隧道口" 步骤1→2 |
| `action_source_refs` | S04-OP4 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: POST创建邻居 (隧道口 tunnel0, MTU=1500)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "tunnel0"
        ebgpMultihop: 1
    expected_result: API返回success, 隧道口邻居创建成功
    trace_refs:
      - TP-M1-021

  - step_id: STEP-002
    step_name: GET验证邻居状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, 接口=tunnel0
    trace_refs:
      - TP-M1-021

  - step_id: STEP-003
    step_name: 清理: DELETE删除邻居
    target: DUT1
    atomic_op:
      op_id: fw_delete_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: API返回success
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | POST创建邻居 (隧道口 tunnel0) | DUT1 | fw_create_bgp6_neighbor ip=2001:db8:1::2, asNumber=100, interface=tunnel0, ebgpMultihop=1 | API返回success |
| 2 | GET验证邻居状态 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 邻居 Established |
| 3 | 清理: DELETE删除邻居 | DUT1 | fw_delete_bgp6_neighbor id=2001:db8:1::2 | API返回success |

### expected_results

1. 隧道口 (tunnel0) 邻居创建成功
2. 邻居状态达到 Established，隧道封装正常
3. 邻居可正常交换 BGP UPDATE 消息

---

## PC-BPG-DAT-042：BVI口邻居建立

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-042 |
| `logic_case_id` | LC-BPG-005 |
| `requirement_ids` | SR-007, SR-007c |
| `feature_tags` | BGP4+, 邻居管理, 接口类型, BVI口 |
| `case_title` | 验证BGP4+邻居通过BVI口建立成功 |
| `priority` | P1 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能 (ASN=100, RouterID=1.1.1.1); BVI口 bvi1 已创建且 IPv6 配置互通; 关联桥接口正常; MTU=1500 |
| `graph_ref` | SP-05 |
| `coverage_goal` | 接口类型覆盖: BVI口邻居建立 Established |
| `trigger_data` | type=BVI口, ip=2001:db8:1::2, iface=bvi1, hop=1, mtu=1500 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | — |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 normal_path "BVI口" 步骤1→2 |
| `action_source_refs` | S04-OP5 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: POST创建邻居 (BVI口 bvi1, MTU=1500)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "bvi1"
        ebgpMultihop: 1
    expected_result: API返回success, BVI口邻居创建成功
    trace_refs:
      - TP-M1-019

  - step_id: STEP-002
    step_name: GET验证邻居状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, 接口=bvi1
    trace_refs:
      - TP-M1-019

  - step_id: STEP-003
    step_name: 清理: DELETE删除邻居
    target: DUT1
    atomic_op:
      op_id: fw_delete_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: API返回success
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | POST创建邻居 (BVI口 bvi1) | DUT1 | fw_create_bgp6_neighbor ip=2001:db8:1::2, asNumber=100, interface=bvi1, ebgpMultihop=1 | API返回success |
| 2 | GET验证邻居状态 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 邻居 Established |
| 3 | 清理: DELETE删除邻居 | DUT1 | fw_delete_bgp6_neighbor id=2001:db8:1::2 | API返回success |

### expected_results

1. BVI口 (bvi1) 邻居创建成功
2. 邻居状态达到 Established，桥接转发正常
3. 邻居可正常交换 BGP UPDATE 消息

---

## PC-BPG-DAT-043：MTU=1280 邻居正常

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-043 |
| `logic_case_id` | LC-BPG-005 |
| `requirement_ids` | SR-007, SR-007c |
| `feature_tags` | BGP4+, 邻居管理, MTU, 边界值 |
| `case_title` | 验证BGP4+邻居接口MTU=1280(IPv6最小MTU)时邻居正常建立 |
| `priority` | P2 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能 (ASN=100, RouterID=1.1.1.1); 物理口 GE0_1 配置 IPv6 互通; 接口 MTU=1280 |
| `graph_ref` | SP-06 |
| `coverage_goal` | 边界守卫: IPv6 最小 MTU=1280 邻居正常 |
| `trigger_data` | type=物理口, ip=2001:db8:1::2, iface=GE0_1, hop=1, mtu=1280 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-022 |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 boundary_path "MTU=1280" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认接口 MTU=1280 已配置
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        name: "GE0_1"
        mtu: 1280
    expected_result: 接口 MTU 配置为 1280
    trace_refs:
      - TP-M1-022

  - step_id: STEP-002
    step_name: POST创建邻居 (MTU=1280)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "GE0_1"
        ebgpMultihop: 1
    expected_result: API返回success
    trace_refs:
      - TP-M1-022

  - step_id: STEP-003
    step_name: GET验证邻居状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, MSS正常协商
    trace_refs:
      - TP-M1-022

  - step_id: STEP-004
    step_name: 清理: DELETE删除邻居, 恢复MTU=1500
    target: DUT1
    atomic_op:
      op_id: fw_delete_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: API返回success
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 配置接口 MTU=1280 | DUT1 | fw_config_interface name=GE0_1, mtu=1280 | MTU 配置生效 |
| 2 | POST创建邻居 | DUT1 | fw_create_bgp6_neighbor ip=2001:db8:1::2, asNumber=100, interface=GE0_1, ebgpMultihop=1 | API返回success |
| 3 | GET验证邻居状态 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 邻居 Established |
| 4 | 清理: DELETE删除邻居 | DUT1 | fw_delete_bgp6_neighbor id=2001:db8:1::2 | API返回success |

### expected_results

1. IPv6 最小 MTU=1280 下邻居正常建立 Established
2. TCP MSS 正常协商，BGP 消息正常交换

---

## PC-BPG-DAT-044：MTU=1279 MSS异常（物理口）

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-044 |
| `logic_case_id` | LC-BPG-005 |
| `requirement_ids` | SR-007, SR-007c |
| `feature_tags` | BGP4+, 邻居管理, MTU, 边界值, 异常路径 |
| `case_title` | 验证BGP4+邻居物理口MTU=1279(低于IPv6最小MTU)时MSS异常 |
| `priority` | P2 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能 (ASN=100, RouterID=1.1.1.1); 物理口 GE0_1 配置 IPv6 互通; 接口 MTU=1279 |
| `graph_ref` | SP-07 |
| `coverage_goal` | 边界守卫: MTU=1279 (min-off) MSS 异常/邻居可能异常 |
| `trigger_data` | type=物理口, ip=2001:db8:1::2, iface=GE0_1, hop=1, mtu=1279 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-022 |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 abnormal_path "MTU=1279 物理口" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 配置接口 MTU=1279
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        name: "GE0_1"
        mtu: 1279
    expected_result: 接口 MTU 配置为 1279
    trace_refs:
      - TP-M1-022

  - step_id: STEP-002
    step_name: POST创建邻居 (MTU=1279)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "GE0_1"
        ebgpMultihop: 1
    expected_result: API返回success (API不校验MTU), 但MSS协商可能异常
    trace_refs:
      - TP-M1-022

  - step_id: STEP-003
    step_name: GET验证邻居状态和MSS
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居可能Established但MSS < IPv6要求的最小值1220, 或邻居建立异常
    trace_refs:
      - TP-M1-022

  - step_id: STEP-004
    step_name: 清理: DELETE删除邻居, 恢复MTU=1500
    target: DUT1
    atomic_op:
      op_id: fw_delete_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: API返回success
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 配置接口 MTU=1279 | DUT1 | fw_config_interface name=GE0_1, mtu=1279 | MTU 配置生效 |
| 2 | POST创建邻居 | DUT1 | fw_create_bgp6_neighbor ip=2001:db8:1::2, asNumber=100, interface=GE0_1, ebgpMultihop=1 | API返回success |
| 3 | GET验证邻居状态 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | MSS异常/邻居异常 |
| 4 | 清理: DELETE删除邻居, 恢复MTU | DUT1 | fw_delete_bgp6_neighbor, fw_config_interface mtu=1500 | API返回success |

### expected_results

1. MTU=1279 低于 IPv6 最小要求 1280
2. TCP MSS 协商异常（小于 RFC 2460 要求的最小值 1220）
3. 邻居可能无法正常交换 UPDATE 消息或无法建立

---

## PC-BPG-DAT-045：MTU=1279 MSS异常（子接口）

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-045 |
| `logic_case_id` | LC-BPG-005 |
| `requirement_ids` | SR-007, SR-007c |
| `feature_tags` | BGP4+, 邻居管理, MTU, 边界值, 异常路径, 子接口 |
| `case_title` | 验证BGP4+邻居子接口MTU=1279时MSS异常 |
| `priority` | P2 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能 (ASN=100, RouterID=1.1.1.1); 子接口 GE0_1.100 已创建且 IPv6 互通; 接口 MTU=1279 |
| `graph_ref` | SP-08 |
| `coverage_goal` | 边界守卫: 子接口 MTU=1279 MSS 异常 |
| `trigger_data` | type=子接口, ip=2001:db8:1::2, iface=GE0_1.100, hop=1, mtu=1279 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-022 |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 abnormal_path "MTU=1279 子接口" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 配置子接口 MTU=1279
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        name: "GE0_1.100"
        mtu: 1279
    expected_result: 子接口 MTU 配置为 1279
    trace_refs:
      - TP-M1-022

  - step_id: STEP-002
    step_name: POST创建邻居 (子接口, MTU=1279)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "GE0_1.100"
        ebgpMultihop: 1
    expected_result: API返回success, 但MSS协商可能异常
    trace_refs:
      - TP-M1-022

  - step_id: STEP-003
    step_name: GET验证邻居状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居可能异常, MSS < 1220
    trace_refs:
      - TP-M1-022

  - step_id: STEP-004
    step_name: 清理: DELETE删除邻居, 恢复MTU=1500
    target: DUT1
    atomic_op:
      op_id: fw_delete_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: API返回success
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 配置子接口 MTU=1279 | DUT1 | fw_config_interface name=GE0_1.100, mtu=1279 | MTU 配置生效 |
| 2 | POST创建邻居 | DUT1 | fw_create_bgp6_neighbor ip=2001:db8:1::2, asNumber=100, interface=GE0_1.100, ebgpMultihop=1 | API返回success |
| 3 | GET验证邻居状态 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | MSS异常 |
| 4 | 清理: DELETE删除邻居 | DUT1 | fw_delete_bgp6_neighbor id=2001:db8:1::2 | API返回success |

### expected_results

1. 子接口 MTU=1279 低于 IPv6 最小要求
2. TCP MSS 协商异常，邻居可能无法正常交换 UPDATE 消息
3. VLAN 标记额外开销可能加剧 MTU 不足问题

---

## PC-BPG-DAT-046：MTU=1279 隧道口异常

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-046 |
| `logic_case_id` | LC-BPG-005 |
| `requirement_ids` | SR-007, SR-007c |
| `feature_tags` | BGP4+, 邻居管理, MTU, 边界值, 异常路径, 隧道口 |
| `case_title` | 验证BGP4+邻居隧道口MTU=1279时隧道封装+小MTU双重影响异常 |
| `priority` | P2 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能 (ASN=100, RouterID=1.1.1.1); 隧道口 tunnel0 已创建且 IPv6 互通; 接口 MTU=1279 |
| `graph_ref` | SP-09 |
| `coverage_goal` | 边界守卫: 隧道口 MTU=1279 隧道封装开销加剧异常 |
| `trigger_data` | type=隧道口, ip=2001:db8:1::2, iface=tunnel0, hop=1, mtu=1279 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | — |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 abnormal_path "MTU=1279 隧道口" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 配置隧道口 MTU=1279
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        name: "tunnel0"
        mtu: 1279
    expected_result: 隧道口 MTU 配置为 1279
    trace_refs:
      - TP-M1-022

  - step_id: STEP-002
    step_name: POST创建邻居 (隧道口, MTU=1279)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "tunnel0"
        ebgpMultihop: 1
    expected_result: API返回success, 但MSS+隧道封装开销叠加导致严重异常
    trace_refs:
      - TP-M1-022

  - step_id: STEP-003
    step_name: GET验证邻居状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居异常或 MSS 极低, 可能无法建立 Established
    trace_refs:
      - TP-M1-022

  - step_id: STEP-004
    step_name: 清理: DELETE删除邻居, 恢复MTU=1500
    target: DUT1
    atomic_op:
      op_id: fw_delete_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: API返回success
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 配置隧道口 MTU=1279 | DUT1 | fw_config_interface name=tunnel0, mtu=1279 | MTU 配置生效 |
| 2 | POST创建邻居 | DUT1 | fw_create_bgp6_neighbor ip=2001:db8:1::2, asNumber=100, interface=tunnel0, ebgpMultihop=1 | API返回success |
| 3 | GET验证邻居状态 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 邻居异常/MSS极低 |
| 4 | 清理: DELETE删除邻居 | DUT1 | fw_delete_bgp6_neighbor id=2001:db8:1::2 | API返回success |

### expected_results

1. 隧道口 MTU=1279 叠加隧道封装开销（通常约 50+ 字节）
2. 有效 MSS 远低于 IPv6 要求，邻居严重异常
3. 邻居可能无法建立 Established 或立即断开

---

## PC-BPG-DAT-047：MTU=0 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-047 |
| `logic_case_id` | LC-BPG-005 |
| `requirement_ids` | SR-007, SR-007c |
| `feature_tags` | BGP4+, 邻居管理, MTU, 异常路径 |
| `case_title` | 验证BGP4+邻居接口MTU=0时拒绝或邻居不可达 |
| `priority` | P2 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能 (ASN=100, RouterID=1.1.1.1); 物理口 GE0_1 已配置 IPv6 |
| `graph_ref` | SP-10 |
| `coverage_goal` | 异常守卫: MTU=0 拒绝 |
| `trigger_data` | type=物理口, ip=2001:db8:1::2, iface=GE0_1, hop=1, mtu=0 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | — |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 abnormal_path "MTU=0" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 尝试配置接口 MTU=0
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        name: "GE0_1"
        mtu: 0
    expected_result: API返回错误, 提示MTU值非法
    trace_refs:
      - TP-M1-022

  - step_id: STEP-002
    step_name: 验证接口 MTU 未变更
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args: {}
    expected_result: 接口配置未受影响
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 尝试配置接口 MTU=0 | DUT1 | fw_config_interface name=GE0_1, mtu=0 | API返回错误 |
| 2 | 验证接口配置未变更 | DUT1 | fw_verify_bgp6_neighbor | 不受影响 |

### expected_results

1. MTU=0 被 API 拒绝或接口不可达
2. 已有配置不受影响

---

## PC-BPG-DAT-048：MTU=-1 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-048 |
| `logic_case_id` | LC-BPG-005 |
| `requirement_ids` | SR-007, SR-007c |
| `feature_tags` | BGP4+, 邻居管理, MTU, 异常路径 |
| `case_title` | 验证BGP4+邻居接口MTU=-1(负数)被拒绝 |
| `priority` | P3 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能 (ASN=100, RouterID=1.1.1.1); 物理口 GE0_1 已配置 IPv6 |
| `graph_ref` | SP-11 |
| `coverage_goal` | 异常守卫: 负数 MTU 拒绝 |
| `trigger_data` | type=物理口, ip=2001:db8:1::2, iface=GE0_1, hop=1, mtu=-1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | — |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 abnormal_path "MTU负数" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 尝试配置接口 MTU=-1
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        name: "GE0_1"
        mtu: -1
    expected_result: API返回错误, 提示MTU值非法
    trace_refs:
      - TP-M1-022

  - step_id: STEP-002
    step_name: 验证接口配置未变更
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args: {}
    expected_result: 接口配置未受影响
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 尝试配置接口 MTU=-1 | DUT1 | fw_config_interface name=GE0_1, mtu=-1 | API返回错误 |
| 2 | 验证接口配置未变更 | DUT1 | fw_verify_bgp6_neighbor | 不受影响 |

### expected_results

1. MTU=-1 (负数) 被 API 拒绝
2. 已有配置不受影响

---

## PC-BPG-DAT-049：接口类型=空拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-049 |
| `logic_case_id` | LC-BPG-005 |
| `requirement_ids` | SR-007, SR-007c |
| `feature_tags` | BGP4+, 邻居管理, 接口类型, 异常路径 |
| `case_title` | 验证BGP4+邻居接口类型为空时创建被拒绝 |
| `priority` | P3 |
| `preconditions` | DUT1 BGP4+全局使能 (ASN=100, RouterID=1.1.1.1) |
| `graph_ref` | SP-12 |
| `coverage_goal` | 异常守卫: 空接口类型拒绝 |
| `trigger_data` | type="", ip=2001:db8:1::2, iface="", hop=1, mtu=1500 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | — |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 abnormal_path "接口类型为空" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: POST创建邻居 (接口="")
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: ""
        ebgpMultihop: 1
    expected_result: API返回错误, 提示接口名为必填项
    trace_refs:
      - TP-M1-014

  - step_id: STEP-002
    step_name: GET验证邻居未创建
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args: {}
    expected_result: 邻居列表中无该条目
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | POST创建邻居 (接口="") | DUT1 | fw_create_bgp6_neighbor ip=2001:db8:1::2, asNumber=100, interface="", ebgpMultihop=1 | API返回错误 |
| 2 | GET验证邻居未创建 | DUT1 | fw_verify_bgp6_neighbor | 无该条目 |

### expected_results

1. 空接口名被 API 拒绝
2. 邻居未被创建

---

## PC-BPG-DAT-050：接口类型=未知拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-050 |
| `logic_case_id` | LC-BPG-005 |
| `requirement_ids` | SR-007, SR-007c |
| `feature_tags` | BGP4+, 邻居管理, 接口类型, 异常路径 |
| `case_title` | 验证BGP4+邻居使用未知/不支持的接口类型创建被拒绝 |
| `priority` | P3 |
| `preconditions` | DUT1 BGP4+全局使能 (ASN=100, RouterID=1.1.1.1) |
| `graph_ref` | SP-13 |
| `coverage_goal` | 异常守卫: 未知接口类型拒绝 |
| `trigger_data` | type=未知, ip=2001:db8:1::2, iface=unknown_virtual_if, hop=1, mtu=1500 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | — |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 abnormal_path "接口类型未知" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: POST创建邻居 (未知接口类型)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "unknown_virtual_if"
        ebgpMultihop: 1
    expected_result: API返回错误, 提示接口不存在或类型不支持
    trace_refs: []

  - step_id: STEP-002
    step_name: GET验证邻居未创建
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args: {}
    expected_result: 邻居列表中无该条目
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | POST创建邻居 (未知接口类型) | DUT1 | fw_create_bgp6_neighbor ip=2001:db8:1::2, asNumber=100, interface=unknown_virtual_if, ebgpMultihop=1 | API返回错误 |
| 2 | GET验证邻居未创建 | DUT1 | fw_verify_bgp6_neighbor | 无该条目 |

### expected_results

1. 未知或不支持的接口类型被 API 拒绝
2. 邻居未被创建

---

## PC-BPG-DAT-051：MTU=68 [待确认]

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-051 |
| `logic_case_id` | LC-BPG-005 |
| `requirement_ids` | SR-007, SR-007c |
| `feature_tags` | BGP4+, 邻居管理, MTU, 边界值 |
| `case_title` | 验证BGP4+邻居接口MTU=68(远低于IPv6最小MTU)的行为 [待确认] |
| `priority` | P3 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能 (ASN=100, RouterID=1.1.1.1); 物理口 GE0_1 已配置 IPv6; 接口 MTU=68 |
| `graph_ref` | SP-14 |
| `coverage_goal` | 边界探查: MTU=68 低于 IPv4 最小, 设备行为待确认 |
| `trigger_data` | type=物理口, ip=2001:db8:1::2, iface=GE0_1, hop=1, mtu=68 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | — |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 abnormal_path "MTU=68" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | LC-BPG-005-GAP-MTU-68 |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 尝试配置接口 MTU=68
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        name: "GE0_1"
        mtu: 68
    expected_result: "[待确认] 设备可能拒绝(MTU值非法) 或 接受但邻居无法建立"
    trace_refs: []
    confirmation_gap_refs:
      - LC-BPG-005-GAP-MTU-68

  - step_id: STEP-002
    step_name: POST创建邻居 (MTU=68)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "GE0_1"
        ebgpMultihop: 1
    expected_result: "[待确认] 若接口MTU=68, MSS极低, 邻居几乎不可能建立"
    trace_refs: []
    confirmation_gap_refs:
      - LC-BPG-005-GAP-MTU-68

  - step_id: STEP-003
    step_name: 清理: 恢复MTU=1500
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        name: "GE0_1"
        mtu: 1500
    expected_result: API返回success
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 尝试配置接口 MTU=68 | DUT1 | fw_config_interface name=GE0_1, mtu=68 | [待确认] |
| 2 | POST创建邻居 (MTU=68) | DUT1 | fw_create_bgp6_neighbor ip=2001:db8:1::2, asNumber=100, interface=GE0_1, ebgpMultihop=1 | [待确认] |
| 3 | 清理: 恢复MTU=1500 | DUT1 | fw_config_interface name=GE0_1, mtu=1500 | API返回success |

### expected_results

1. [待确认] MTU=68 是否被设备接受（68 为 IPv4 最小 MTU）
2. 若接受，MSS 极低（约 28 字节），BGP 邻居几乎无法建立
3. 若拒绝，接口 MTU 配置未变，接口保持正常

---

## PC-BPG-DAT-052：MTU=9000 [待确认]

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-052 |
| `logic_case_id` | LC-BPG-005 |
| `requirement_ids` | SR-007, SR-007c |
| `feature_tags` | BGP4+, 邻居管理, MTU, 边界值 |
| `case_title` | 验证BGP4+邻居接口MTU=9000(巨帧)的行为 [待确认] |
| `priority` | P3 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能 (ASN=100, RouterID=1.1.1.1); 物理口 GE0_1 已配置 IPv6; 接口 MTU=9000 |
| `graph_ref` | SP-15 |
| `coverage_goal` | 边界探查: 巨帧 MTU=9000 是否支持 |
| `trigger_data` | type=物理口, ip=2001:db8:1::2, iface=GE0_1, hop=1, mtu=9000 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | — |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 boundary_path "MTU=9000 jumbo" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | LC-BPG-005-GAP-MTU-JUMBO |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 尝试配置接口 MTU=9000
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        name: "GE0_1"
        mtu: 9000
    expected_result: "[待确认] 设备可能接受(支持巨帧) 或 拒绝(超出硬件MTU上限)"
    trace_refs: []
    confirmation_gap_refs:
      - LC-BPG-005-GAP-MTU-JUMBO

  - step_id: STEP-002
    step_name: POST创建邻居 (MTU=9000)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "GE0_1"
        ebgpMultihop: 1
    expected_result: "[待确认] 若MTU=9000被接受, 邻居应正常Established且MSS更大"
    trace_refs: []
    confirmation_gap_refs:
      - LC-BPG-005-GAP-MTU-JUMBO

  - step_id: STEP-003
    step_name: GET验证邻居状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: "[待确认] 若支持则Established且MSS=8960; 否则MTU配置失败"
    trace_refs: []
    confirmation_gap_refs:
      - LC-BPG-005-GAP-MTU-JUMBO

  - step_id: STEP-004
    step_name: 清理: DELETE删除邻居, 恢复MTU=1500
    target: DUT1
    atomic_op:
      op_id: fw_delete_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: API返回success
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 尝试配置接口 MTU=9000 | DUT1 | fw_config_interface name=GE0_1, mtu=9000 | [待确认] |
| 2 | POST创建邻居 (MTU=9000) | DUT1 | fw_create_bgp6_neighbor ip=2001:db8:1::2, asNumber=100, interface=GE0_1, ebgpMultihop=1 | [待确认] |
| 3 | GET验证邻居状态 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | [待确认] |
| 4 | 清理: DELETE删除邻居, 恢复MTU | DUT1 | fw_delete_bgp6_neighbor, fw_config_interface mtu=1500 | API返回success |

### expected_results

1. [待确认] 巨帧 MTU=9000 是否支持取决于设备硬件能力
2. 若支持: 邻居正常 Established，MSS 更大（约 8960），大包 UPDATE 消息效率更高
3. 若不支持: MTU 配置被拒绝，接口保持现有 MTU

---
