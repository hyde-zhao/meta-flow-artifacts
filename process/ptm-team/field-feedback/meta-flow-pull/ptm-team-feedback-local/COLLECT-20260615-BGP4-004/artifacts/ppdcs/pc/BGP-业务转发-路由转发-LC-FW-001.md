# BGP4+ ECMP与负载均衡 — 物理用例

> 生成时间: 2026-06-12 | 设计Skill: parameter-design
> 逻辑用例: LC-FW-001 | 目录: BGP-业务转发-路由转发

---

## PC-FW-PAR-001：双链路IBGP ECMP基线

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW-PAR-001 |
| `logic_case_id` | LC-FW-001 |
| `requirement_ids` | SR-FW-001 |
| `feature_tags` | BGP4+, ECMP, 负载均衡, IBGP |
| `case_title` | 验证双链路IBGP下ECMP启用后两条路径均有流量转发 |
| `priority` | P1 |
| `preconditions` | DUT1与DUT2通过两条IBGP链路连接, ECMP已启用 |
| `graph_ref` | SP-01 |
| `coverage_goal` | ECMP基线: 双链路IBGP均转发 |
| `trigger_data` | ECMP=enable, 路径数=2 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-FW-001 |
| `scenario_refs` | S03 |
| `scenario_chain_refs` | S03 normal_path |
| `action_source_refs` | DR-FW-001 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认DUT1-DUT2两条IBGP链路Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args: {}
    expected_result: 两个IBGP邻居均Established
    trace_refs:
      - DR-FW-001

  - step_id: STEP-002
    step_name: 启用ECMP
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        ecmp: enable
        max_paths: 2
    expected_result: ECMP启用成功
    trace_refs:
      - DR-FW-001

  - step_id: STEP-003
    step_name: DUT2通告相同路由通过两条链路
    target: DUT2
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        prefix: "2001:db8:f01::/64"
    expected_result: 路由通过两条链路通告
    trace_refs:
      - DR-FW-001

  - step_id: STEP-004
    step_name: 验证DUT1路由表存在两条等价路径
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:f01::/64"
    expected_result: 路由表中存在2条等价下一跳
    trace_refs:
      - DR-FW-001

  - step_id: STEP-005
    step_name: TG1发起流量验证ECMP负载分担
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:f01::1"
        duration: 30
    expected_result: 两条链路均有流量通过, 负载分担
    trace_refs:
      - DR-FW-001
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认两条IBGP链路Established | DUT1 | get-bgp6neighbor | 2个邻居Established |
| 2 | 启用ECMP max_paths=2 | DUT1 | fw_config_bgp_global ecmp=enable | 配置成功 |
| 3 | DUT2通告路由 | DUT2 | fw_config_bgp_global | 配置成功 |
| 4 | 验证2条等价路径 | DUT1 | fw_verify_bgp6_route | 2条等价下一跳 |
| 5 | TG1发起流量验证 | TG1 | tg_start_traffic_stream | 双链路均有流量 |

### expected_results

1. ECMP 启用后路由表存在两条等价路径
2. 流量在两条链路上均匀分担
3. 无丢包或路径异常

---

## PC-FW-PAR-002：ECMP未启用对照

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW-PAR-002 |
| `logic_case_id` | LC-FW-001 |
| `requirement_ids` | SR-FW-001 |
| `feature_tags` | BGP4+, ECMP, 对照验证 |
| `case_title` | 验证ECMP未启用时双链路仅一条路径转发（对照验证） |
| `priority` | P2 |
| `preconditions` | DUT1与DUT2通过两条IBGP链路连接, ECMP未启用 |
| `graph_ref` | SP-01 |
| `coverage_goal` | ECMP对照: 未启用时单路径 |
| `trigger_data` | ECMP=disable, 路径数=2 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-FW-002 |
| `scenario_refs` | S03 |
| `scenario_chain_refs` | S03 control_path |
| `action_source_refs` | DR-FW-002 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认两条IBGP链路Established, ECMP未启用
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args: {}
    expected_result: 2个邻居Established, ECMP=disable
    trace_refs:
      - DR-FW-002

  - step_id: STEP-002
    step_name: DUT2通告路由
    target: DUT2
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        prefix: "2001:db8:f02::/64"
    expected_result: 路由通告成功
    trace_refs:
      - DR-FW-002

  - step_id: STEP-003
    step_name: 验证DUT1仅有1条优选路径
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:f02::/64"
    expected_result: 路由表中仅1条优选下一跳
    trace_refs:
      - DR-FW-002

  - step_id: STEP-004
    step_name: TG1发起流量验证仅单链路有流量
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:f02::1"
        duration: 30
    expected_result: 仅一条链路有流量通过
    trace_refs:
      - DR-FW-002
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认双链路ECMP未启用 | DUT1 | get-bgp6neighbor | ECMP=disable |
| 2 | DUT2通告路由 | DUT2 | fw_config_bgp_global | 配置成功 |
| 3 | 验证仅1条优选路径 | DUT1 | fw_verify_bgp6_route | 1条下一跳 |
| 4 | TG1流量验证 | TG1 | tg_start_traffic_stream | 仅单链路有流量 |

### expected_results

1. ECMP 未启用时路由表仅一条优选路径
2. 流量仅在优选链路上转发
3. 与 ECMP 启用形成对照

---

## PC-FW-PAR-003：ECMP流量验证

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW-PAR-003 |
| `logic_case_id` | LC-FW-001 |
| `requirement_ids` | SR-FW-001 |
| `feature_tags` | BGP4+, ECMP, 流量验证, 负载均衡 |
| `case_title` | 验证ECMP流量在两链路间按5元组哈希均衡分发 |
| `priority` | P1 |
| `preconditions` | DUT1-DUT2双链路IBGP ECMP已启用, 路由等价 |
| `graph_ref` | SP-01 |
| `coverage_goal` | ECMP流量验证: 哈希均衡分发 |
| `trigger_data` | ECMP=enable, 负载模式=五元组哈希 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-FW-003 |
| `scenario_refs` | S03 |
| `scenario_chain_refs` | S03 normal_path |
| `action_source_refs` | DR-FW-003 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认ECMP已启用, 两条IBGP链路路径等价
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:f03::/64"
    expected_result: 2条等价路径存在
    trace_refs:
      - DR-FW-003

  - step_id: STEP-002
    step_name: TG1发起多流流量 (不同源IP/端口)
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:f03::1"
        src_range: "2001:db8:src::1-2001:db8:src::100"
        sport_range: "10000-20000"
        duration: 60
    expected_result: 多流流量发起成功
    trace_refs:
      - DR-FW-003

  - step_id: STEP-003
    step_name: 验证两条链路流量分布均衡
    target: DUT1
    atomic_op:
      op_id: get-interface-stats
      args:
        interfaces: ["GE0_1", "GE0_2"]
    expected_result: 两条链路流量分布偏差<20%
    trace_refs:
      - DR-FW-003
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认ECMP等价路径 | DUT1 | fw_verify_bgp6_route | 2条等价路径 |
| 2 | TG1发起多流流量 | TG1 | tg_start_traffic_stream | 多流发起成功 |
| 3 | 验证链路流量均衡 | DUT1 | get-interface-stats | 偏差<20% |

### expected_results

1. 多流流量在两链路间按五元组哈希分发
2. 链路流量分布偏差 < 20%
3. 无乱序或丢包

---

## PC-FW-PAR-004：带宽比例负载 100M:200M

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW-PAR-004 |
| `logic_case_id` | LC-FW-001 |
| `requirement_ids` | SR-FW-001 |
| `feature_tags` | BGP4+, ECMP, 带宽比例, 负载均衡 |
| `case_title` | 验证ECMP按带宽比例100M:200M进行非等价负载分担 |
| `priority` | P1 |
| `preconditions` | DUT1-DUT2双链路IBGP, ECMP启用带宽比例模式, 链路A=100M, 链路B=200M |
| `graph_ref` | SP-01 |
| `coverage_goal` | 负载模式: 带宽比例 1:2 |
| `trigger_data` | 负载模式=带宽, 比例=100M:200M |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-FW-004 |
| `scenario_refs` | S03 |
| `scenario_chain_refs` | S03 bandwidth_path |
| `action_source_refs` | DR-FW-004 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 配置链路A带宽=100M, 链路B带宽=200M
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        interface: "GE0_1"
        bandwidth: "100M"
    expected_result: 链路A带宽=100M
    trace_refs:
      - DR-FW-004

  - step_id: STEP-002
    step_name: 配置链路B带宽=200M
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        interface: "GE0_2"
        bandwidth: "200M"
    expected_result: 链路B带宽=200M
    trace_refs:
      - DR-FW-004

  - step_id: STEP-003
    step_name: 启用ECMP带宽比例模式
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        ecmp: enable
        load_balance_mode: bandwidth
    expected_result: 带宽比例模式启用
    trace_refs:
      - DR-FW-004

  - step_id: STEP-004
    step_name: TG1发起流量验证比例
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:f04::1"
        rate: "250M"
        duration: 60
    expected_result: 链路A流量约83M, 链路B流量约167M (比例约1:2)
    trace_refs:
      - DR-FW-004
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 配置链路A带宽100M | DUT1 | fw_config_interface bandwidth=100M | 配置成功 |
| 2 | 配置链路B带宽200M | DUT1 | fw_config_interface bandwidth=200M | 配置成功 |
| 3 | 启用带宽比例ECMP | DUT1 | fw_config_bgp_global load_balance_mode=bandwidth | 配置成功 |
| 4 | TG1流量验证比例 | TG1 | tg_start_traffic_stream rate=250M | 比例≈1:2 |

### expected_results

1. ECMP 按带宽比例分配流量: 100M : 200M ≈ 1 : 2
2. 链路A 流量约 83M, 链路B 流量约 167M
3. 比例偏差在允许范围内

---

## PC-FW-PAR-005：权重比例负载 3:1

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW-PAR-005 |
| `logic_case_id` | LC-FW-001 |
| `requirement_ids` | SR-FW-001 |
| `feature_tags` | BGP4+, ECMP, 权重比例, 负载均衡 |
| `case_title` | 验证ECMP按权重比例3:1进行非等价负载分担 |
| `priority` | P1 |
| `preconditions` | DUT1-DUT2双链路IBGP, ECMP启用权重比例模式, 链路A权重=3, 链路B权重=1 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 负载模式: 权重比例 3:1 |
| `trigger_data` | 负载模式=权重, 比例=3:1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-FW-005 |
| `scenario_refs` | S03 |
| `scenario_chain_refs` | S03 weight_path |
| `action_source_refs` | DR-FW-005 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 配置链路A权重=3, 链路B权重=1
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        interface: "GE0_1"
        weight: 3
    expected_result: 链路A权重=3
    trace_refs:
      - DR-FW-005

  - step_id: STEP-002
    step_name: 配置链路B权重=1
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        interface: "GE0_2"
        weight: 1
    expected_result: 链路B权重=1
    trace_refs:
      - DR-FW-005

  - step_id: STEP-003
    step_name: 启用ECMP权重比例模式
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        ecmp: enable
        load_balance_mode: weight
    expected_result: 权重比例模式启用
    trace_refs:
      - DR-FW-005

  - step_id: STEP-004
    step_name: TG1发起流量验证比例
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:f05::1"
        rate: "200M"
        duration: 60
    expected_result: 链路A流量约150M, 链路B流量约50M (比例约3:1)
    trace_refs:
      - DR-FW-005
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 配置链路A权重3 | DUT1 | fw_config_interface weight=3 | 配置成功 |
| 2 | 配置链路B权重1 | DUT1 | fw_config_interface weight=1 | 配置成功 |
| 3 | 启用权重比例ECMP | DUT1 | fw_config_bgp_global load_balance_mode=weight | 配置成功 |
| 4 | TG1流量验证比例 | TG1 | tg_start_traffic_stream rate=200M | 比例≈3:1 |

### expected_results

1. ECMP 按权重比例分配流量: 3 : 1
2. 链路A 流量约 150M, 链路B 流量约 50M
3. 比例偏差在允许范围内

---

## PC-FW-PAR-006：16路径ECMP

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW-PAR-006 |
| `logic_case_id` | LC-FW-001 |
| `requirement_ids` | SR-FW-001 |
| `feature_tags` | BGP4+, ECMP, 最大路径, 边界 |
| `case_title` | 验证ECMP支持最大16条等价路径负载分担 |
| `priority` | P2 |
| `preconditions` | DUT1与DUT2通过16条IBGP链路连接, ECMP max_paths=16 |
| `graph_ref` | SP-01 |
| `coverage_goal` | ECMP边界: 最大16路径 |
| `trigger_data` | ECMP=enable, 路径数=16 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-FW-006 |
| `scenario_refs` | S03 |
| `scenario_chain_refs` | S03 boundary_path |
| `action_source_refs` | DR-FW-006 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认16条IBGP链路均Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args: {}
    expected_result: 16个邻居均Established
    trace_refs:
      - DR-FW-006

  - step_id: STEP-002
    step_name: 启用ECMP max_paths=16
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        ecmp: enable
        max_paths: 16
    expected_result: ECMP配置成功
    trace_refs:
      - DR-FW-006

  - step_id: STEP-003
    step_name: 验证DUT1路由表存在16条等价路径
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:f06::/64"
    expected_result: 路由表中存在16条等价下一跳
    trace_refs:
      - DR-FW-006

  - step_id: STEP-004
    step_name: TG1发起流量验证16路径负载分担
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:f06::1"
        duration: 60
    expected_result: 16条链路均有流量通过
    trace_refs:
      - DR-FW-006
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认16条IBGP Established | DUT1 | get-bgp6neighbor | 16个邻居Established |
| 2 | 启用ECMP max_paths=16 | DUT1 | fw_config_bgp_global ecmp=enable, max_paths=16 | 配置成功 |
| 3 | 验证16条等价路径 | DUT1 | fw_verify_bgp6_route | 16条下一跳 |
| 4 | TG1流量验证 | TG1 | tg_start_traffic_stream | 16链路均有流量 |

### expected_results

1. ECMP 支持最大 16 条等价路径
2. 路由表正确维护 16 条等价下一跳
3. 流量在所有链路上分担

---

## PC-FW-PAR-007：单链路故障切换

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW-PAR-007 |
| `logic_case_id` | LC-FW-001 |
| `requirement_ids` | SR-FW-001 |
| `feature_tags` | BGP4+, ECMP, 故障切换, 高可用 |
| `case_title` | 验证ECMP其中一条链路故障后流量自动切换到剩余链路 |
| `priority` | P1 |
| `preconditions` | DUT1-DUT2双链路IBGP ECMP正常运行 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 故障切换: 单链路故障 → 切至剩余链路 |
| `trigger_data` | 故障切换=单链路down |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-FW-007 |
| `scenario_refs` | S03 |
| `scenario_chain_refs` | S03 failure_path |
| `action_source_refs` | DR-FW-007 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认ECMP 2路径正常流量
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:f07::/64"
    expected_result: 2条等价路径, 流量正常
    trace_refs:
      - DR-FW-007

  - step_id: STEP-002
    step_name: TG1发起持续流量
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:f07::1"
        duration: 120
    expected_result: 流量持续发送
    trace_refs:
      - DR-FW-007

  - step_id: STEP-003
    step_name: 链路A故障 (admin down GE0_1)
    target: DUT1
    atomic_op:
      op_id: set-interface-down
      args:
        interface: "GE0_1"
    expected_result: 接口状态=down, 邻居断开
    trace_refs:
      - DR-FW-007

  - step_id: STEP-004
    step_name: 验证路由收敛为1条路径
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:f07::/64"
    expected_result: 仅剩1条等价路径, 无GE0_1
    trace_refs:
      - DR-FW-007

  - step_id: STEP-005
    step_name: 验证TG1流量未中断
    target: TG1
    atomic_op:
      op_id: get-traffic-stats
      args: {}
    expected_result: 流量切换到剩余链路, 丢包数在可接受范围内
    trace_refs:
      - DR-FW-007
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认ECMP双路径正常 | DUT1 | fw_verify_bgp6_route | 2条等价路径 |
| 2 | TG1发起持续流量 | TG1 | tg_start_traffic_stream | 流量持续 |
| 3 | 链路A故障 down | DUT1 | set-interface-down GE0_1 | 接口down |
| 4 | 验证路由收敛1条路径 | DUT1 | fw_verify_bgp6_route | 1条路径 |
| 5 | 验证流量切换 | TG1 | get-traffic-stats | 流量切换, 丢包少 |

### expected_results

1. 链路故障后路由快速收敛
2. 流量自动切换到剩余链路
3. 切换过程中丢包数在可接受范围

---

## PC-FW-PAR-008：链路恢复

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW-PAR-008 |
| `logic_case_id` | LC-FW-001 |
| `requirement_ids` | SR-FW-001 |
| `feature_tags` | BGP4+, ECMP, 链路恢复, 高可用 |
| `case_title` | 验证ECMP故障链路恢复后流量自动重新负载分担 |
| `priority` | P1 |
| `preconditions` | DUT1-DUT2 ECMP运行中, 一条链路已故障 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 故障恢复: 链路恢复 → 重新加入ECMP |
| `trigger_data` | 链路恢复=admin up |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-FW-008 |
| `scenario_refs` | S03 |
| `scenario_chain_refs` | S03 recovery_path |
| `action_source_refs` | DR-FW-008 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认当前仅1条路径 (故障态)
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:f08::/64"
    expected_result: 仅1条等价路径
    trace_refs:
      - DR-FW-008

  - step_id: STEP-002
    step_name: TG1发起持续流量
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:f08::1"
        duration: 120
    expected_result: 流量持续发送
    trace_refs:
      - DR-FW-008

  - step_id: STEP-003
    step_name: 恢复链路A (admin up GE0_1)
    target: DUT1
    atomic_op:
      op_id: set-interface-up
      args:
        interface: "GE0_1"
    expected_result: 接口状态=up, 邻居重建Established
    trace_refs:
      - DR-FW-008

  - step_id: STEP-004
    step_name: 验证路由恢复为2条等价路径
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:f08::/64"
    expected_result: 恢复2条等价路径
    trace_refs:
      - DR-FW-008

  - step_id: STEP-005
    step_name: 验证链路A重新参与负载分担
    target: DUT1
    atomic_op:
      op_id: get-interface-stats
      args:
        interfaces: ["GE0_1", "GE0_2"]
    expected_result: 两条链路均有流量
    trace_refs:
      - DR-FW-008
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认当前单路径故障态 | DUT1 | fw_verify_bgp6_route | 1条路径 |
| 2 | TG1发起持续流量 | TG1 | tg_start_traffic_stream | 流量持续 |
| 3 | 恢复链路A | DUT1 | set-interface-up GE0_1 | 接口up, 邻居恢复 |
| 4 | 验证恢复2条等价路径 | DUT1 | fw_verify_bgp6_route | 2条路径 |
| 5 | 验证链路A重新分担 | DUT1 | get-interface-stats | 双链路均有流量 |

### expected_results

1. 链路恢复后 BGP 邻居自动重建
2. ECMP 自动将恢复链路加入等价路径
3. 流量重新在两条链路间分担
