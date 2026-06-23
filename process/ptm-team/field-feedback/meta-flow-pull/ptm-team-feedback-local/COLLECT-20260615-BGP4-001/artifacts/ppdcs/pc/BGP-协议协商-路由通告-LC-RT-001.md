# BGP4+ 路由引入 — 物理用例

> 生成时间: 2026-06-12 | 设计Skill: parameter-design
> 逻辑用例: LC-RT-001 | 目录: BGP-协议协商-路由通告

---

## PC-RT-PAR-001：直连路由引入，MED=100

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-PAR-001 |
| `logic_case_id` | LC-RT-001 |
| `requirement_ids` | SR-RT-001 |
| `feature_tags` | BGP4+, 路由引入, connected, MED |
| `case_title` | 验证将connected直连路由引入BGP4+，MED=100，对端学到直连路由 |
| `priority` | P1 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能, IBGP邻居已Established, DUT1存在直连路由 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 路由引入基线: connected协议 + MED=100 |
| `trigger_data` | protocol=connected, med=100 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-RT-001 |
| `scenario_refs` | S01 |
| `scenario_chain_refs` | S01 normal_path |
| `action_source_refs` | DR-RT-001 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认IBGP邻居Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established
    trace_refs:
      - DR-RT-001

  - step_id: STEP-002
    step_name: DUT1配置connected路由引入，MED=100
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_redist
      args:
        protocol: connected
        med: 100
    expected_result: API返回success, 路由引入配置成功
    trace_refs:
      - DR-RT-001

  - step_id: STEP-003
    step_name: 对端DUT2验证学到直连路由
    target: DUT2
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "[DUT1直连路由前缀]"
    expected_result: BGP路由表中存在DUT1直连路由, MED=100
    trace_refs:
      - DR-RT-001
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认IBGP邻居Established | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 状态=Established |
| 2 | 配置connected路由引入MED=100 | DUT1 | fw_config_bgp6_redist protocol=connected, med=100 | API返回success |
| 3 | 验证对端学到直连路由 | DUT2 | fw_verify_bgp6_route | 路由存在, MED=100 |

### expected_results

1. DUT1 成功将 connected 直连路由引入 BGP4+
2. DUT2 BGP 路由表中学到该直连路由
3. 路由属性 MED=100

---

## PC-RT-PAR-002：静态路由引入，MED=50

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-PAR-002 |
| `logic_case_id` | LC-RT-001 |
| `requirement_ids` | SR-RT-001 |
| `feature_tags` | BGP4+, 路由引入, static, MED |
| `case_title` | 验证将static静态路由引入BGP4+，MED=50，对端学到静态路由 |
| `priority` | P1 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能, IBGP邻居已Established, DUT1存在静态路由 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 路由引入基线: static协议 + MED=50 |
| `trigger_data` | protocol=static, med=50 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-RT-002 |
| `scenario_refs` | S01 |
| `scenario_chain_refs` | S01 normal_path |
| `action_source_refs` | DR-RT-002 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认IBGP邻居Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established
    trace_refs:
      - DR-RT-002

  - step_id: STEP-002
    step_name: DUT1配置static路由引入，MED=50
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_redist
      args:
        protocol: static
        med: 50
    expected_result: API返回success, 路由引入配置成功
    trace_refs:
      - DR-RT-002

  - step_id: STEP-003
    step_name: 对端DUT2验证学到静态路由
    target: DUT2
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "[DUT1静态路由前缀]"
    expected_result: BGP路由表中存在DUT1静态路由, MED=50
    trace_refs:
      - DR-RT-002
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认IBGP邻居Established | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 状态=Established |
| 2 | 配置static路由引入MED=50 | DUT1 | fw_config_bgp6_redist protocol=static, med=50 | API返回success |
| 3 | 验证对端学到静态路由 | DUT2 | fw_verify_bgp6_route | 路由存在, MED=50 |

### expected_results

1. DUT1 成功将 static 静态路由引入 BGP4+
2. DUT2 BGP 路由表中学到该静态路由
3. 路由属性 MED=50

---

## PC-RT-PAR-003：OSPFv3路由引入，MED=200

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-PAR-003 |
| `logic_case_id` | LC-RT-001 |
| `requirement_ids` | SR-RT-001 |
| `feature_tags` | BGP4+, 路由引入, ospfv3, MED |
| `case_title` | 验证将ospfv3路由引入BGP4+，MED=200，对端学到OSPFv3路由 |
| `priority` | P1 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能, IBGP邻居已Established, DUT1存在OSPFv3路由 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 路由引入基线: ospfv3协议 + MED=200 |
| `trigger_data` | protocol=ospfv3, med=200 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-RT-003 |
| `scenario_refs` | S01 |
| `scenario_chain_refs` | S01 normal_path |
| `action_source_refs` | DR-RT-003 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认IBGP邻居Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established
    trace_refs:
      - DR-RT-003

  - step_id: STEP-002
    step_name: DUT1配置ospfv3路由引入，MED=200
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_redist
      args:
        protocol: ospfv3
        med: 200
    expected_result: API返回success, 路由引入配置成功
    trace_refs:
      - DR-RT-003

  - step_id: STEP-003
    step_name: 对端DUT2验证学到OSPFv3路由
    target: DUT2
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "[DUT1 OSPFv3路由前缀]"
    expected_result: BGP路由表中存在DUT1 OSPFv3路由, MED=200
    trace_refs:
      - DR-RT-003
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认IBGP邻居Established | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 状态=Established |
| 2 | 配置ospfv3路由引入MED=200 | DUT1 | fw_config_bgp6_redist protocol=ospfv3, med=200 | API返回success |
| 3 | 验证对端学到OSPFv3路由 | DUT2 | fw_verify_bgp6_route | 路由存在, MED=200 |

### expected_results

1. DUT1 成功将 ospfv3 路由引入 BGP4+
2. DUT2 BGP 路由表中学到该 OSPFv3 路由
3. 路由属性 MED=200

---

## PC-RT-PAR-004：MED最小边界值验证，MED=0

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-PAR-004 |
| `logic_case_id` | LC-RT-001 |
| `requirement_ids` | SR-RT-001 |
| `feature_tags` | BGP4+, 路由引入, MED边界, connected |
| `case_title` | 验证MED=0最小边界值时路由引入成功 |
| `priority` | P2 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能, IBGP邻居已Established |
| `graph_ref` | SP-01 |
| `coverage_goal` | MED边界守卫: 最小值0 |
| `trigger_data` | protocol=connected, med=0 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-RT-004 |
| `scenario_refs` | S01 |
| `scenario_chain_refs` | S01 boundary_path |
| `action_source_refs` | DR-RT-004 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认IBGP邻居Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established
    trace_refs:
      - DR-RT-004

  - step_id: STEP-002
    step_name: DUT1配置connected路由引入，MED=0
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_redist
      args:
        protocol: connected
        med: 0
    expected_result: API返回success, 接受MED=0
    trace_refs:
      - DR-RT-004

  - step_id: STEP-003
    step_name: 对端DUT2验证学到路由且MED=0
    target: DUT2
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "[DUT1直连路由前缀]"
    expected_result: BGP路由表中路由MED=0
    trace_refs:
      - DR-RT-004
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认IBGP邻居Established | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 状态=Established |
| 2 | 配置路由引入MED=0 | DUT1 | fw_config_bgp6_redist protocol=connected, med=0 | API返回success |
| 3 | 验证对端路由MED=0 | DUT2 | fw_verify_bgp6_route | 路由存在, MED=0 |

### expected_results

1. MED=0 为合法最小边界值，API 接受配置
2. 对端正确学到路由，MED=0
3. 路由转发不受影响

---

## PC-RT-PAR-005：MED最大边界值验证，MED=4294967295

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-PAR-005 |
| `logic_case_id` | LC-RT-001 |
| `requirement_ids` | SR-RT-001 |
| `feature_tags` | BGP4+, 路由引入, MED边界, static |
| `case_title` | 验证MED=4294967295最大边界值时路由引入成功 |
| `priority` | P2 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能, IBGP邻居已Established |
| `graph_ref` | SP-01 |
| `coverage_goal` | MED边界守卫: 最大值 4294967295 (2^32-1) |
| `trigger_data` | protocol=static, med=4294967295 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-RT-005 |
| `scenario_refs` | S01 |
| `scenario_chain_refs` | S01 boundary_path |
| `action_source_refs` | DR-RT-005 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认IBGP邻居Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established
    trace_refs:
      - DR-RT-005

  - step_id: STEP-002
    step_name: DUT1配置static路由引入，MED=4294967295
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_redist
      args:
        protocol: static
        med: 4294967295
    expected_result: API返回success, 接受MED=4294967295
    trace_refs:
      - DR-RT-005

  - step_id: STEP-003
    step_name: 对端DUT2验证学到路由且MED=4294967295
    target: DUT2
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "[DUT1静态路由前缀]"
    expected_result: BGP路由表中路由MED=4294967295
    trace_refs:
      - DR-RT-005
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认IBGP邻居Established | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 状态=Established |
| 2 | 配置路由引入MED=4294967295 | DUT1 | fw_config_bgp6_redist protocol=static, med=4294967295 | API返回success |
| 3 | 验证对端路由MED=4294967295 | DUT2 | fw_verify_bgp6_route | 路由存在, MED=4294967295 |

### expected_results

1. MED=4294967295 (2^32-1) 为合法最大边界值，API 接受配置
2. 对端正确学到路由，MED=4294967295
3. 路由属性不溢出或截断

---

## PC-RT-PAR-006：MED越界值拒绝，MED=4294967296

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-PAR-006 |
| `logic_case_id` | LC-RT-001 |
| `requirement_ids` | SR-RT-001 |
| `feature_tags` | BGP4+, 路由引入, MED越界, 异常处理 |
| `case_title` | 验证MED=4294967296越界值被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能, IBGP邻居已Established |
| `graph_ref` | SP-01 |
| `coverage_goal` | MED边界守卫: 越界值 4294967296 (2^32) |
| `trigger_data` | protocol=connected, med=4294967296 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-RT-006 |
| `scenario_refs` | S01 |
| `scenario_chain_refs` | S01 abnormal_path |
| `action_source_refs` | DR-RT-006 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认IBGP邻居Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established
    trace_refs:
      - DR-RT-006

  - step_id: STEP-002
    step_name: DUT1尝试配置connected路由引入，MED=4294967296
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_redist
      args:
        protocol: connected
        med: 4294967296
    expected_result: API返回error, MED越界被拒绝 (超出32位无符号整数范围)
    trace_refs:
      - DR-RT-006

  - step_id: STEP-003
    step_name: 验证配置未生效
    target: DUT1
    atomic_op:
      op_id: get-bgpconfig
      args: {}
    expected_result: 路由引入配置中不存在MED=4294967296的条目
    trace_refs:
      - DR-RT-006
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认IBGP邻居Established | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 状态=Established |
| 2 | 尝试配置MED=4294967296 | DUT1 | fw_config_bgp6_redist protocol=connected, med=4294967296 | API返回error |
| 3 | 验证配置未生效 | DUT1 | get-bgpconfig | 不存在越界MED配置 |

### expected_results

1. MED=4294967296 超出 32 位无符号整数范围
2. API 返回错误拒绝配置
3. 设备不会因为非法输入而异常

---

## PC-RT-PAR-007：非法协议拒绝，protocol=rip

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-PAR-007 |
| `logic_case_id` | LC-RT-001 |
| `requirement_ids` | SR-RT-001 |
| `feature_tags` | BGP4+, 路由引入, 协议校验, 异常处理 |
| `case_title` | 验证非法协议rip被拒绝引入BGP4+ |
| `priority` | P2 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能, IBGP邻居已Established |
| `graph_ref` | SP-01 |
| `coverage_goal` | 协议校验: 不支持rip协议引入 |
| `trigger_data` | protocol=rip |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-RT-007 |
| `scenario_refs` | S01 |
| `scenario_chain_refs` | S01 abnormal_path |
| `action_source_refs` | DR-RT-007 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认IBGP邻居Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established
    trace_refs:
      - DR-RT-007

  - step_id: STEP-002
    step_name: DUT1尝试配置rip路由引入
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_redist
      args:
        protocol: rip
    expected_result: API返回error, 不支持rip协议
    trace_refs:
      - DR-RT-007

  - step_id: STEP-003
    step_name: 验证配置未生效
    target: DUT1
    atomic_op:
      op_id: get-bgpconfig
      args: {}
    expected_result: 不存在rip引入配置
    trace_refs:
      - DR-RT-007
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认IBGP邻居Established | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 状态=Established |
| 2 | 尝试配置rip路由引入 | DUT1 | fw_config_bgp6_redist protocol=rip | API返回error |
| 3 | 验证配置未生效 | DUT1 | get-bgpconfig | 不存在rip引入 |

### expected_results

1. BGP4+ 不支持 RIP（IPv4路由协议）引入
2. API 返回错误，拒绝配置
3. 现有配置不受影响

---

## PC-RT-PAR-008：IPv4协议拒绝，protocol=ospf

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-PAR-008 |
| `logic_case_id` | LC-RT-001 |
| `requirement_ids` | SR-RT-001 |
| `feature_tags` | BGP4+, 路由引入, IPv4协议, 异常处理 |
| `case_title` | 验证IPv4协议ospf被拒绝引入BGP4+（仅支持IPv6协议） |
| `priority` | P2 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能, IBGP邻居已Established |
| `graph_ref` | SP-01 |
| `coverage_goal` | 协议校验: IPv4协议ospf拒绝引入BGP4+ |
| `trigger_data` | protocol=ospf |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-RT-008 |
| `scenario_refs` | S01 |
| `scenario_chain_refs` | S01 abnormal_path |
| `action_source_refs` | DR-RT-008 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认IBGP邻居Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established
    trace_refs:
      - DR-RT-008

  - step_id: STEP-002
    step_name: DUT1尝试配置ospf路由引入
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_redist
      args:
        protocol: ospf
    expected_result: API返回error, BGP4+仅支持IPv6协议(connected/static/ospfv3)
    trace_refs:
      - DR-RT-008

  - step_id: STEP-003
    step_name: 验证配置未生效
    target: DUT1
    atomic_op:
      op_id: get-bgpconfig
      args: {}
    expected_result: 不存在ospf引入配置
    trace_refs:
      - DR-RT-008
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认IBGP邻居Established | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 状态=Established |
| 2 | 尝试配置ospf路由引入 | DUT1 | fw_config_bgp6_redist protocol=ospf | API返回error |
| 3 | 验证配置未生效 | DUT1 | get-bgpconfig | 不存在ospf引入 |

### expected_results

1. BGP4+ 仅支持 IPv6 协议引入，ospf（IPv4 OSPFv2）被拒绝
2. API 返回明确错误信息
3. 现有配置不受影响

---

## PC-RT-PAR-009：connected路由过滤，tap/admin类型

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-PAR-009 |
| `logic_case_id` | LC-RT-001 |
| `requirement_ids` | SR-RT-001 |
| `feature_tags` | BGP4+, 路由引入, 路由过滤, connected |
| `case_title` | 验证connected引入时tap/admin类型路由被过滤不引入BGP4+ |
| `priority` | P1 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能, IBGP邻居已Established, DUT1存在tap/admin接口路由 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 路由过滤: tap/admin接口路由不引入BGP4+ |
| `trigger_data` | protocol=connected, 过滤路由类型=tap/admin |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-RT-009 |
| `scenario_refs` | S01 |
| `scenario_chain_refs` | S01 filter_path |
| `action_source_refs` | DR-RT-009 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认IBGP邻居Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established
    trace_refs:
      - DR-RT-009

  - step_id: STEP-002
    step_name: DUT1配置connected路由引入
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_redist
      args:
        protocol: connected
    expected_result: API返回success
    trace_refs:
      - DR-RT-009

  - step_id: STEP-003
    step_name: 验证tap/admin路由未被引入
    target: DUT2
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "[DUT1 tap接口路由前缀]"
    expected_result: BGP路由表中不存在tap/admin接口路由
    trace_refs:
      - DR-RT-009

  - step_id: STEP-004
    step_name: 验证正常物理口路由已引入
    target: DUT2
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "[DUT1正常物理口路由前缀]"
    expected_result: BGP路由表中存在正常物理口路由
    trace_refs:
      - DR-RT-009
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认IBGP邻居Established | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 状态=Established |
| 2 | 配置connected路由引入 | DUT1 | fw_config_bgp6_redist protocol=connected | API返回success |
| 3 | 验证tap/admin路由未被引入 | DUT2 | fw_verify_bgp6_route | 不存在tap/admin路由 |
| 4 | 验证正常物理口路由已引入 | DUT2 | fw_verify_bgp6_route | 存在正常物理口路由 |

### expected_results

1. connected 路由引入过滤 tap/admin 接口路由
2. DUT2 BGP 路由表中不出现 tap/admin 路由
3. 正常物理口路由正常引入不受影响

---

## PC-RT-PAR-010：static路由过滤，ISP/IPSec/NAT66类型

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-PAR-010 |
| `logic_case_id` | LC-RT-001 |
| `requirement_ids` | SR-RT-001 |
| `feature_tags` | BGP4+, 路由引入, 路由过滤, static |
| `case_title` | 验证static引入时ISP/IPSec/NAT66类型路由被过滤不引入BGP4+ |
| `priority` | P1 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能, IBGP邻居已Established, DUT1存在ISP/IPSec/NAT66静态路由 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 路由过滤: ISP/IPSec/NAT66类型静态路由不引入BGP4+ |
| `trigger_data` | protocol=static, 过滤路由类型=ISP/IPSec/NAT66 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-RT-010 |
| `scenario_refs` | S01 |
| `scenario_chain_refs` | S01 filter_path |
| `action_source_refs` | DR-RT-010 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认IBGP邻居Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established
    trace_refs:
      - DR-RT-010

  - step_id: STEP-002
    step_name: DUT1配置static路由引入
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_redist
      args:
        protocol: static
    expected_result: API返回success
    trace_refs:
      - DR-RT-010

  - step_id: STEP-003
    step_name: 验证ISP路由未被引入
    target: DUT2
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "[DUT1 ISP静态路由前缀]"
    expected_result: BGP路由表中不存在ISP路由
    trace_refs:
      - DR-RT-010

  - step_id: STEP-004
    step_name: 验证IPSec/NAT66路由未被引入
    target: DUT2
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "[DUT1 IPSec/NAT66路由前缀]"
    expected_result: BGP路由表中不存在IPSec/NAT66路由
    trace_refs:
      - DR-RT-010

  - step_id: STEP-005
    step_name: 验证正常静态路由已引入
    target: DUT2
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "[DUT1正常静态路由前缀]"
    expected_result: BGP路由表中存在正常静态路由
    trace_refs:
      - DR-RT-010
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认IBGP邻居Established | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 状态=Established |
| 2 | 配置static路由引入 | DUT1 | fw_config_bgp6_redist protocol=static | API返回success |
| 3 | 验证ISP路由未被引入 | DUT2 | fw_verify_bgp6_route | 不存在ISP路由 |
| 4 | 验证IPSec/NAT66路由未被引入 | DUT2 | fw_verify_bgp6_route | 不存在IPSec/NAT66路由 |
| 5 | 验证正常静态路由已引入 | DUT2 | fw_verify_bgp6_route | 存在正常静态路由 |

### expected_results

1. static 路由引入过滤 ISP/IPSec/NAT66 类型路由
2. DUT2 BGP 路由表中不出现 ISP/IPSec/NAT66 路由
3. 正常静态路由正常引入不受影响

---

## PC-RT-PAR-011：OSPFv3邻居断开后路由撤销

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-PAR-011 |
| `logic_case_id` | LC-RT-001 |
| `requirement_ids` | SR-RT-001 |
| `feature_tags` | BGP4+, 路由引入, 路由撤销, ospfv3, 故障恢复 |
| `case_title` | 验证OSPFv3邻居断开后通过BGP4+引入的路由被撤销 |
| `priority` | P1 |
| `preconditions` | DUT1 ospfv3路由已引入BGP4+, DUT2 BGP路由表已有OSPFv3路由 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 路由撤销: OSPFv3源路由消失后BGP路由撤销 |
| `trigger_data` | ospfv3邻居断开 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-RT-011 |
| `scenario_refs` | S01 |
| `scenario_chain_refs` | S01 failure_path |
| `action_source_refs` | DR-RT-011 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认DUT2 BGP路由表已学到OSPFv3引入路由
    target: DUT2
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "[DUT1 OSPFv3路由前缀]"
    expected_result: BGP路由表中存在OSPFv3引入路由
    trace_refs:
      - DR-RT-011

  - step_id: STEP-002
    step_name: 断开DUT1 OSPFv3邻居
    target: DUT1
    atomic_op:
      op_id: set-ospfv3-neighbor-down
      args:
        interface: "[OSPFv3邻居接口]"
    expected_result: OSPFv3邻居断开, 源路由消失
    trace_refs:
      - DR-RT-011

  - step_id: STEP-003
    step_name: 验证DUT2 BGP路由表中对应路由被撤销
    target: DUT2
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "[DUT1 OSPFv3路由前缀]"
    expected_result: BGP路由表中不存在OSPFv3引入路由 (已撤销)
    trace_refs:
      - DR-RT-011

  - step_id: STEP-004
    step_name: 恢复OSPFv3邻居
    target: DUT1
    atomic_op:
      op_id: set-ospfv3-neighbor-up
      args:
        interface: "[OSPFv3邻居接口]"
    expected_result: OSPFv3邻居恢复, 源路由恢复
    trace_refs:
      - DR-RT-011

  - step_id: STEP-005
    step_name: 验证DUT2 BGP路由重新学到OSPFv3路由
    target: DUT2
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "[DUT1 OSPFv3路由前缀]"
    expected_result: BGP路由表中重新出现OSPFv3引入路由
    trace_refs:
      - DR-RT-011
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认DUT2已有OSPFv3引入路由 | DUT2 | fw_verify_bgp6_route | 路由存在 |
| 2 | 断开DUT1 OSPFv3邻居 | DUT1 | set-ospfv3-neighbor-down | OSPFv3邻居断开 |
| 3 | 验证DUT2路由被撤销 | DUT2 | fw_verify_bgp6_route | 路由不存在 |
| 4 | 恢复OSPFv3邻居 | DUT1 | set-ospfv3-neighbor-up | OSPFv3邻居恢复 |
| 5 | 验证DUT2路由重新学到 | DUT2 | fw_verify_bgp6_route | 路由重新存在 |

### expected_results

1. OSPFv3 邻居断开后，对应引入路由从 BGP 路由表撤销
2. OSPFv3 邻居恢复后，路由重新引入 BGP 路由表
3. 路由撤销/恢复及时，无残留路由
