# BGP4+ 链路/邻居级可靠性 — 物理用例

> 生成时间: 2026-06-12 | 设计Skill: parameter-design
> 逻辑用例: LC-REL-003 | 目录: BGP-协议协商-状态协商

---

## PC-REL3-PAR-001：链路down后邻居进入 Active/Connect, admin up 恢复

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL3-PAR-001 |
| `logic_case_id` | LC-REL-003 |
| `requirement_ids` | SR-REL-003 |
| `feature_tags` | BGP4+, 链路可靠性, 链路down, 自动恢复 |
| `case_title` | 验证物理链路down后邻居进入Active/Connect, admin up后自动恢复Established |
| `priority` | P1 |
| `preconditions` | DUT1-DUT2 IBGP邻居已Established, 通过物理链路直连 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 链路级可靠性: 链路down → Active/Connect → admin up恢复 |
| `trigger_data` | 链路故障类型=链路down, admin down接口, 再admin up |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M3-001 |
| `scenario_refs` | S-REL-003 |
| `scenario_chain_refs` | S-REL-003 normal_path 步骤1→2→3 |
| `action_source_refs` | S-REL-003-OP1, S-REL-003-OP2 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认邻居Established
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, 物理链路正常
    trace_refs:
      - TP-REL-M3-001

  - step_id: STEP-002
    step_name: 物理链路admin down
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        interface: "GE0_1"
        state: "admin_down"
    expected_result: 接口状态=down
    trace_refs:
      - S-REL-003-OP1

  - step_id: STEP-003
    step_name: 观察邻居状态变化
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Active或Connect (检测到链路故障)
    trace_refs:
      - TP-REL-M3-001

  - step_id: STEP-004
    step_name: 物理链路admin up恢复
    target: DUT1
    atomic_op:
      op_id: fw_config_interface
      args:
        interface: "GE0_1"
        state: "admin_up"
    expected_result: 接口状态=up
    trace_refs:
      - S-REL-003-OP2

  - step_id: STEP-005
    step_name: 验证邻居自动重建Established
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, 路由恢复
    trace_refs:
      - TP-REL-M3-001
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认邻居Established | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 邻居=Established |
| 2 | 链路admin down | DUT1 | fw_config_interface interface=GE0_1, state=admin_down | 接口=down |
| 3 | 观察邻居状态 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 状态=Active/Connect |
| 4 | 链路admin up | DUT1 | fw_config_interface interface=GE0_1, state=admin_up | 接口=up |
| 5 | 验证邻居恢复 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 邻居=Established |

### expected_results

1. 物理链路 down 后 BGP4+ 邻居从 Established 迁移到 Active/Connect
2. 物理链路 admin up 后 TCP 连接重建, 邻居自动恢复到 Established
3. 恢复过程经过 Connect→OpenSent→OpenConfirm→Established 标准状态迁移

---

## PC-REL3-PAR-002：keepalive全部丢失 Hold Timer 超时→Idle→恢复

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL3-PAR-002 |
| `logic_case_id` | LC-REL-003 |
| `requirement_ids` | SR-REL-003 |
| `feature_tags` | BGP4+, 链路可靠性, Keepalive超时, Hold Timer, 自动恢复 |
| `case_title` | 验证Keepalive全部丢失Hold Timer(180s)超时后进入Idle, 恢复后重建 |
| `priority` | P1 |
| `preconditions` | DUT1-DUT2 IBGP邻居已Established, Hold Time=180s, Keepalive Time=60s |
| `graph_ref` | SP-02 |
| `coverage_goal` | 邻居级可靠性: keepalive全部丢失→Hold Timer超时→Idle→恢复 |
| `trigger_data` | Keepalive报文全部丢失, Hold Timer=180s |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M3-002 |
| `scenario_refs` | S-REL-003 |
| `scenario_chain_refs` | S-REL-003 normal_path 步骤4→5→6 |
| `action_source_refs` | S-REL-003-OP3, S-REL-003-OP4 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认邻居Established, Hold Time=180s
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, holdTime=180s, keepaliveTime=60s
    trace_refs:
      - TP-REL-M3-002

  - step_id: STEP-002
    step_name: 阻断所有Keepalive报文
    target: DUT1
    atomic_op:
      op_id: fw_block_packet
      args:
        protocol: "bgp"
        port: 179
        src: "2001:db8:1::1"
        dst: "2001:db8:1::2"
    expected_result: Keepalive报文被全部丢弃
    trace_refs:
      - S-REL-003-OP3

  - step_id: STEP-003
    step_name: 等待Hold Timer(180s)超时后观察邻居状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Idle (Hold Timer超时)
    trace_refs:
      - TP-REL-M3-002

  - step_id: STEP-004
    step_name: 解除Keepalive阻断
    target: DUT1
    atomic_op:
      op_id: fw_unblock_packet
      args:
        protocol: "bgp"
        port: 179
        src: "2001:db8:1::1"
        dst: "2001:db8:1::2"
    expected_result: 阻断解除, Keepalive恢复正常
    trace_refs:
      - S-REL-003-OP4

  - step_id: STEP-005
    step_name: 验证邻居自动重建Established
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established (自动重建)
    trace_refs:
      - TP-REL-M3-002
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认邻居Established | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 状态=Established, holdTime=180s |
| 2 | 阻断Keepalive | DUT1 | fw_block_packet protocol=bgp | Keepalive全部丢失 |
| 3 | 等180s观察邻居 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 状态=Idle |
| 4 | 解除阻断 | DUT1 | fw_unblock_packet protocol=bgp | Keepalive恢复 |
| 5 | 验证邻居恢复 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 状态=Established |

### expected_results

1. Keepalive 全部丢失后 Hold Timer(180s) 准确超时
2. 邻居从 Established 进入 Idle
3. Keepalive 恢复后邻居自动重建到 Established
4. 超时时间准确为 180s (允许正常误差)

---

## PC-REL3-PAR-003：keepalive部分丢失50% 保持 Established

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL3-PAR-003 |
| `logic_case_id` | LC-REL-003 |
| `requirement_ids` | SR-REL-003 |
| `feature_tags` | BGP4+, 链路可靠性, Keepalive部分丢失, 稳定性 |
| `case_title` | 验证Keepalive丢包率50%时邻居保持Established不震荡 |
| `priority` | P2 |
| `preconditions` | DUT1-DUT2 IBGP邻居已Established, Hold Time=180s |
| `graph_ref` | SP-03 |
| `coverage_goal` | 边界守卫: keepalive部分丢失50%时邻居稳定 |
| `trigger_data` | 丢包率=50%, Keepalive部分丢失 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M3-003 |
| `scenario_refs` | S-REL-003 |
| `scenario_chain_refs` | S-REL-003 normal_path 步骤7 |
| `action_source_refs` | S-REL-003-OP5 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认邻居Established
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, Hold Time=180s
    trace_refs:
      - TP-REL-M3-003

  - step_id: STEP-002
    step_name: 对BGP Keepalive报文施加50%丢包率
    target: DUT1
    atomic_op:
      op_id: fw_block_packet
      args:
        protocol: "bgp"
        port: 179
        loss_rate: 50
    expected_result: Keepalive报文丢包率=50%
    trace_refs:
      - S-REL-003-OP5

  - step_id: STEP-003
    step_name: Hold Timer周期(900s)内持续观察邻居状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
        duration: 900
    expected_result: 邻居状态=Established, 无震荡, Hold Timer不超时
    trace_refs:
      - TP-REL-M3-003

  - step_id: STEP-004
    step_name: 验证路由交换正常
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args: {}
    expected_result: 路由表稳定, 无路由撤回
    trace_refs:
      - TP-REL-M3-003

  - step_id: STEP-005
    step_name: 恢复0%丢包
    target: DUT1
    atomic_op:
      op_id: fw_unblock_packet
      args:
        protocol: "bgp"
        port: 179
    expected_result: Keepalive恢复正常, 邻居仍为Established
    trace_refs:
      - TP-REL-M3-003
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认邻居Established | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 状态=Established |
| 2 | 施加50%丢包 | DUT1 | fw_block_packet loss_rate=50 | 丢包率=50% |
| 3 | 长周期观察邻居 | DUT1 | fw_verify_bgp6_neighbor duration=900s | 状态=Established无震荡 |
| 4 | 验证路由交换正常 | DUT1 | fw_verify_bgp6_route | 路由稳定 |
| 5 | 恢复0%丢包 | DUT1 | fw_unblock_packet | 邻居仍为Established |

### expected_results

1. Keepalive 丢包率 50% 时, Hold Timer 经过 5 个 Keepalive 周期后未超时
2. 邻居在整个观察周期内始终保持 Established
3. 路由表稳定, 无路由震荡或撤回
4. 恢复 0% 丢包后邻居和路由均正常

---

## PC-REL3-PAR-004：大流量数据面压力下控制面保持 Established

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL3-PAR-004 |
| `logic_case_id` | LC-REL-003 |
| `requirement_ids` | SR-REL-003 |
| `feature_tags` | BGP4+, 链路可靠性, 数据面压力, 控制面隔离 |
| `case_title` | 验证大流量数据面压力下BGP控制面邻居保持Established |
| `priority` | P2 |
| `preconditions` | DUT1-DUT2 IBGP邻居已Established, TG1可产生大流量 |
| `graph_ref` | SP-04 |
| `coverage_goal` | 压力守卫: 大流量数据面 → 控制面稳定 |
| `trigger_data` | 大流量数据面压力 (线速流量), 观察BGP控制面状态 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M3-004 |
| `scenario_refs` | S-REL-003 |
| `scenario_chain_refs` | S-REL-003 normal_path 步骤8→9 |
| `action_source_refs` | S-REL-003-OP6 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认邻居Established, 数据面无流量
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, 控制面正常
    trace_refs:
      - TP-REL-M3-004

  - step_id: STEP-002
    step_name: TG1发送线速大流量
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        rate: "line_rate"
        duration: 600
    expected_result: 数据面产生线速大流量
    trace_refs:
      - S-REL-003-OP6

  - step_id: STEP-003
    step_name: 大流量持续期间观察BGP邻居状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
        duration: 600
    expected_result: 邻居状态=Established, 无震荡, 控制面Keepalive正常
    trace_refs:
      - TP-REL-M3-004

  - step_id: STEP-004
    step_name: 验证路由表稳定
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args: {}
    expected_result: 路由表稳定, 无路由震荡
    trace_refs:
      - TP-REL-M3-004

  - step_id: STEP-005
    step_name: 停止大流量并验证控制面
    target: TG1
    atomic_op:
      op_id: tg_stop_traffic_stream
      args: {}
    expected_result: 流量停止, 邻居仍为Established
    trace_refs:
      - TP-REL-M3-004
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认邻居Established | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 状态=Established |
| 2 | TG1发送线速流量 | TG1 | tg_start_traffic_stream rate=line_rate, duration=600s | 大流量产生 |
| 3 | 大流量期间观察邻居 | DUT1 | fw_verify_bgp6_neighbor duration=600s | 状态=Established无震荡 |
| 4 | 验证路由表 | DUT1 | fw_verify_bgp6_route | 路由稳定 |
| 5 | 停止流量验证 | TG1 | tg_stop_traffic_stream | 邻居仍为Established |

### expected_results

1. 线速大流量下数据面不挤占 BGP 控制面资源 (CPU/队列)
2. BGP Keepalive 报文优先级足够高, 不被数据面流量冲掉
3. 邻居在整个大流量期间保持不变 Established
4. 路由表稳定, 无因控制面拥塞导致的路由震荡

---
