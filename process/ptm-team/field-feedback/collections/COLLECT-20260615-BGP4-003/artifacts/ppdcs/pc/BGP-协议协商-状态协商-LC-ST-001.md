# BGP4+ 邻居状态机 — 物理用例

> 生成时间: 2026-06-12 | 设计Skill: state-design
> 逻辑用例: LC-ST-001 | 目录: BGP-协议协商-状态协商

---

## PC-ST-001-01：邻居六态正向迁移

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-ST-001-01 |
| `logic_case_id` | LC-ST-001 |
| `requirement_ids` | SR-010 |
| `feature_tags` | BGP4+, 状态机, 邻居建立 |
| `case_title` | 验证BGP4+邻居按Idle→Connect→OpenSent→OpenConfirm→Established正向建立 |
| `priority` | P0 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能, 接口IPv6直连互通 (2001:db8:1::/64) |
| `graph_ref` | SP-01 |
| `coverage_goal` | 主生命周期: 六态正向全覆盖 |
| `trigger_data` | POST bgp6neighbor {ip="2001:db8:1::2", asNumber=100, interface="GE0_1", ebgpMultihop=1} |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M3-001, S04-OP2, S04-OP3 |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | S04 normal_path |
| `action_source_refs` | S04-OP2, S04-OP3 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认BGP4+全局使能且接口IPv6就绪
    target: DUT1, DUT2
    atomic_op:
      op_id: get-bgpconfig
      args: {}
    expected_result: enable=true, asNumber=100, routerId为合法IPv4
    trace_refs:
      - TP-M3-001

  - step_id: STEP-002
    step_name: DUT1创建IBGP邻居指向DUT2
    target: DUT1
    atomic_op:
      op_id: post-bgp6neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "GE0_1"
        ebgpMultihop: 1
    expected_result: API返回success
    trace_refs:
      - S04-OP2

  - step_id: STEP-003
    step_name: 观察邻居状态从Idle迁移至Connect
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Connect (TCP连接发起中)
    trace_refs:
      - TP-M3-001

  - step_id: STEP-004
    step_name: 观察邻居状态迁移至OpenSent
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=OpenSent (OPEN消息已发送)
    trace_refs:
      - TP-M3-001

  - step_id: STEP-005
    step_name: 观察邻居状态迁移至OpenConfirm
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=OpenConfirm (OPEN已确认)
    trace_refs:
      - TP-M3-001

  - step_id: STEP-006
    step_name: 最终验证邻居达到Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, 可交换UPDATE消息
    trace_refs:
      - TP-M3-001
      - S04-OP3
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认BGP4+全局使能 | DUT1, DUT2 | get-bgpconfig | enable=true, asNumber=100 |
| 2 | DUT1创建IBGP邻居指向DUT2 | DUT1 | post-bgp6neighbor ip=2001:db8:1::2, asNumber=100, interface=GE0_1 | API返回success |
| 3 | 观察邻居状态: Idle→Connect | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 状态=Connect |
| 4 | 观察邻居状态: Connect→OpenSent | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 状态=OpenSent |
| 5 | 观察邻居状态: OpenSent→OpenConfirm | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 状态=OpenConfirm |
| 6 | 最终验证Established | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 状态=Established |

### expected_results

1. 邻居严格按照 Idle → Connect → OpenSent → OpenConfirm → Established 顺序迁移
2. 各中间状态可被观察到（不跳过任何状态）
3. 最终 Established 状态稳定，可交换路由

---

## PC-ST-001-03：物理口链路故障恢复

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-ST-001-03 |
| `logic_case_id` | LC-ST-001 |
| `requirement_ids` | SR-010 |
| `feature_tags` | BGP4+, 状态机, 链路故障, 物理口 |
| `case_title` | 验证物理口链路down后邻居进入Active, up后恢复Established |
| `priority` | P0 |
| `preconditions` | DUT1-DUT2 IBGP邻居已Established (通过物理口GE0_1) |
| `graph_ref` | SP-03 |
| `coverage_goal` | 回退恢复: 物理口 T11→T18 (S5→S2→S5) |
| `trigger_data` | 接口类型=物理口, admin down → admin up |
| `topology_binding_refs` | node3_dut2_tg1_link3 (L1: DUT1.port1 ↔ DUT2.port1) |
| `trace_refs` | TP-M3-005, S08-OP3, S08-OP4 |
| `scenario_refs` | S08 |
| `scenario_chain_refs` | S08 normal_path 步骤4→5 |
| `action_source_refs` | S08-OP3, S08-OP4 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认邻居Established (物理口GE0_1)
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, interface=GE0_1
    trace_refs:
      - TP-M3-005

  - step_id: STEP-002
    step_name: 物理口admin down
    target: DUT1
    atomic_op:
      op_id: set-interface-down
      args:
        interface: "GE0_1"
    expected_result: 接口状态=down
    trace_refs:
      - S08-OP3

  - step_id: STEP-003
    step_name: 观察邻居状态变化
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Active (检测到链路故障)
    trace_refs:
      - TP-M3-005

  - step_id: STEP-004
    step_name: 物理口admin up恢复
    target: DUT1
    atomic_op:
      op_id: set-interface-up
      args:
        interface: "GE0_1"
    expected_result: 接口状态=up
    trace_refs:
      - S08-OP4

  - step_id: STEP-005
    step_name: 验证邻居重建Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established (TCP重建 → OpenSent→OpenConfirm→Established)
    trace_refs:
      - TP-M3-005
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认邻居Established | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 状态=Established, interface=GE0_1 |
| 2 | 物理口admin down | DUT1 | set-interface-down interface=GE0_1 | 接口状态=down |
| 3 | 观察邻居状态变化 | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 状态=Active |
| 4 | 物理口admin up恢复 | DUT1 | set-interface-up interface=GE0_1 | 接口状态=up |
| 5 | 验证邻居重建Established | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 状态=Established |

### expected_results

1. 物理口 down 后邻居进入 Active
2. 物理口 up 后邻居自动重建到 Established
3. 恢复过程经历 Connect→OpenSent→OpenConfirm→Established

---

## PC-ST-001-09：Keepalive超时断开

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-ST-001-09 |
| `logic_case_id` | LC-ST-001 |
| `requirement_ids` | SR-010 |
| `feature_tags` | BGP4+, 状态机, Keepalive超时 |
| `case_title` | 验证Keepalive全部丢失Hold Timer(180s)超时后邻居进入Idle |
| `priority` | P0 |
| `preconditions` | DUT1-DUT2 IBGP邻居已Established, Hold Time=180s |
| `graph_ref` | SP-09 |
| `coverage_goal` | 异常回退: T12 (S5→S0) Keepalive超时 |
| `trigger_data` | 安全策略阻断TCP/179或Keepalive报文, Hold Timer=180s |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M3-003, S08-OP5 |
| `scenario_refs` | S08 |
| `scenario_chain_refs` | S08 normal_path 步骤6, abnormal_path "keepalive全部丢失" |
| `action_source_refs` | S08-OP5 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认邻居Established, Hold Time=180s
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, holdTime=180s, keepaliveTime=60s
    trace_refs:
      - TP-M3-003

  - step_id: STEP-002
    step_name: 安全策略阻断Keepalive报文 (或阻断TCP/179端口)
    target: DUT1 (或中间网络设备)
    atomic_op:
      op_id: block-keepalive
      args:
        src: "2001:db8:1::1"
        dst: "2001:db8:1::2"
        port: 179
    expected_result: Keepalive报文被丢弃, 对端持续未收到Keepalive
    trace_refs:
      - S08-OP5

  - step_id: STEP-003
    step_name: 等待Hold Timer(180s)超时后观察邻居状态
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Idle (Hold Timer超时)
    trace_refs:
      - TP-M3-003

  - step_id: STEP-004
    step_name: 解除Keepalive阻断
    target: DUT1 (或中间网络设备)
    atomic_op:
      op_id: unblock-keepalive
      args:
        src: "2001:db8:1::1"
        dst: "2001:db8:1::2"
    expected_result: 解除阻断
    trace_refs:
      - TP-M3-003

  - step_id: STEP-005
    step_name: 验证邻居自动重建Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established (自动重建)
    trace_refs:
      - TP-M3-003
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认邻居Established | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 状态=Established, holdTime=180s |
| 2 | 阻断Keepalive报文 | 网络设备 | block-keepalive | Keepalive被丢弃 |
| 3 | 等180s后观察邻居 | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 状态=Idle |
| 4 | 解除阻断 | 网络设备 | unblock-keepalive | 阻断解除 |
| 5 | 验证邻居自动重建 | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 状态=Established |

### expected_results

1. Keepalive 阻断后 Hold Timer(180s) 超时，邻居进入 Idle
2. 解除阻断后邻居自动重建到 Established
3. 超时时间准确为 180s（±允许误差）

---

## PC-ST-001-12：RouterID冲突导致邻居无法建立

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-ST-001-12 |
| `logic_case_id` | LC-ST-001 |
| `requirement_ids` | SR-004, SR-010 |
| `feature_tags` | BGP4+, 状态机, RouterID冲突, 非法迁移 |
| `case_title` | 验证两端RouterID相同时OPEN消息校验失败邻居无法建立 |
| `priority` | P1 |
| `preconditions` | DUT1 RouterID=1.1.1.1, DUT2 RouterID=1.1.1.1 (相同) |
| `graph_ref` | SP-12 |
| `coverage_goal` | 非法迁移: T13 RouterID冲突 (S5→S0) |
| `trigger_data` | DUT1 routerId="1.1.1.1", DUT2 routerId="1.1.1.1" |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | LC-ST-001 P15 |
| `scenario_refs` | S04 |
| `scenario_chain_refs` | — |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 配置DUT1和DUT2为相同RouterID
    target: DUT1, DUT2
    atomic_op:
      op_id: put-bgpconfig
      args:
        routerId: "1.1.1.1"
    expected_result: API返回success, 两端RouterID均为1.1.1.1
    trace_refs:
      - LC-ST-001 P15

  - step_id: STEP-002
    step_name: DUT1创建IBGP邻居指向DUT2
    target: DUT1
    atomic_op:
      op_id: post-bgp6neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "GE0_1"
    expected_result: API返回success, 发起连接
    trace_refs:
      - LC-ST-001 P15

  - step_id: STEP-003
    step_name: 验证邻居无法建立 (Bad BGP Identifier)
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态≠Established, 收到NOTIFICATION (Error Code: Bad BGP Identifier)
    trace_refs:
      - LC-ST-001 P15

  - step_id: STEP-004
    step_name: 清理: 还原RouterID
    target: DUT2
    atomic_op:
      op_id: put-bgpconfig
      args:
        routerId: "2.2.2.2"
    expected_result: DUT2 RouterID恢复为2.2.2.2
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 配置两端相同RouterID | DUT1, DUT2 | put-bgpconfig routerId=1.1.1.1 | API返回success |
| 2 | DUT1创建IBGP邻居 | DUT1 | post-bgp6neighbor ip=2001:db8:1::2, asNumber=100 | API返回success |
| 3 | 验证邻居无法建立 | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | NOTIFICATION: Bad BGP Identifier |
| 4 | 清理: 还原RouterID | DUT2 | put-bgpconfig routerId=2.2.2.2 | 恢复 |

### expected_results

1. OPEN 消息中检测到 RouterID 冲突 (两端相同)
2. 发送 NOTIFICATION 错误码: Bad BGP Identifier
3. 邻居无法进入 Established，回到 Idle

---

## PC-ST-001-16：Hold Time=0 [待确认]

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-ST-001-16 |
| `logic_case_id` | LC-ST-001 |
| `requirement_ids` | SR-010 |
| `feature_tags` | BGP4+, 状态机, Hold Time边界 |
| `case_title` | 验证Hold Time=0时邻居协商行为 [待确认] |
| `priority` | P2 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能 |
| `graph_ref` | SP-18 |
| `coverage_goal` | 边界守卫: Hold Time=0 [待确认] |
| `trigger_data` | DUT1 Hold Time=0, DUT2 Hold Time=180s |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | LC-ST-001 P12 |
| `scenario_refs` | — |
| `scenario_chain_refs` | — |
| `action_source_refs` | — |
| `confirmation_gap_refs` | GAP-ST-001: Hold Time=0 行为待确认 |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: "[待确认] 设置DUT1 Hold Time=0, DUT2 Hold Time=180s"
    target: DUT1, DUT2
    atomic_op:
      op_id: put-bgpconfig
      args:
        holdTime: "[待确认: 0 或 3(最小)]"
    expected_result: "[待确认] API是否接受Hold Time=0"
    trace_refs:
      - LC-ST-001 P12
    confirmation_gap_refs:
      - GAP-ST-001

  - step_id: STEP-002
    step_name: "[待确认] 创建邻居并观察协商结果"
    target: DUT1
    atomic_op:
      op_id: post-bgp6neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
    expected_result: "[待确认] 协商后Hold Time取值: 双方都不发Keepalive, 或仅DUT2侧=180s"
    trace_refs:
      - LC-ST-001 P12
    confirmation_gap_refs:
      - GAP-ST-001

  - step_id: STEP-003
    step_name: "[待确认] 验证Keepalive发送行为"
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: "[待确认] Keepalive定时器=0(不发送) 或 使用协商值"
    trace_refs:
      - LC-ST-001 P12
    confirmation_gap_refs:
      - GAP-ST-001
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | [待确认] 设置DUT1 Hold Time=0 | DUT1, DUT2 | put-bgpconfig holdTime=[待确认] | [待确认] |
| 2 | [待确认] 创建邻居观察协商 | DUT1 | post-bgp6neighbor | [待确认] |
| 3 | [待确认] 验证Keepalive行为 | DUT1 | get-bgp6neighbor | [待确认] |

### expected_results

[待确认] 需确认设备是否支持 Hold Time=0：
- 若支持: 双方不发送 Keepalive，邻居永不超时断开
- 若不支持: API 拒绝 Hold Time=0，或内部使用最小值(如3s)
