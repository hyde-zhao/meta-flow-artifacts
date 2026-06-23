# BGP4+ 容量/性能边界 — 物理用例

> 生成时间: 2026-06-12 | 设计Skill: parameter-design
> 逻辑用例: LC-REL-005 | 目录: BGP-协议协商-路由通告

---

## PC-REL5-PAR-001：创建16个BGP4+邻居全部 Established

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL5-PAR-001 |
| `logic_case_id` | LC-REL-005 |
| `requirement_ids` | SR-REL-005 |
| `feature_tags` | BGP4+, 容量边界, 邻居数量, 规格 |
| `case_title` | 验证创建16个BGP4+邻居全部进入Established |
| `priority` | P1 |
| `preconditions` | DUT1 BGP4+全局使能, 有16个可用IPv6邻居地址 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 容量边界: 最大规格=16个邻居全部建立 |
| `trigger_data` | 创建16个邻居, 邻居数量=16 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M5-001 |
| `scenario_refs` | S-REL-005 |
| `scenario_chain_refs` | S-REL-005 normal_path 步骤1 |
| `action_source_refs` | S-REL-005-OP1 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认BGP4+全局使能
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: BGP enable=true, AS号=100
    trace_refs:
      - TP-REL-M5-001

  - step_id: STEP-002
    step_name: 依次创建16个BGP4+邻居
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        count: 16
        start_ip: "2001:db8:1::1"
        as_number: 100
    expected_result: 16个邻居全部创建成功
    trace_refs:
      - S-REL-005-OP1

  - step_id: STEP-003
    step_name: 验证所有邻居状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        all: true
    expected_result: 16个邻居状态=Established
    trace_refs:
      - TP-REL-M5-001

  - step_id: STEP-004
    step_name: 验证邻居间路由交换正常
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args: {}
    expected_result: 从各邻居学习到路由, 路由表项符合预期
    trace_refs:
      - TP-REL-M5-001
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认BGP全局使能 | DUT1 | fw_verify_bgp_global | enable=true |
| 2 | 创建16个BGP4+邻居 | DUT1 | fw_create_bgp6_neighbor count=16 | 全部创建成功 |
| 3 | 验证邻居全部Established | DUT1 | fw_verify_bgp6_neighbor all=true | 16个全部=Established |
| 4 | 验证路由交换正常 | DUT1 | fw_verify_bgp6_route | 路由表正常 |

### expected_results

1. 16个BGP4+邻居全部创建成功并进入 Established
2. 每个邻居可正常交换 UPDATE/Keepalive 消息
3. 邻居间无互相干扰
4. DUT1 能同时维护16个邻居的 BGP 状态机

---

## PC-REL5-PAR-002：创建第17个超规格邻居 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL5-PAR-002 |
| `logic_case_id` | LC-REL-005 |
| `requirement_ids` | SR-REL-005 |
| `feature_tags` | BGP4+, 容量边界, 超规格, 拒绝 |
| `case_title` | 验证创建第17个BGP4+邻居超出规格被拒绝 |
| `priority` | P1 |
| `preconditions` | DUT1已有16个BGP4+邻居Established, 达到最大规格 |
| `graph_ref` | SP-02 |
| `coverage_goal` | 容量边界: 超规格拒绝 (17 > 16) |
| `trigger_data` | 创建第17个邻居, 邻居数量=17 (超规格) |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M5-002 |
| `scenario_refs` | S-REL-005 |
| `scenario_chain_refs` | S-REL-005 abnormal_path "超规格邻居" |
| `action_source_refs` | S-REL-005-OP2 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认当前已有16个邻居Established
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        all: true
    expected_result: 当前邻居数=16, 全部Established
    trace_refs:
      - TP-REL-M5-002

  - step_id: STEP-002
    step_name: 尝试创建第17个BGP4+邻居
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        count: 1
        ip: "2001:db8:1::12"
        as_number: 100
    expected_result: API返回失败, 提示超出邻居规格上限
    trace_refs:
      - S-REL-005-OP2

  - step_id: STEP-003
    step_name: 验证已有16个邻居不受影响
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        all: true
    expected_result: 原有16个邻居状态仍为Established
    trace_refs:
      - TP-REL-M5-002
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认16个邻居Established | DUT1 | fw_verify_bgp6_neighbor all=true | 邻居数=16 |
| 2 | 尝试创建第17个邻居 | DUT1 | fw_create_bgp6_neighbor count=1 | API拒绝, 超规格 |
| 3 | 验证已有邻居不受影响 | DUT1 | fw_verify_bgp6_neighbor all=true | 16个仍Established |

### expected_results

1. 创建第17个邻居时 API 返回明确错误 (超规格)
2. 错误信息包含规格限制说明 (max=16)
3. 已有16个邻居状态不受影响
4. 不允许部分创建或静默失败

---

## PC-REL5-PAR-003：注入5K路由全部学习成功

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL5-PAR-003 |
| `logic_case_id` | LC-REL-005 |
| `requirement_ids` | SR-REL-005 |
| `feature_tags` | BGP4+, 容量边界, 路由容量, 5K |
| `case_title` | 验证注入5K条BGP4+路由后DUT全部学习成功 |
| `priority` | P1 |
| `preconditions` | DUT1-DUT2 IBGP邻居已Established |
| `graph_ref` | SP-03 |
| `coverage_goal` | 容量边界: 5K路由全学习 |
| `trigger_data` | 注入路由量=5000条 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M5-003 |
| `scenario_refs` | S-REL-005 |
| `scenario_chain_refs` | S-REL-005 normal_path 步骤2 |
| `action_source_refs` | S-REL-005-OP3 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认邻居Established, 初始路由表为空或已知
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args: {}
    expected_result: 路由表初始状态已记录
    trace_refs:
      - TP-REL-M5-003

  - step_id: STEP-002
    step_name: DUT2通告5K条BGP4+路由到DUT1
    target: DUT2
    atomic_op:
      op_id: fw_inject_bgp6_route
      args:
        count: 5000
        prefix_range: "2001:db8::/32"
    expected_result: 5K条路由通告成功
    trace_refs:
      - S-REL-005-OP3

  - step_id: STEP-003
    step_name: 验证DUT1学到全部5K条路由
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        expected_count: 5000
    expected_result: DUT1路由表包含5K条BGP4+路由, 全部正确
    trace_refs:
      - TP-REL-M5-003

  - step_id: STEP-004
    step_name: 验证邻居Established
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established
    trace_refs:
      - TP-REL-M5-003
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认初始路由表 | DUT1 | fw_verify_bgp6_route | 基线已记录 |
| 2 | 注入5K条路由 | DUT2 | fw_inject_bgp6_route count=5000 | 5K条通告成功 |
| 3 | 验证DUT1学到5K条 | DUT1 | fw_verify_bgp6_route expected_count=5000 | 5K条路由正确 |
| 4 | 验证邻居状态 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 邻居=Established |

### expected_results

1. DUT1成功学习全部5K条BGP4+路由
2. 路由前缀/下一跳/AS-Path/Community等属性完整正确
3. 邻居在路由学习期间保持 Established
4. 路由学习完成时间合理 (不超时)

---

## PC-REL5-PAR-004：注入50K路由全部学习成功 内存增长<20%

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL5-PAR-004 |
| `logic_case_id` | LC-REL-005 |
| `requirement_ids` | SR-REL-005 |
| `feature_tags` | BGP4+, 容量边界, 路由容量, 50K, 内存 |
| `case_title` | 验证注入50K条BGP4+路由全部学习成功且内存增长<20% |
| `priority` | P1 |
| `preconditions` | DUT1-DUT2 IBGP邻居已Established, 记录bgpd内存基线 |
| `graph_ref` | SP-04 |
| `coverage_goal` | 容量边界: 50K路由全学习, 内存增长<20% |
| `trigger_data` | 注入路由量=50000条 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M5-004 |
| `scenario_refs` | S-REL-005 |
| `scenario_chain_refs` | S-REL-005 normal_path 步骤3 |
| `action_source_refs` | S-REL-005-OP4 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 记录bgpd内存基线和初始路由表
    target: DUT1
    atomic_op:
      op_id: get-system-memory
      args:
        process: "bgpd"
    expected_result: bgpd内存基线M0, 路由表初始状态已记录
    trace_refs:
      - TP-REL-M5-004

  - step_id: STEP-002
    step_name: DUT2通告50K条BGP4+路由到DUT1
    target: DUT2
    atomic_op:
      op_id: fw_inject_bgp6_route
      args:
        count: 50000
        prefix_range: "2001:db8::/32"
    expected_result: 50K条路由通告成功
    trace_refs:
      - S-REL-005-OP4

  - step_id: STEP-003
    step_name: 验证DUT1学到全部50K条路由
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        expected_count: 50000
    expected_result: DUT1路由表包含50K条BGP4+路由
    trace_refs:
      - TP-REL-M5-004

  - step_id: STEP-004
    step_name: 验证bgpd内存增长
    target: DUT1
    atomic_op:
      op_id: get-system-memory
      args:
        process: "bgpd"
    expected_result: bgpd内存M1, (M1-M0)/M0 < 20%
    trace_refs:
      - TP-REL-M5-004

  - step_id: STEP-005
    step_name: 验证邻居Established
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established
    trace_refs:
      - TP-REL-M5-004
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 记录bgpd内存基线 | DUT1 | get-system-memory process=bgpd | 基线M0 |
| 2 | 注入50K条路由 | DUT2 | fw_inject_bgp6_route count=50000 | 50K条通告成功 |
| 3 | 验证50K条路由 | DUT1 | fw_verify_bgp6_route expected_count=50000 | 50K条正确 |
| 4 | 验证内存增长 | DUT1 | get-system-memory process=bgpd | (M1-M0)/M0 < 20% |
| 5 | 验证邻居状态 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 邻居=Established |

### expected_results

1. DUT1成功学习全部50K条BGP4+路由
2. bgpd内存增长 <20% (与基线相比)
3. 路由学习过程中无异常日志或进程重启
4. 邻居在大量路由学习后仍保持 Established

---

## PC-REL5-PAR-005：注入160K路由全部学习成功

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL5-PAR-005 |
| `logic_case_id` | LC-REL-005 |
| `requirement_ids` | SR-REL-005 |
| `feature_tags` | BGP4+, 容量边界, 路由容量, 160K, 最大规格 |
| `case_title` | 验证注入160K条BGP4+路由后DUT全部学习成功 |
| `priority` | P1 |
| `preconditions` | DUT1-DUT2 IBGP邻居已Established, 有足够内存容纳160K路由 |
| `graph_ref` | SP-05 |
| `coverage_goal` | 容量边界: 160K路由全学习 (最大路由规格) |
| `trigger_data` | 注入路由量=160000条 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M5-005 |
| `scenario_refs` | S-REL-005 |
| `scenario_chain_refs` | S-REL-005 normal_path 步骤4 |
| `action_source_refs` | S-REL-005-OP5 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认初始状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: BGP全局使能, 邻居=Established
    trace_refs:
      - TP-REL-M5-005

  - step_id: STEP-002
    step_name: DUT2分批通告160K条BGP4+路由到DUT1
    target: DUT2
    atomic_op:
      op_id: fw_inject_bgp6_route
      args:
        count: 160000
        prefix_range: "2001:db8::/32"
        batch_size: 10000
    expected_result: 160K条路由分16批通告完成
    trace_refs:
      - S-REL-005-OP5

  - step_id: STEP-003
    step_name: 验证DUT1学到全部160K条路由
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        expected_count: 160000
    expected_result: DUT1路由表包含160K条BGP4+路由, 前缀/属性正确
    trace_refs:
      - TP-REL-M5-005

  - step_id: STEP-004
    step_name: 验证bgpd进程稳定
    target: DUT1
    atomic_op:
      op_id: fw_verify_process
      args:
        process_name: "bgpd"
    expected_result: bgpd进程=running, 内存使用正常
    trace_refs:
      - TP-REL-M5-005

  - step_id: STEP-005
    step_name: 验证邻居Established
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established
    trace_refs:
      - TP-REL-M5-005
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认初始状态 | DUT1 | fw_verify_bgp_global | 使能, 邻居=Established |
| 2 | 分批注入160K条路由 | DUT2 | fw_inject_bgp6_route count=160000, batch_size=10000 | 分批通告完成 |
| 3 | 验证160K条路由 | DUT1 | fw_verify_bgp6_route expected_count=160000 | 160K条正确 |
| 4 | 验证bgpd稳定 | DUT1 | fw_verify_process process_name=bgpd | bgpd=running |
| 5 | 验证邻居状态 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 邻居=Established |

### expected_results

1. DUT1成功学习全部160K条BGP4+路由 (最大规格)
2. 路由学习过程中 bgpd 进程稳定不崩溃
3. 所有路由前缀/属性正确
4. 邻居在最大路由规格下保持 Established

---

## PC-REL5-PAR-006：收敛时间 ≤5s 达标

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL5-PAR-006 |
| `logic_case_id` | LC-REL-005 |
| `requirement_ids` | SR-REL-005 |
| `feature_tags` | BGP4+, 性能边界, 收敛时间, ≤5s |
| `case_title` | 验证BGP4+路由收敛时间≤5s达标 |
| `priority` | P1 |
| `preconditions` | DUT1-DUT2 IBGP邻居已Established, 路由已稳定, 可精确计时 |
| `graph_ref` | SP-06 |
| `coverage_goal` | 性能边界: 收敛时间≤5s |
| `trigger_data` | 收敛时间=≤5s, 触发路由变更并计时 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M5-006 |
| `scenario_refs` | S-REL-005 |
| `scenario_chain_refs` | S-REL-005 normal_path 步骤5 |
| `action_source_refs` | S-REL-005-OP6 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认邻居Established且路由稳定
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args: {}
    expected_result: 路由表已稳定, 邻居=Established
    trace_refs:
      - TP-REL-M5-006

  - step_id: STEP-002
    step_name: 触发路由变更 (新增路由) 并记录起始时间T0
    target: DUT2
    atomic_op:
      op_id: fw_inject_bgp6_route
      args:
        count: 1000
        timestamp: true
    expected_result: 记录T0, 1000条新路由开始通告
    trace_refs:
      - S-REL-005-OP6

  - step_id: STEP-003
    step_name: 等待DUT1路由表收敛并记录结束时间T1
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        expected_count: "<初始+1000>"
        timeout: 10
    expected_result: T1 = 路由表达到预期的时间, 收敛时间 = T1-T0 ≤ 5s
    trace_refs:
      - TP-REL-M5-006

  - step_id: STEP-004
    step_name: 验证收敛时间 ≤5s
    target: DUT1
    atomic_op:
      op_id: assert-timing
      args:
        delta: "T1-T0"
        max: 5s
    expected_result: 收敛时间 ≤5s
    trace_refs:
      - TP-REL-M5-006
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认路由稳定 | DUT1 | fw_verify_bgp6_route | 路由稳定 |
| 2 | 触发变更记录T0 | DUT2 | fw_inject_bgp6_route count=1000, timestamp=true | T0已记录 |
| 3 | 等待收敛记录T1 | DUT1 | fw_verify_bgp6_route expected_count=<预期> | T1已记录 |
| 4 | 验证收敛时间 | DUT1 | assert-timing delta=T1-T0, max=5s | 收敛时间≤5s |

### expected_results

1. 收敛时间 = 从路由变更触发到 DUT1 路由表完全达到预期的时间
2. 收敛时间 ≤ 5s
3. 收敛过程中邻居保持 Established
4. 多次测量取平均值, 排除异常值

---

## PC-REL5-PAR-007：建立时间 ≤5s 达标

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL5-PAR-007 |
| `logic_case_id` | LC-REL-005 |
| `requirement_ids` | SR-REL-005 |
| `feature_tags` | BGP4+, 性能边界, 建立时间, ≤5s |
| `case_title` | 验证BGP4+邻居建立时间≤5s达标 |
| `priority` | P1 |
| `preconditions` | DUT1 DUT2 BGP4+全局使能, 邻居已配置但未连接 |
| `graph_ref` | SP-07 |
| `coverage_goal` | 性能边界: 建立时间≤5s |
| `trigger_data` | 建立时间=≤5s, 触发邻居建立并计时 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M5-007 |
| `scenario_refs` | S-REL-005 |
| `scenario_chain_refs` | S-REL-005 normal_path 步骤6 |
| `action_source_refs` | S-REL-005-OP7 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确保邻居当前不存在或处于Idle状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Idle 或 不存在
    trace_refs:
      - TP-REL-M5-007

  - step_id: STEP-002
    step_name: 创建BGP4+邻居并记录起始时间T0
    target: DUT1
    atomic_op:
      op_id: post-bgp6neighbor
      args:
        ip: "2001:db8:1::2"
        asNumber: 100
        interface: "GE0_1"
        timestamp: true
    expected_result: T0 = 邻居创建时间, API返回success
    trace_refs:
      - S-REL-005-OP7

  - step_id: STEP-003
    step_name: 轮询邻居直到Established并记录T1
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor-until-established
      args:
        id: "2001:db8:1::2"
        timeout: 10
    expected_result: T1 = 邻居Established时间, 建立时间 = T1-T0
    trace_refs:
      - TP-REL-M5-007

  - step_id: STEP-004
    step_name: 验证建立时间 ≤5s
    target: DUT1
    atomic_op:
      op_id: assert-timing
      args:
        delta: "T1-T0"
        max: 5s
    expected_result: 建立时间 ≤5s
    trace_refs:
      - TP-REL-M5-007
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认邻居为Idle | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 状态=Idle |
| 2 | 创建邻居记录T0 | DUT1 | post-bgp6neighbor timestamp=true | T0已记录 |
| 3 | 轮询Established记录T1 | DUT1 | get-bgp6neighbor-until-established | T1已记录 |
| 4 | 验证建立时间 | DUT1 | assert-timing delta=T1-T0, max=5s | 建立时间≤5s |

### expected_results

1. 建立时间 = 从邻居创建到状态达到 Established 的时间
2. 建立时间 ≤ 5s
3. 建立过程经历 Idle → Connect → OpenSent → OpenConfirm → Established
4. TCP 连接和 BGP OPEN 协商在时间要求内完成

---

## PC-REL5-PAR-008：收敛时间 >5s 不达标 [待确认]

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL5-PAR-008 |
| `logic_case_id` | LC-REL-005 |
| `requirement_ids` | SR-REL-005 |
| `feature_tags` | BGP4+, 性能边界, 收敛时间, >5s, 边界 |
| `case_title` | 验证收敛时间>5s时的系统行为 [待确认] |
| `priority` | P2 |
| `preconditions` | DUT1大路由量 (接近160K), 构造收敛时间>5s场景 |
| `graph_ref` | SP-08 |
| `coverage_goal` | 性能边界: 收敛时间>5s, 不达标行为 [待确认] |
| `trigger_data` | 大路由量下触发变更, 收敛时间>5s |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M5-008 |
| `scenario_refs` | S-REL-005 |
| `scenario_chain_refs` | S-REL-005 abnormal_path "收敛时间>5s" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | GAP-REL-004: 收敛时间>5s行为待确认 |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: "[待确认] 构造大路由量场景使收敛时间>5s"
    target: DUT1, DUT2
    atomic_op:
      op_id: fw_inject_bgp6_route
      args:
        count: "[待确认: 接近160K路由 + 大批量变更]"
    expected_result: "[待确认] 路由量大, 预期收敛时间>5s"
    trace_refs:
      - TP-REL-M5-008
    confirmation_gap_refs:
      - GAP-REL-004

  - step_id: STEP-002
    step_name: "[待确认] 触发路由变更并计时"
    target: DUT2
    atomic_op:
      op_id: fw_inject_bgp6_route
      args:
        count: "[待确认]"
        timestamp: true
    expected_result: "[待确认] 收敛时间>5s, DUT1路由表逐步更新"
    trace_refs:
      - TP-REL-M5-008
    confirmation_gap_refs:
      - GAP-REL-004

  - step_id: STEP-003
    step_name: "[待确认] 观察收敛过程中的邻居和路由状态"
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: "[待确认] 邻居保持Established, 路由持续更新但未达预期"
    trace_refs:
      - TP-REL-M5-008
    confirmation_gap_refs:
      - GAP-REL-004
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | [待确认] 构造大路由场景 | DUT1, DUT2 | fw_inject_bgp6_route count=[待确认] | [待确认] 收敛时间>5s |
| 2 | [待确认] 触发变更计时 | DUT2 | fw_inject_bgp6_route timestamp=true | [待确认] 收敛时间>5s |
| 3 | [待确认] 观察过程行为 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | [待确认] |

### expected_results

[待确认] 需确认收敛时间>5s时的系统行为：
- 邻居是否保持 Established (不因收敛慢而断开)
- 路由是否最终能全部正确收敛
- 是否有告警或性能降级通知
- CPU/内存使用是否异常

---

## PC-REL5-PAR-009：建立时间 >5s 不达标 [待确认]

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL5-PAR-009 |
| `logic_case_id` | LC-REL-005 |
| `requirement_ids` | SR-REL-005 |
| `feature_tags` | BGP4+, 性能边界, 建立时间, >5s, 边界 |
| `case_title` | 验证邻居建立时间>5s时的系统行为 [待确认] |
| `priority` | P2 |
| `preconditions` | DUT1高负载 (CPU高/多邻居同时建立), 构造建立时间>5s场景 |
| `graph_ref` | SP-09 |
| `coverage_goal` | 性能边界: 建立时间>5s, 不达标行为 [待确认] |
| `trigger_data` | 高负载下创建邻居, 建立时间>5s |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M5-009 |
| `scenario_refs` | S-REL-005 |
| `scenario_chain_refs` | S-REL-005 abnormal_path "建立时间>5s" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | GAP-REL-005: 建立时间>5s行为待确认 |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: "[待确认] 构造高负载场景使建立时间>5s"
    target: DUT1
    atomic_op:
      op_id: "[待确认] stress_cpu 或 同时创建15个邻居"
      args: {}
    expected_result: "[待确认] DUT1 CPU负载高, 预期建立时间>5s"
    trace_refs:
      - TP-REL-M5-009
    confirmation_gap_refs:
      - GAP-REL-005

  - step_id: STEP-002
    step_name: "[待确认] 创建新邻居并计时"
    target: DUT1
    atomic_op:
      op_id: post-bgp6neighbor
      args:
        ip: "2001:db8:1::12"
        asNumber: 100
        timestamp: true
    expected_result: "[待确认] 建立时间>5s, 邻居最终可能仍能建立"
    trace_refs:
      - TP-REL-M5-009
    confirmation_gap_refs:
      - GAP-REL-005

  - step_id: STEP-003
    step_name: "[待确认] 轮询邻居直到Established或超时"
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor-until-established
      args:
        id: "2001:db8:1::12"
        timeout: 30
    expected_result: "[待确认] 建立时间>5s 或 建立失败"
    trace_refs:
      - TP-REL-M5-009
    confirmation_gap_refs:
      - GAP-REL-005
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | [待确认] 构造高负载场景 | DUT1 | [待确认] | [待确认] 建立时间>5s |
| 2 | [待确认] 创建邻居计时 | DUT1 | post-bgp6neighbor timestamp=true | [待确认] 建立时间>5s |
| 3 | [待确认] 轮询邻接结果 | DUT1 | get-bgp6neighbor-until-established timeout=30s | [待确认] |

### expected_results

[待确认] 需确认建立时间>5s时的系统行为：
- 邻居是否能最终建立成功 (即使 >5s)
- 是否有建立超时导致失败的上限
- 高负载下 BGP 进程是否有优先级保护
- 多邻居同时建立时的排队行为

---

## PC-REL5-PAR-010：16邻居+160K路由+线速流量组合压力

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL5-PAR-010 |
| `logic_case_id` | LC-REL-005 |
| `requirement_ids` | SR-REL-005 |
| `feature_tags` | BGP4+, 性能边界, 组合压力, 最大规格 |
| `case_title` | 验证16邻居+160K路由+线速流量组合压力下系统正常 |
| `priority` | P1 |
| `preconditions` | DUT1 16个BGP4+邻居全部Established, 路由160K, TG1就绪 |
| `graph_ref` | SP-10 |
| `coverage_goal` | 组合压力: 16邻居+160K路由+线速流量 |
| `trigger_data` | 16邻居, 160K路由, 线速数据面流量同时运行 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M5-010 |
| `scenario_refs` | S-REL-005 |
| `scenario_chain_refs` | S-REL-005 normal_path 步骤8 |
| `action_source_refs` | S-REL-005-OP8 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认16个邻居全部Established, 160K路由已学习
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        all: true
    expected_result: 16个邻居=Established, 路由=160K
    trace_refs:
      - TP-REL-M5-010

  - step_id: STEP-002
    step_name: TG1发送线速数据面流量
    target: TG1
    atomic_op:
      op_id: tg_start_traffic_stream
      args:
        rate: "line_rate"
        duration: 600
    expected_result: 线速流量启动
    trace_refs:
      - S-REL-005-OP8

  - step_id: STEP-003
    step_name: 组合压力期间持续验证邻居状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        all: true
        duration: 600
    expected_result: 16个邻居保持Established, 无震荡
    trace_refs:
      - TP-REL-M5-010

  - step_id: STEP-004
    step_name: 验证路由表稳定
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args: {}
    expected_result: 160K路由表稳定, 无丢失或异常
    trace_refs:
      - TP-REL-M5-010

  - step_id: STEP-005
    step_name: 停止流量并验证系统恢复正常
    target: TG1
    atomic_op:
      op_id: tg_stop_traffic_stream
      args: {}
    expected_result: 流量停止, 系统状态正常, 邻居仍Established
    trace_refs:
      - TP-REL-M5-010
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认规格满载 | DUT1 | fw_verify_bgp6_neighbor all=true | 16邻居Established, 160K路由 |
| 2 | 发送线速流量 | TG1 | tg_start_traffic_stream rate=line_rate, duration=600s | 线速流量 |
| 3 | 验证邻居稳定 | DUT1 | fw_verify_bgp6_neighbor all=true, duration=600s | 16邻居保持Established |
| 4 | 验证路由表 | DUT1 | fw_verify_bgp6_route | 160K路由稳定 |
| 5 | 停止流量验证 | TG1 | tg_stop_traffic_stream | 系统正常 |

### expected_results

1. 16邻居+160K路由+线速流量组合压力下系统不崩溃
2. 所有邻居保持 Established, 无震荡
3. 路由表 (160K) 完整无丢失
4. 数据面转发表与路由表一致
5. CPU/内存使用在合理范围内

---

## PC-REL5-PAR-011：16邻居各100条路由+1h长稳

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL5-PAR-011 |
| `logic_case_id` | LC-REL-005 |
| `requirement_ids` | SR-REL-005 |
| `feature_tags` | BGP4+, 性能边界, 长稳, 多邻居 |
| `case_title` | 验证16个邻居各100条路由持续1小时长稳运行 |
| `priority` | P2 |
| `preconditions` | DUT1 16个BGP4+邻居全部Established, 每个邻居通告100条路由 |
| `graph_ref` | SP-11 |
| `coverage_goal` | 长稳压力: 16邻居 × 100路由 × 1小时 |
| `trigger_data` | 16邻居, 每邻居100条路由, 连续运行1h |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M5-011 |
| `scenario_refs` | S-REL-005 |
| `scenario_chain_refs` | S-REL-005 normal_path 步骤9 |
| `action_source_refs` | S-REL-005-OP9 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认16个邻居各100条路由已学习
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        all: true
    expected_result: 16个邻居=Established, 每个邻居路由数=100, 总路由=1600
    trace_refs:
      - TP-REL-M5-011

  - step_id: STEP-002
    step_name: 长稳运行1小时, 每10分钟采样验证
    target: DUT1
    atomic_op:
      op_id: monitor-stability
      args:
        duration: 3600
        interval: 600
        metrics:
          - neighbor_state
          - route_count
          - bgpd_memory
          - bgpd_cpu
    expected_result: 每次采样邻居状态稳定, 路由数不变
    trace_refs:
      - S-REL-005-OP9

  - step_id: STEP-003
    step_name: 1h后验证所有邻居和路由
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        all: true
    expected_result: 16个邻居=Established, 每个邻居100条路由无丢失
    trace_refs:
      - TP-REL-M5-011

  - step_id: STEP-004
    step_name: 验证bgpd内存和CPU稳定
    target: DUT1
    atomic_op:
      op_id: get-system-memory
      args:
        process: "bgpd"
    expected_result: 内存增长<5%, CPU稳定无突刺
    trace_refs:
      - TP-REL-M5-011
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认初始状态 | DUT1 | fw_verify_bgp6_neighbor all=true | 16邻居各100条路由 |
| 2 | 1h长稳运行采样 | DUT1 | monitor-stability duration=3600s | 每次采样正常 |
| 3 | 验证最终邻居路由 | DUT1 | fw_verify_bgp6_neighbor all=true | 16邻居Established, 路由无丢失 |
| 4 | 验证资源使用 | DUT1 | get-system-memory process=bgpd | 内存/CPU稳定 |

### expected_results

1. 16邻居各100条路由在1小时内稳定无震荡
2. 所有邻居保持 Established
3. 路由表无路由丢失或异常
4. bgpd 内存增长 <5%, CPU 无异常突刺
5. 无日志错误或告警

---
