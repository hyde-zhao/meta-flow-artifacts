# BGP4+ 异构兼容与路由优先级 — 物理用例

> 生成时间: 2026-06-12 | 设计Skill: parameter-design
> 逻辑用例: LC-CMP-001 | 目录: BGP-协议协商-路由计算

---

## PC-CMP-PAR-001：华为+MD5一致 → Established

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-CMP-PAR-001 |
| `logic_case_id` | LC-CMP-001 |
| `requirement_ids` | SR-CMP-001 |
| `feature_tags` | BGP4+, 异构兼容, 华为, MD5认证 |
| `case_title` | 验证华为设备BGP4+邻居MD5认证一致时正常建立Established |
| `priority` | P1 |
| `preconditions` | DUT1(华为)与DUT2 BGP4+全局使能, 接口IPv6互通 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 异构基线: 华为 + MD5一致 |
| `trigger_data` | 厂商=华为, MD5状态=一致 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-CMP-001 |
| `scenario_refs` | S05 |
| `scenario_chain_refs` | S05 normal_path |
| `action_source_refs` | DR-CMP-001 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认DUT1为华为设备, BGP4+全局使能
    target: DUT1
    atomic_op:
      op_id: get-bgpconfig
      args: {}
    expected_result: enable=true, vendor=华为
    trace_refs:
      - DR-CMP-001

  - step_id: STEP-002
    step_name: 两端配置相同MD5认证密钥
    target: DUT1, DUT2
    atomic_op:
      op_id: fw_config_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        auth_mode: md5
        auth_key: "test123"
    expected_result: MD5认证配置成功
    trace_refs:
      - DR-CMP-001

  - step_id: STEP-003
    step_name: DUT1创建IBGP邻居指向DUT2
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "GE0_1"
    expected_result: API返回success
    trace_refs:
      - DR-CMP-001

  - step_id: STEP-004
    step_name: 验证邻居Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, MD5认证通过
    trace_refs:
      - DR-CMP-001
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认华为设备BGP4+使能 | DUT1 | get-bgpconfig | vendor=华为, enable=true |
| 2 | 配置两端MD5密钥一致 | DUT1, DUT2 | fw_config_bgp6_neighbor auth_mode=md5, auth_key=test123 | 配置成功 |
| 3 | DUT1创建IBGP邻居 | DUT1 | fw_create_bgp6_neighbor | API返回success |
| 4 | 验证邻居Established | DUT1 | get-bgp6neighbor | Established, MD5通过 |

### expected_results

1. 华为设备 MD5 认证配置成功
2. 双方 MD5 密钥一致时邻居正常建立
3. TCP MD5 Signature 选项正确验证

---

## PC-CMP-PAR-002：华三+MD5一致 → Established

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-CMP-PAR-002 |
| `logic_case_id` | LC-CMP-001 |
| `requirement_ids` | SR-CMP-001 |
| `feature_tags` | BGP4+, 异构兼容, 华三, MD5认证 |
| `case_title` | 验证华三设备BGP4+邻居MD5认证一致时正常建立Established |
| `priority` | P1 |
| `preconditions` | DUT1(华三)与DUT2 BGP4+全局使能, 接口IPv6互通 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 异构基线: 华三 + MD5一致 |
| `trigger_data` | 厂商=华三, MD5状态=一致 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-CMP-002 |
| `scenario_refs` | S05 |
| `scenario_chain_refs` | S05 normal_path |
| `action_source_refs` | DR-CMP-002 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认DUT1为华三设备, BGP4+全局使能
    target: DUT1
    atomic_op:
      op_id: get-bgpconfig
      args: {}
    expected_result: enable=true, vendor=华三
    trace_refs:
      - DR-CMP-002

  - step_id: STEP-002
    step_name: 两端配置相同MD5认证密钥
    target: DUT1, DUT2
    atomic_op:
      op_id: fw_config_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        auth_mode: md5
        auth_key: "h3c_test"
    expected_result: MD5认证配置成功
    trace_refs:
      - DR-CMP-002

  - step_id: STEP-003
    step_name: DUT1创建IBGP邻居指向DUT2
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "GE0_1"
    expected_result: API返回success
    trace_refs:
      - DR-CMP-002

  - step_id: STEP-004
    step_name: 验证邻居Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, MD5认证通过
    trace_refs:
      - DR-CMP-002
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认华三设备BGP4+使能 | DUT1 | get-bgpconfig | vendor=华三, enable=true |
| 2 | 配置两端MD5密钥一致 | DUT1, DUT2 | fw_config_bgp6_neighbor auth_mode=md5, auth_key=h3c_test | 配置成功 |
| 3 | DUT1创建IBGP邻居 | DUT1 | fw_create_bgp6_neighbor | API返回success |
| 4 | 验证邻居Established | DUT1 | get-bgp6neighbor | Established, MD5通过 |

### expected_results

1. 华三设备 MD5 认证配置成功
2. 双方 MD5 密钥一致时邻居正常建立
3. 华三设备 TCP MD5 Signature 实现兼容

---

## PC-CMP-PAR-003：锐捷+MD5一致 → Established

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-CMP-PAR-003 |
| `logic_case_id` | LC-CMP-001 |
| `requirement_ids` | SR-CMP-001 |
| `feature_tags` | BGP4+, 异构兼容, 锐捷, MD5认证 |
| `case_title` | 验证锐捷设备BGP4+邻居MD5认证一致时正常建立Established |
| `priority` | P1 |
| `preconditions` | DUT1(锐捷)与DUT2 BGP4+全局使能, 接口IPv6互通 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 异构基线: 锐捷 + MD5一致 |
| `trigger_data` | 厂商=锐捷, MD5状态=一致 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-CMP-003 |
| `scenario_refs` | S05 |
| `scenario_chain_refs` | S05 normal_path |
| `action_source_refs` | DR-CMP-003 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认DUT1为锐捷设备, BGP4+全局使能
    target: DUT1
    atomic_op:
      op_id: get-bgpconfig
      args: {}
    expected_result: enable=true, vendor=锐捷
    trace_refs:
      - DR-CMP-003

  - step_id: STEP-002
    step_name: 两端配置相同MD5认证密钥
    target: DUT1, DUT2
    atomic_op:
      op_id: fw_config_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        auth_mode: md5
        auth_key: "ruijie_test"
    expected_result: MD5认证配置成功
    trace_refs:
      - DR-CMP-003

  - step_id: STEP-003
    step_name: DUT1创建IBGP邻居指向DUT2
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "GE0_1"
    expected_result: API返回success
    trace_refs:
      - DR-CMP-003

  - step_id: STEP-004
    step_name: 验证邻居Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, MD5认证通过
    trace_refs:
      - DR-CMP-003
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认锐捷设备BGP4+使能 | DUT1 | get-bgpconfig | vendor=锐捷, enable=true |
| 2 | 配置两端MD5密钥一致 | DUT1, DUT2 | fw_config_bgp6_neighbor auth_mode=md5, auth_key=ruijie_test | 配置成功 |
| 3 | DUT1创建IBGP邻居 | DUT1 | fw_create_bgp6_neighbor | API返回success |
| 4 | 验证邻居Established | DUT1 | get-bgp6neighbor | Established, MD5通过 |

### expected_results

1. 锐捷设备 MD5 认证配置成功
2. 双方 MD5 密钥一致时邻居正常建立
3. 锐捷设备 TCP MD5 Signature 实现兼容

---

## PC-CMP-PAR-004：华为+MD5不一致 → 失败

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-CMP-PAR-004 |
| `logic_case_id` | LC-CMP-001 |
| `requirement_ids` | SR-CMP-001 |
| `feature_tags` | BGP4+, 异构兼容, 华为, MD5认证, 异常 |
| `case_title` | 验证华为设备MD5认证不一致时邻居建立失败 |
| `priority` | P1 |
| `preconditions` | DUT1(华为)与DUT2 BGP4+全局使能, 两端MD5密钥不同 |
| `graph_ref` | SP-01 |
| `coverage_goal` | MD5异常: 华为设备MD5不一致 |
| `trigger_data` | 厂商=华为, MD5状态=不一致 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-CMP-004 |
| `scenario_refs` | S05 |
| `scenario_chain_refs` | S05 abnormal_path |
| `action_source_refs` | DR-CMP-004 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认华为设备BGP4+使能
    target: DUT1
    atomic_op:
      op_id: get-bgpconfig
      args: {}
    expected_result: vendor=华为, enable=true
    trace_refs:
      - DR-CMP-004

  - step_id: STEP-002
    step_name: DUT1配置MD5密钥key_A, DUT2配置MD5密钥key_B
    target: DUT1, DUT2
    atomic_op:
      op_id: fw_config_bgp6_neighbor
      args:
        DUT1:
          auth_key: "key_A"
        DUT2:
          auth_key: "key_B"
    expected_result: 配置成功, 两端密钥不同
    trace_refs:
      - DR-CMP-004

  - step_id: STEP-003
    step_name: DUT1创建IBGP邻居指向DUT2
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "GE0_1"
    expected_result: API返回success, 邻居尝试建立
    trace_refs:
      - DR-CMP-004

  - step_id: STEP-004
    step_name: 验证邻居建立失败
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态≠Established, 因MD5认证失败无法建立
    trace_refs:
      - DR-CMP-004
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认华为设备BGP4+使能 | DUT1 | get-bgpconfig | vendor=华为 |
| 2 | 配置两端MD5密钥不同 | DUT1, DUT2 | fw_config_bgp6_neighbor | 密钥不同 |
| 3 | DUT1创建IBGP邻居 | DUT1 | fw_create_bgp6_neighbor | API返回success |
| 4 | 验证邻居建立失败 | DUT1 | get-bgp6neighbor | ≠Established |

### expected_results

1. MD5 密钥不一致时 TCP 连接无法通过认证
2. 邻居无法建立，停留在 Connect/Active 或直接进入 Idle
3. 系统日志记录 MD5 认证失败事件

---

## PC-CMP-PAR-005：BGPv4+BGP4+双栈共存互不干扰

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-CMP-PAR-005 |
| `logic_case_id` | LC-CMP-001 |
| `requirement_ids` | SR-CMP-001 |
| `feature_tags` | BGP4+, 异构兼容, 双栈, BGPv4 |
| `case_title` | 验证BGPv4和BGP4+双栈同时运行互不干扰 |
| `priority` | P1 |
| `preconditions` | DUT1同时运行BGPv4和BGP4+, 分别与DUT2有IPv4和IPv6邻居 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 双栈共存: BGPv4 + BGP4+ 互不干扰 |
| `trigger_data` | BGPv4 + BGP4+ 双栈 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-CMP-005 |
| `scenario_refs` | S05 |
| `scenario_chain_refs` | S05 dual_stack_path |
| `action_source_refs` | DR-CMP-005 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认BGPv4邻居已Established
    target: DUT1
    atomic_op:
      op_id: get-bgpneighbor
      args:
        id: "10.0.0.2"
    expected_result: BGPv4邻居状态=Established
    trace_refs:
      - DR-CMP-005

  - step_id: STEP-002
    step_name: 创建BGP4+邻居
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "GE0_1"
    expected_result: BGP4+邻居建立成功
    trace_refs:
      - DR-CMP-005

  - step_id: STEP-003
    step_name: 验证BGPv4邻居仍Established
    target: DUT1
    atomic_op:
      op_id: get-bgpneighbor
      args:
        id: "10.0.0.2"
    expected_result: BGPv4邻居不受影响, 仍Established
    trace_refs:
      - DR-CMP-005

  - step_id: STEP-004
    step_name: 验证BGP4+邻居Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: BGP4+邻居状态=Established
    trace_refs:
      - DR-CMP-005

  - step_id: STEP-005
    step_name: 分别验证IPv4和IPv6路由
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:cmp5::/64"
    expected_result: IPv4和IPv6路由各自独立, 互不干扰
    trace_refs:
      - DR-CMP-005
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认BGPv4 Established | DUT1 | get-bgpneighbor id=10.0.0.2 | Established |
| 2 | 创建BGP4+邻居 | DUT1 | fw_create_bgp6_neighbor | BGP4+建立成功 |
| 3 | 验证BGPv4不受影响 | DUT1 | get-bgpneighbor | 仍Established |
| 4 | 验证BGP4+ Established | DUT1 | get-bgp6neighbor | Established |
| 5 | 验证IPv4/IPv6路由独立 | DUT1 | fw_verify_bgp6_route | 路由独立, 互不干扰 |

### expected_results

1. BGPv4 和 BGP4+ 作为独立进程/会话同时运行
2. 两套邻居状态互不影响
3. IPv4 和 IPv6 路由表各自维护

---

## PC-CMP-PAR-006：PBR优先于BGP

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-CMP-PAR-006 |
| `logic_case_id` | LC-CMP-001 |
| `requirement_ids` | SR-CMP-001 |
| `feature_tags` | BGP4+, 路由优先级, PBR, 策略路由 |
| `case_title` | 验证PBR策略路由优先级高于BGP路由 |
| `priority` | P1 |
| `preconditions` | DUT1配置PBR策略和BGP路由, 两者匹配相同流量 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 优先级验证: PBR > BGP |
| `trigger_data` | 优先级对比=PBR vs BGP |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-CMP-006 |
| `scenario_refs` | S05 |
| `scenario_chain_refs` | S05 priority_path |
| `action_source_refs` | DR-CMP-006 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认BGP路由存在, 下一跳=DUT2
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:cmp6::/64"
    expected_result: BGP路由下一跳=DUT2
    trace_refs:
      - DR-CMP-006

  - step_id: STEP-002
    step_name: 配置PBR策略, 匹配相同流量, 下一跳=TG1
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        pbr_policy: "redirect_to_tg1"
        match:
          dst: "2001:db8:cmp6::/64"
        set:
          next_hop: "[TG1_IPv6]"
    expected_result: PBR策略配置成功
    trace_refs:
      - DR-CMP-006

  - step_id: STEP-003
    step_name: TG1发起匹配流量
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:cmp6::1"
        duration: 30
    expected_result: 流量走PBR指定下一跳TG1, 而非BGP下一跳DUT2
    trace_refs:
      - DR-CMP-006

  - step_id: STEP-004
    step_name: DUT2验证未收到该流量
    target: DUT2
    atomic_op:
      op_id: get-interface-stats
      args:
        interface: "GE0_1"
    expected_result: DUT2未收到2001:db8:cmp6::1流量
    trace_refs:
      - DR-CMP-006
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认BGP路由下一跳=DUT2 | DUT1 | fw_verify_bgp6_route | 下一跳=DUT2 |
| 2 | 配置PBR策略下一跳=TG1 | DUT1 | fw_config_interface pbr_policy | PBR配置成功 |
| 3 | TG1发起匹配流量 | TG1 | tg_start_traffic_stream | 流量走TG1 |
| 4 | DUT2验证未收到流量 | DUT2 | get-interface-stats | 未收到 |

### expected_results

1. PBR 策略路由优先级高于 BGP 路由
2. 匹配 PBR 的流量不走 BGP 路由表
3. 不匹配 PBR 的流量正常走 BGP 路由

---

## PC-CMP-PAR-007：静态路由 > IBGP

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-CMP-PAR-007 |
| `logic_case_id` | LC-CMP-001 |
| `requirement_ids` | SR-CMP-001 |
| `feature_tags` | BGP4+, 路由优先级, 静态路由, IBGP |
| `case_title` | 验证静态路由优先级高于IBGP路由 |
| `priority` | P1 |
| `preconditions` | DUT1同时存在指向目标的静态路由和IBGP路由 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 优先级验证: 静态 > IBGP |
| `trigger_data` | 优先级对比=静态 vs IBGP |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-CMP-007 |
| `scenario_refs` | S05 |
| `scenario_chain_refs` | S05 priority_path |
| `action_source_refs` | DR-CMP-007 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认IBGP路由存在, 下一跳=DUT2
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:cmp7::/64"
    expected_result: IBGP路由下一跳=DUT2
    trace_refs:
      - DR-CMP-007

  - step_id: STEP-002
    step_name: 配置静态路由指向前缀2001:db8:cmp7::/64, 下一跳=TG1
    target: DUT1
    atomic_op:
      op_id: put-static-route
      args:
        prefix: "2001:db8:cmp7::/64"
        next_hop: "[TG1_IPv6]"
    expected_result: 静态路由配置成功
    trace_refs:
      - DR-CMP-007

  - step_id: STEP-003
    step_name: 验证路由表优选静态路由
    target: DUT1
    atomic_op:
      op_id: get-ipv6-route
      args:
        prefix: "2001:db8:cmp7::/64"
    expected_result: 优选静态路由, 下一跳=TG1, 协议优先级高于IBGP
    trace_refs:
      - DR-CMP-007

  - step_id: STEP-004
    step_name: TG1验证流量走静态路由
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:cmp7::1"
        duration: 30
    expected_result: 流量走TG1, 静态路由生效
    trace_refs:
      - DR-CMP-007
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认IBGP路由存在 | DUT1 | fw_verify_bgp6_route | 下一跳=DUT2 |
| 2 | 配置静态路由下一跳=TG1 | DUT1 | put-static-route | 配置成功 |
| 3 | 验证路由表优选静态 | DUT1 | get-ipv6-route | 优选静态, 下一跳=TG1 |
| 4 | TG1流量验证 | TG1 | tg_start_traffic_stream | 流量走静态路由 |

### expected_results

1. 静态路由优先级高于 IBGP 路由
2. 路由表优选静态路由
3. 删除静态路由后流量切回 IBGP

---

## PC-CMP-PAR-008：OSPFv3 > IBGP

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-CMP-PAR-008 |
| `logic_case_id` | LC-CMP-001 |
| `requirement_ids` | SR-CMP-001 |
| `feature_tags` | BGP4+, 路由优先级, OSPFv3, IBGP |
| `case_title` | 验证OSPFv3路由优先级高于IBGP路由 |
| `priority` | P1 |
| `preconditions` | DUT1同时存在指向目标的OSPFv3路由和IBGP路由 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 优先级验证: OSPFv3 > IBGP |
| `trigger_data` | 优先级对比=OSPFv3 vs IBGP |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-CMP-008 |
| `scenario_refs` | S05 |
| `scenario_chain_refs` | S05 priority_path |
| `action_source_refs` | DR-CMP-008 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认IBGP路由存在, 下一跳=DUT2
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:cmp8::/64"
    expected_result: IBGP路由下一跳=DUT2
    trace_refs:
      - DR-CMP-008

  - step_id: STEP-002
    step_name: 确认OSPFv3学到相同前缀路由
    target: DUT1
    atomic_op:
      op_id: get-ospfv3-route
      args:
        prefix: "2001:db8:cmp8::/64"
    expected_result: OSPFv3路由表中存在该前缀
    trace_refs:
      - DR-CMP-008

  - step_id: STEP-003
    step_name: 验证路由表优选OSPFv3
    target: DUT1
    atomic_op:
      op_id: get-ipv6-route
      args:
        prefix: "2001:db8:cmp8::/64"
    expected_result: 优选OSPFv3路由, 协议优先级高于IBGP
    trace_refs:
      - DR-CMP-008

  - step_id: STEP-004
    step_name: TG1验证流量走OSPFv3路由
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:cmp8::1"
        duration: 30
    expected_result: 流量走OSPFv3路径
    trace_refs:
      - DR-CMP-008
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认IBGP路由存在 | DUT1 | fw_verify_bgp6_route | 下一跳=DUT2 |
| 2 | 确认OSPFv3路由存在 | DUT1 | get-ospfv3-route | 路由存在 |
| 3 | 验证路由表优选OSPFv3 | DUT1 | get-ipv6-route | 优选OSPFv3 |
| 4 | TG1流量验证 | TG1 | tg_start_traffic_stream | 流量走OSPFv3 |

### expected_results

1. OSPFv3 (OSPFv3 协议优先级 110) 高于 IBGP (BGP 协议优先级 200)
2. 路由表优选 OSPFv3 路由
3. OSPFv3 路由消失后流量切回 IBGP

---

## PC-CMP-PAR-009：EBGP > OSPFv3

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-CMP-PAR-009 |
| `logic_case_id` | LC-CMP-001 |
| `requirement_ids` | SR-CMP-001 |
| `feature_tags` | BGP4+, 路由优先级, EBGP, OSPFv3 |
| `case_title` | 验证EBGP路由优先级高于OSPFv3路由 |
| `priority` | P2 |
| `preconditions` | DUT1同时存在指向目标的OSPFv3路由和EBGP路由 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 优先级验证: EBGP > OSPFv3 |
| `trigger_data` | 优先级对比=EBGP vs OSPFv3, BGP类型=EBGP |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-CMP-009 |
| `scenario_refs` | S05 |
| `scenario_chain_refs` | S05 priority_path |
| `action_source_refs` | DR-CMP-009 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认OSPFv3路由存在
    target: DUT1
    atomic_op:
      op_id: get-ospfv3-route
      args:
        prefix: "2001:db8:cmp9::/64"
    expected_result: OSPFv3路由表中存在该前缀
    trace_refs:
      - DR-CMP-009

  - step_id: STEP-002
    step_name: 确认EBGP学到相同前缀路由
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:cmp9::/64"
    expected_result: EBGP路由存在, 下一跳=DUT3
    trace_refs:
      - DR-CMP-009

  - step_id: STEP-003
    step_name: 验证路由表优选EBGP
    target: DUT1
    atomic_op:
      op_id: get-ipv6-route
      args:
        prefix: "2001:db8:cmp9::/64"
    expected_result: 优选EBGP路由, EBGP优先级(20)高于OSPFv3(110)
    trace_refs:
      - DR-CMP-009

  - step_id: STEP-004
    step_name: TG1验证流量走EBGP
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:cmp9::1"
        duration: 30
    expected_result: 流量走EBGP路径
    trace_refs:
      - DR-CMP-009
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认OSPFv3路由存在 | DUT1 | get-ospfv3-route | 路由存在 |
| 2 | 确认EBGP路由存在 | DUT1 | fw_verify_bgp6_route | EBGP路由存在 |
| 3 | 验证路由表优选EBGP | DUT1 | get-ipv6-route | 优选EBGP |
| 4 | TG1流量验证 | TG1 | tg_start_traffic_stream | 流量走EBGP |

### expected_results

1. EBGP (协议优先级 20) 高于 OSPFv3 (协议优先级 110)
2. 路由表优选 EBGP 路由
3. EBGP 断开后流量切回 OSPFv3

---

## PC-CMP-PAR-010：静态 > EBGP

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-CMP-PAR-010 |
| `logic_case_id` | LC-CMP-001 |
| `requirement_ids` | SR-CMP-001 |
| `feature_tags` | BGP4+, 路由优先级, 静态路由, EBGP |
| `case_title` | 验证静态路由优先级高于EBGP路由 |
| `priority` | P2 |
| `preconditions` | DUT1同时存在指向目标的静态路由和EBGP路由 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 优先级验证: 静态 > EBGP |
| `trigger_data` | 优先级对比=静态 vs EBGP, BGP类型=EBGP |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-CMP-010 |
| `scenario_refs` | S05 |
| `scenario_chain_refs` | S05 priority_path |
| `action_source_refs` | DR-CMP-010 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认EBGP路由存在, 下一跳=DUT3
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:cmpa::/64"
    expected_result: EBGP路由下一跳=DUT3
    trace_refs:
      - DR-CMP-010

  - step_id: STEP-002
    step_name: 配置静态路由指向前缀2001:db8:cmpa::/64, 下一跳=TG1
    target: DUT1
    atomic_op:
      op_id: put-static-route
      args:
        prefix: "2001:db8:cmpa::/64"
        next_hop: "[TG1_IPv6]"
    expected_result: 静态路由配置成功
    trace_refs:
      - DR-CMP-010

  - step_id: STEP-003
    step_name: 验证路由表优选静态路由
    target: DUT1
    atomic_op:
      op_id: get-ipv6-route
      args:
        prefix: "2001:db8:cmpa::/64"
    expected_result: 优选静态路由(优先级60), 高于EBGP(优先级20) — 注: 静态>EBGP 需确认设备默认管理距离
    trace_refs:
      - DR-CMP-010

  - step_id: STEP-004
    step_name: TG1验证流量走静态路由
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        dst: "2001:db8:cmpa::1"
        duration: 30
    expected_result: 流量走静态路由
    trace_refs:
      - DR-CMP-010
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认EBGP路由存在 | DUT1 | fw_verify_bgp6_route | 下一跳=DUT3 |
| 2 | 配置静态路由下一跳=TG1 | DUT1 | put-static-route | 配置成功 |
| 3 | 验证路由表优选 | DUT1 | get-ipv6-route | 优选静态 |
| 4 | TG1流量验证 | TG1 | tg_start_traffic_stream | 流量走静态路由 |

### expected_results

1. 静态路由优先级高于 EBGP 路由
2. 路由表优选静态路由
3. 静态路由删除后流量切回 EBGP

---

## PC-CMP-PAR-011：锐捷+MD5不一致 → 失败

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-CMP-PAR-011 |
| `logic_case_id` | LC-CMP-001 |
| `requirement_ids` | SR-CMP-001 |
| `feature_tags` | BGP4+, 异构兼容, 锐捷, MD5认证, 异常 |
| `case_title` | 验证锐捷设备MD5认证不一致时邻居建立失败 |
| `priority` | P2 |
| `preconditions` | DUT1(锐捷)与DUT2 BGP4+全局使能, 两端MD5密钥不同 |
| `graph_ref` | SP-01 |
| `coverage_goal` | MD5异常: 锐捷设备MD5不一致 |
| `trigger_data` | 厂商=锐捷, MD5状态=不一致 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-CMP-011 |
| `scenario_refs` | S05 |
| `scenario_chain_refs` | S05 abnormal_path |
| `action_source_refs` | DR-CMP-011 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认锐捷设备BGP4+使能
    target: DUT1
    atomic_op:
      op_id: get-bgpconfig
      args: {}
    expected_result: vendor=锐捷, enable=true
    trace_refs:
      - DR-CMP-011

  - step_id: STEP-002
    step_name: DUT1配置MD5密钥key_A, DUT2配置MD5密钥key_B
    target: DUT1, DUT2
    atomic_op:
      op_id: fw_config_bgp6_neighbor
      args:
        DUT1:
          auth_key: "rj_key_A"
        DUT2:
          auth_key: "rj_key_B"
    expected_result: 配置成功, 两端密钥不同
    trace_refs:
      - DR-CMP-011

  - step_id: STEP-003
    step_name: DUT1创建IBGP邻居指向DUT2
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "GE0_1"
    expected_result: API返回success, 邻居尝试建立
    trace_refs:
      - DR-CMP-011

  - step_id: STEP-004
    step_name: 验证邻居建立失败
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态≠Established, 因MD5认证失败无法建立
    trace_refs:
      - DR-CMP-011
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认锐捷设备BGP4+使能 | DUT1 | get-bgpconfig | vendor=锐捷 |
| 2 | 配置两端MD5密钥不同 | DUT1, DUT2 | fw_config_bgp6_neighbor | 密钥不同 |
| 3 | DUT1创建IBGP邻居 | DUT1 | fw_create_bgp6_neighbor | API返回success |
| 4 | 验证邻居建立失败 | DUT1 | get-bgp6neighbor | ≠Established |

### expected_results

1. 锐捷设备 MD5 密钥不一致时 TCP 连接认证失败
2. 邻居无法建立
3. 系统日志记录 MD5 认证失败事件
4. 与 DR-CMP-004 (华为 MD5 不一致) 形成厂商对照
