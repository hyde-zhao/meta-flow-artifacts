# BGP4+ 邻居 CRUD — 物理用例

> 生成时间: 2026-06-12 | 设计Skill: data-design
> 逻辑用例: LC-BPG-003 | 目录: BGP-配置管理-BPG配置

---

## PC-BPG-DAT-024：邻居 CRUD 正常流程

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-024 |
| `logic_case_id` | LC-BPG-003 |
| `requirement_ids` | SR-007, SR-007a, SR-007b, SR-007c, SR-007d |
| `feature_tags` | BGP4+, 邻居管理, CRUD |
| `case_title` | 验证BGP4+邻居创建→查询→修改→删除全生命周期正常流程 |
| `priority` | P1 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能 (asNumber=100, routerId=1.1.1.1); 接口 GE0_1/GE0_2 IPv6 已配置互通 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 主生命周期: 邻居 CRUD 全覆盖 (创建→查询→修改→批量查询→删除→空列表) |
| `trigger_data` | neighbor1: ip=2001:db8:1::2, iface=GE0_1, hop=1; neighbor2: ip=fd00:192:168:1::1, iface=GE0_2, hop=5 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-010, TP-M1-013 |
| `scenario_refs` | S03 |
| `scenario_chain_refs` | S03 normal_path 步骤1→2→3→4→5 |
| `action_source_refs` | S03-OP1, S03-OP2, S03-OP3, S03-OP4 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: POST创建邻居1 (2001:db8:1::2, GE0_1, multihop=1)
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
      - TP-M1-010

  - step_id: STEP-002
    step_name: POST创建邻居2 (fd00:192:168:1::1, GE0_2, multihop=5)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "fd00:192:168:1::1"
        asNumber: 100
        interface: "GE0_2"
        ebgpMultihop: 5
    expected_result: API返回success
    trace_refs:
      - TP-M1-010

  - step_id: STEP-003
    step_name: GET查询单个邻居状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居存在, 状态为 Established 或 Active
    trace_refs:
      - TP-M1-013

  - step_id: STEP-004
    step_name: PUT修改邻居1 multihop 值
    target: DUT1
    atomic_op:
      op_id: fw_update_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
        ebgpMultihop: 3
    expected_result: API返回success, 修改生效
    trace_refs:
      - TP-M1-013

  - step_id: STEP-005
    step_name: GET批量查询所有邻居
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args: {}
    expected_result: 返回2个邻居条目, 邻居1 multihop=3
    trace_refs:
      - TP-M1-013

  - step_id: STEP-006
    step_name: DELETE删除邻居1
    target: DUT1
    atomic_op:
      op_id: fw_delete_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: API返回success, 邻居1已删除
    trace_refs:
      - TP-M1-013

  - step_id: STEP-007
    step_name: DELETE删除邻居2
    target: DUT1
    atomic_op:
      op_id: fw_delete_bgp6_neighbor
      args:
        id: "fd00:192:168:1::1"
    expected_result: API返回success
    trace_refs:
      - TP-M1-013

  - step_id: STEP-008
    step_name: GET验证邻居列表为空
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args: {}
    expected_result: 邻居列表为空, 无残留
    trace_refs:
      - TP-M1-013
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | POST创建邻居1 | DUT1 | fw_create_bgp6_neighbor ip=2001:db8:1::2, asNumber=100, interface=GE0_1, ebgpMultihop=1 | API返回success |
| 2 | POST创建邻居2 | DUT1 | fw_create_bgp6_neighbor ip=fd00:192:168:1::1, asNumber=100, interface=GE0_2, ebgpMultihop=5 | API返回success |
| 3 | GET查询单个邻居 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 邻居存在 |
| 4 | PUT修改邻居multihop | DUT1 | fw_update_bgp6_neighbor id=2001:db8:1::2, ebgpMultihop=3 | API返回success |
| 5 | GET批量查询 | DUT1 | fw_verify_bgp6_neighbor | 返回2个邻居 |
| 6 | DELETE删除邻居1 | DUT1 | fw_delete_bgp6_neighbor id=2001:db8:1::2 | API返回success |
| 7 | DELETE删除邻居2 | DUT1 | fw_delete_bgp6_neighbor id=fd00:192:168:1::1 | API返回success |
| 8 | GET验证空列表 | DUT1 | fw_verify_bgp6_neighbor | 邻居列表为空 |

### expected_results

1. 邻居创建成功，POST 返回 success
2. 邻居查询返回正确的配置和状态信息
3. 邻居修改生效，multihop 值更新
4. 邻居删除成功，列表恢复为空
5. CRUD 全流程无残留或状态异常

---

## PC-BPG-DAT-025：multihop max=255

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-025 |
| `logic_case_id` | LC-BPG-003 |
| `requirement_ids` | SR-007d |
| `feature_tags` | BGP4+, 邻居管理, 边界值 |
| `case_title` | 验证BGP4+邻居使用最大multihop=255创建成功 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+全局使能 (ASN=100); 接口 GE0_1 配置 IPv6 地址且路由可达 |
| `graph_ref` | SP-02 |
| `coverage_goal` | 边界守卫: multihop max-on=255 合法接收 |
| `trigger_data` | ip=2001:db8:1::2, iface=GE0_1, hop=255 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-015 |
| `scenario_refs` | S03 |
| `scenario_chain_refs` | S03 boundary_path "multihop=max" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: POST创建邻居 (multihop=255)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "GE0_1"
        ebgpMultihop: 255
    expected_result: API返回success, 最大multihop被接受
    trace_refs:
      - TP-M1-015

  - step_id: STEP-002
    step_name: GET验证邻居配置
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居存在, ebgpMultihop=255
    trace_refs:
      - TP-M1-015

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
| 1 | POST创建邻居 (multihop=255) | DUT1 | fw_create_bgp6_neighbor ip=2001:db8:1::2, asNumber=100, interface=GE0_1, ebgpMultihop=255 | API返回success |
| 2 | GET验证邻居配置 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | ebgpMultihop=255 |
| 3 | 清理: DELETE删除邻居 | DUT1 | fw_delete_bgp6_neighbor id=2001:db8:1::2 | API返回success |

### expected_results

1. multihop=255 被正常接受，邻居创建成功
2. 配置查询返回 ebgpMultihop=255

---

## PC-BPG-DAT-026：邻居 IP=::1 回环拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-026 |
| `logic_case_id` | LC-BPG-003 |
| `requirement_ids` | SR-007a |
| `feature_tags` | BGP4+, 邻居管理, 异常路径 |
| `case_title` | 验证BGP4+邻居IP=::1(回环地址)创建被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+全局使能 (ASN=100); 接口 GE0_1 已配置 |
| `graph_ref` | SP-03 |
| `coverage_goal` | 异常守卫: 回环地址不可作为邻居 IP |
| `trigger_data` | ip=::1, iface=GE0_1, hop=1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-011 |
| `scenario_refs` | S03 |
| `scenario_chain_refs` | S03 abnormal_path "IP=回环" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: POST创建邻居 (IP=::1)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "::1"
        asNumber: 100
        interface: "GE0_1"
        ebgpMultihop: 1
    expected_result: API返回错误, 提示不能使用回环地址
    trace_refs:
      - TP-M1-011

  - step_id: STEP-002
    step_name: GET验证邻居未创建
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args: {}
    expected_result: 邻居列表中无 ::1 条目
    trace_refs:
      - TP-M1-011
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | POST创建邻居 (IP=::1) | DUT1 | fw_create_bgp6_neighbor ip=::1, asNumber=100, interface=GE0_1, ebgpMultihop=1 | API返回错误 |
| 2 | GET验证邻居未创建 | DUT1 | fw_verify_bgp6_neighbor | 无 ::1 条目 |

### expected_results

1. 回环地址 ::1 被 API 拒绝
2. 邻居未被创建

---

## PC-BPG-DAT-027：邻居 IP=ff02::1 组播拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-027 |
| `logic_case_id` | LC-BPG-003 |
| `requirement_ids` | SR-007a |
| `feature_tags` | BGP4+, 邻居管理, 异常路径 |
| `case_title` | 验证BGP4+邻居IP=ff02::1(组播地址)创建被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+全局使能 (ASN=100); 接口 GE0_1 已配置 |
| `graph_ref` | SP-04 |
| `coverage_goal` | 异常守卫: 组播地址不可作为邻居 IP |
| `trigger_data` | ip=ff02::1, iface=GE0_1, hop=1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-011 |
| `scenario_refs` | S03 |
| `scenario_chain_refs` | S03 abnormal_path "IP=组播" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: POST创建邻居 (IP=ff02::1)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "ff02::1"
        asNumber: 100
        interface: "GE0_1"
        ebgpMultihop: 1
    expected_result: API返回错误, 提示不能使用组播地址
    trace_refs:
      - TP-M1-011

  - step_id: STEP-002
    step_name: GET验证邻居未创建
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args: {}
    expected_result: 邻居列表中无 ff02::1 条目
    trace_refs:
      - TP-M1-011
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | POST创建邻居 (IP=ff02::1) | DUT1 | fw_create_bgp6_neighbor ip=ff02::1, asNumber=100, interface=GE0_1, ebgpMultihop=1 | API返回错误 |
| 2 | GET验证邻居未创建 | DUT1 | fw_verify_bgp6_neighbor | 无 ff02::1 条目 |

### expected_results

1. 组播地址 ff02::1 (ff00::/8) 被 API 拒绝
2. 邻居未被创建

---

## PC-BPG-DAT-028：邻居 IP=:: 零地址拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-028 |
| `logic_case_id` | LC-BPG-003 |
| `requirement_ids` | SR-007a |
| `feature_tags` | BGP4+, 邻居管理, 异常路径 |
| `case_title` | 验证BGP4+邻居IP=::(零地址/未指定)创建被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+全局使能 (ASN=100); 接口 GE0_1 已配置 |
| `graph_ref` | SP-05 |
| `coverage_goal` | 异常守卫: 零地址不可作为邻居 IP |
| `trigger_data` | ip=::, iface=GE0_1, hop=1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-011 |
| `scenario_refs` | S03 |
| `scenario_chain_refs` | S03 abnormal_path "IP=零地址" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: POST创建邻居 (IP=::)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "::"
        asNumber: 100
        interface: "GE0_1"
        ebgpMultihop: 1
    expected_result: API返回错误, 提示IP地址不能为全零
    trace_refs:
      - TP-M1-011

  - step_id: STEP-002
    step_name: GET验证邻居未创建
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args: {}
    expected_result: 邻居列表中无 :: 条目
    trace_refs:
      - TP-M1-011
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | POST创建邻居 (IP=::) | DUT1 | fw_create_bgp6_neighbor ip=::, asNumber=100, interface=GE0_1, ebgpMultihop=1 | API返回错误 |
| 2 | GET验证邻居未创建 | DUT1 | fw_verify_bgp6_neighbor | 无 :: 条目 |

### expected_results

1. 零地址 :: 被 API 拒绝
2. 邻居未被创建

---

## PC-BPG-DAT-029：邻居 IP=IPv4 格式拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-029 |
| `logic_case_id` | LC-BPG-003 |
| `requirement_ids` | SR-007a |
| `feature_tags` | BGP4+, 邻居管理, 异常路径 |
| `case_title` | 验证BGP4+邻居IP使用IPv4格式被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+全局使能 (ASN=100); 接口 GE0_1 已配置 |
| `graph_ref` | SP-06 |
| `coverage_goal` | 异常守卫: IPv4 格式 IP 不可作为 BGP4+ 邻居 |
| `trigger_data` | ip=192.168.1.1, iface=GE0_1, hop=1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-011 |
| `scenario_refs` | S03 |
| `scenario_chain_refs` | S03 abnormal_path "IP=IPv4格式" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: POST创建邻居 (IP=192.168.1.1)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "192.168.1.1"
        asNumber: 100
        interface: "GE0_1"
        ebgpMultihop: 1
    expected_result: API返回错误, 提示必须使用IPv6地址
    trace_refs:
      - TP-M1-011

  - step_id: STEP-002
    step_name: GET验证邻居未创建
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args: {}
    expected_result: 邻居列表中无 192.168.1.1 条目
    trace_refs:
      - TP-M1-011
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | POST创建邻居 (IP=IPv4格式) | DUT1 | fw_create_bgp6_neighbor ip=192.168.1.1, asNumber=100, interface=GE0_1, ebgpMultihop=1 | API返回错误 |
| 2 | GET验证邻居未创建 | DUT1 | fw_verify_bgp6_neighbor | 无该条目 |

### expected_results

1. IPv4 格式地址 192.168.1.1 被 API 拒绝
2. 邻居未被创建

---

## PC-BPG-DAT-030：接口=空串拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-030 |
| `logic_case_id` | LC-BPG-003 |
| `requirement_ids` | SR-007c |
| `feature_tags` | BGP4+, 邻居管理, 异常路径 |
| `case_title` | 验证BGP4+邻居接口名为空串创建被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+全局使能 (ASN=100) |
| `graph_ref` | SP-07 |
| `coverage_goal` | 异常守卫: 空接口名拒绝 |
| `trigger_data` | ip=2001:db8:1::2, iface="", hop=1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-014 |
| `scenario_refs` | S03 |
| `scenario_chain_refs` | S03 abnormal_path "接口为空" |
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
    trace_refs:
      - TP-M1-014
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

## PC-BPG-DAT-031：multihop=0 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-031 |
| `logic_case_id` | LC-BPG-003 |
| `requirement_ids` | SR-007d |
| `feature_tags` | BGP4+, 邻居管理, 边界值, 异常路径 |
| `case_title` | 验证BGP4+邻居multihop=0创建被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+全局使能 (ASN=100); 接口 GE0_1 已配置 |
| `graph_ref` | SP-08 |
| `coverage_goal` | 边界守卫: multihop min-off=0 拒绝 |
| `trigger_data` | ip=2001:db8:1::2, iface=GE0_1, hop=0 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-015 |
| `scenario_refs` | S03 |
| `scenario_chain_refs` | S03 abnormal_path "multihop=0" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: POST创建邻居 (multihop=0)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "GE0_1"
        ebgpMultihop: 0
    expected_result: API返回错误, 提示multihop范围1~255
    trace_refs:
      - TP-M1-015

  - step_id: STEP-002
    step_name: GET验证邻居未创建
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args: {}
    expected_result: 邻居列表中无该条目
    trace_refs:
      - TP-M1-015
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | POST创建邻居 (multihop=0) | DUT1 | fw_create_bgp6_neighbor ip=2001:db8:1::2, asNumber=100, interface=GE0_1, ebgpMultihop=0 | API返回错误 |
| 2 | GET验证邻居未创建 | DUT1 | fw_verify_bgp6_neighbor | 无该条目 |

### expected_results

1. multihop=0 被 API 拒绝
2. 邻居未被创建

---

## PC-BPG-DAT-032：multihop=256 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-032 |
| `logic_case_id` | LC-BPG-003 |
| `requirement_ids` | SR-007d |
| `feature_tags` | BGP4+, 邻居管理, 边界值, 异常路径 |
| `case_title` | 验证BGP4+邻居multihop=256(超出uint8)创建被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+全局使能 (ASN=100); 接口 GE0_1 已配置 |
| `graph_ref` | SP-09 |
| `coverage_goal` | 边界守卫: multihop max-off=256 拒绝 |
| `trigger_data` | ip=2001:db8:1::2, iface=GE0_1, hop=256 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-015 |
| `scenario_refs` | S03 |
| `scenario_chain_refs` | S03 abnormal_path "multihop=256" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: POST创建邻居 (multihop=256)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "GE0_1"
        ebgpMultihop: 256
    expected_result: API返回错误, 提示multihop超出范围
    trace_refs:
      - TP-M1-015

  - step_id: STEP-002
    step_name: GET验证邻居未创建
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args: {}
    expected_result: 邻居列表中无该条目
    trace_refs:
      - TP-M1-015
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | POST创建邻居 (multihop=256) | DUT1 | fw_create_bgp6_neighbor ip=2001:db8:1::2, asNumber=100, interface=GE0_1, ebgpMultihop=256 | API返回错误 |
| 2 | GET验证邻居未创建 | DUT1 | fw_verify_bgp6_neighbor | 无该条目 |

### expected_results

1. multihop=256 被 API 拒绝
2. 邻居未被创建

---

## PC-BPG-DAT-033：接口不存在拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-033 |
| `logic_case_id` | LC-BPG-003 |
| `requirement_ids` | SR-007c |
| `feature_tags` | BGP4+, 邻居管理, 异常路径 |
| `case_title` | 验证BGP4+邻居接口名不存在时创建被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+全局使能 (ASN=100); 接口 GE99_99 不存在 |
| `graph_ref` | SP-10 |
| `coverage_goal` | 异常守卫: 不存在的接口拒绝 |
| `trigger_data` | ip=2001:db8:1::2, iface=GE99_99, hop=1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | — |
| `scenario_refs` | S03 |
| `scenario_chain_refs` | S03 abnormal_path "接口不存在" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: POST创建邻居 (接口=GE99_99)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "GE99_99"
        ebgpMultihop: 1
    expected_result: API返回错误, 提示接口不存在
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
| 1 | POST创建邻居 (接口不存在) | DUT1 | fw_create_bgp6_neighbor ip=2001:db8:1::2, asNumber=100, interface=GE99_99, ebgpMultihop=1 | API返回错误 |
| 2 | GET验证邻居未创建 | DUT1 | fw_verify_bgp6_neighbor | 无该条目 |

### expected_results

1. 不存在的接口名 GE99_99 被 API 拒绝
2. 邻居未被创建

---

## PC-BPG-DAT-034：multihop=-1 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-034 |
| `logic_case_id` | LC-BPG-003 |
| `requirement_ids` | SR-007d |
| `feature_tags` | BGP4+, 邻居管理, 异常路径 |
| `case_title` | 验证BGP4+邻居multihop=-1(负数)创建被拒绝 |
| `priority` | P3 |
| `preconditions` | DUT1 BGP4+全局使能 (ASN=100); 接口 GE0_1 已配置 |
| `graph_ref` | SP-11 |
| `coverage_goal` | 异常守卫: 负数 multihop 拒绝 |
| `trigger_data` | ip=2001:db8:1::2, iface=GE0_1, hop=-1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | — |
| `scenario_refs` | S03 |
| `scenario_chain_refs` | S03 abnormal_path "multihop负数" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: POST创建邻居 (multihop=-1)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "GE0_1"
        ebgpMultihop: -1
    expected_result: API返回错误, 提示值非法
    trace_refs:
      - TP-M1-015

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
| 1 | POST创建邻居 (multihop=-1) | DUT1 | fw_create_bgp6_neighbor ip=2001:db8:1::2, asNumber=100, interface=GE0_1, ebgpMultihop=-1 | API返回错误 |
| 2 | GET验证邻居未创建 | DUT1 | fw_verify_bgp6_neighbor | 无该条目 |

### expected_results

1. multihop=-1 (负数) 被 API 拒绝
2. 邻居未被创建

---

## PC-BPG-DAT-035：multihop=null 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-035 |
| `logic_case_id` | LC-BPG-003 |
| `requirement_ids` | SR-007d |
| `feature_tags` | BGP4+, 邻居管理, 异常路径 |
| `case_title` | 验证BGP4+邻居multihop=null创建被拒绝或取默认值 |
| `priority` | P3 |
| `preconditions` | DUT1 BGP4+全局使能 (ASN=100); 接口 GE0_1 已配置 |
| `graph_ref` | SP-12 |
| `coverage_goal` | 异常守卫: multihop 空值行为 |
| `trigger_data` | ip=2001:db8:1::2, iface=GE0_1, hop=null |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | — |
| `scenario_refs` | S03 |
| `scenario_chain_refs` | S03 abnormal_path "multihop为空" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: POST创建邻居 (multihop=null)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "GE0_1"
        ebgpMultihop: null
    expected_result: API返回错误(必填校验) 或 默认取1
    trace_refs:
      - TP-M1-015

  - step_id: STEP-002
    step_name: GET验证状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args: {}
    expected_result: 若默认取1则邻居创建成功且ebgpMultihop=1; 若拒绝则邻居不存在
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | POST创建邻居 (multihop=null) | DUT1 | fw_create_bgp6_neighbor ip=2001:db8:1::2, asNumber=100, interface=GE0_1, ebgpMultihop=null | 返回错误或取默认值1 |
| 2 | GET验证状态 | DUT1 | fw_verify_bgp6_neighbor | 根据结果确认 |

### expected_results

1. multihop=null 被拒绝或默认取 1
2. 系统状态保持一致

---

## PC-BPG-DAT-036：fe80::1 link-local 地址 [待确认]

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-036 |
| `logic_case_id` | LC-BPG-003 |
| `requirement_ids` | SR-007a |
| `feature_tags` | BGP4+, 邻居管理, 边界值 |
| `case_title` | 验证BGP4+邻居IP=fe80::1(链路本地地址)的行为 [待确认] |
| `priority` | P3 |
| `preconditions` | DUT1 BGP4+全局使能 (ASN=100); 接口 GE0_1 已配置 |
| `graph_ref` | SP-13 |
| `coverage_goal` | 边界探查: link-local 地址是否允许作为邻居 IP |
| `trigger_data` | ip=fe80::1, iface=GE0_1, hop=1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | — |
| `scenario_refs` | S03 |
| `scenario_chain_refs` | S03 abnormal_path "IP=link-local" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | LC-BPG-003-GAP-LL |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: POST创建邻居 (IP=fe80::1)
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "fe80::1"
        asNumber: 100
        interface: "GE0_1"
        ebgpMultihop: 1
    expected_result: "[待确认] API可能接受(link-local可用) 或 拒绝(fe80::/10视为特殊情况)"
    trace_refs: []
    confirmation_gap_refs:
      - LC-BPG-003-GAP-LL

  - step_id: STEP-002
    step_name: GET验证邻居状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "fe80::1"
    expected_result: "[待确认] 若接受则邻居存在且可能Established; 若拒绝则不存在"
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | POST创建邻居 (IP=fe80::1) | DUT1 | fw_create_bgp6_neighbor ip=fe80::1, asNumber=100, interface=GE0_1, ebgpMultihop=1 | [待确认] |
| 2 | GET验证邻居状态 | DUT1 | fw_verify_bgp6_neighbor id=fe80::1 | [待确认] |

### expected_results

1. [待确认] 链路本地地址 fe80::1 是否允许作为 BGP4+ 邻居 IP，取决于实现
2. 场景A: 接受为合法邻居 IP
3. 场景B: 拒绝，提示 link-local 地址不可用

---

## PC-BPG-DAT-037：单字符接口名 [待确认]

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-037 |
| `logic_case_id` | LC-BPG-003 |
| `requirement_ids` | SR-007c |
| `feature_tags` | BGP4+, 邻居管理, 边界值 |
| `case_title` | 验证BGP4+邻居使用单字符接口名的行为 [待确认] |
| `priority` | P3 |
| `preconditions` | DUT1 BGP4+全局使能 (ASN=100); 确认接口 "A" 是否存在 |
| `graph_ref` | SP-14 |
| `coverage_goal` | 边界探查: 单字符接口名是否合法 |
| `trigger_data` | ip=2001:db8:1::2, iface=A, hop=1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | — |
| `scenario_refs` | S03 |
| `scenario_chain_refs` | S03 abnormal_path "接口=单字符" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | LC-BPG-003-GAP-IFACE-1CHAR |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: POST创建邻居 (接口="A")
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "A"
        ebgpMultihop: 1
    expected_result: "[待确认] 若接口A存在且UP则可能接受; 若不存在或名称非法则拒绝"
    trace_refs: []
    confirmation_gap_refs:
      - LC-BPG-003-GAP-IFACE-1CHAR

  - step_id: STEP-002
    step_name: GET验证邻居状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args: {}
    expected_result: "[待确认] 根据结果确认单字符接口名是否合法"
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | POST创建邻居 (接口="A") | DUT1 | fw_create_bgp6_neighbor ip=2001:db8:1::2, asNumber=100, interface=A, ebgpMultihop=1 | [待确认] |
| 2 | GET验证邻居状态 | DUT1 | fw_verify_bgp6_neighbor | [待确认] |

### expected_results

1. [待确认] 单字符接口名是否被接受取决于设备接口命名规则和是否存在
2. 若接口 "A" 不存在或命名规则禁止单字符，则拒绝
3. 若接口 "A" 存在且合法，则可能接受

---
