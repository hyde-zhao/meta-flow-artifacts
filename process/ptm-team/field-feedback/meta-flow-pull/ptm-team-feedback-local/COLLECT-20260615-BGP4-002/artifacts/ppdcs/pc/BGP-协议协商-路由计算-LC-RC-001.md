# BGP4+ 路由选路 — 物理用例

> 生成时间: 2026-06-12 | 设计Skill: parameter-design
> 逻辑用例: LC-RC-001 | 目录: BGP-协议协商-路由计算

---

## PC-RC-PAR-001：IBGP MED=100 vs MED=50 选MED=50

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RC-PAR-001 |
| `logic_case_id` | LC-RC-001 |
| `requirement_ids` | SR-RC-001 |
| `feature_tags` | BGP4+, 路由选路, IBGP, MED比较 |
| `case_title` | 验证IBGP两条路径MED=100和MED=50时优选MED=50 |
| `priority` | P1 |
| `preconditions` | DUT1与DUT2/DUT3建立IBGP邻居均Established, DUT2通告路由MED=100, DUT3通告路由MED=50 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 选路基线: IBGP MED比较选小 |
| `trigger_data` | 路径来源=IBGP, MED_A=100, MED_C=50 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-01 |
| `scenario_refs` | S02 |
| `scenario_chain_refs` | S02 normal_path |
| `action_source_refs` | DR-01 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认DUT1与DUT2/DUT3 IBGP邻居Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: DUT2邻居状态=Established
    trace_refs:
      - DR-01

  - step_id: STEP-002
    step_name: 确认DUT1与DUT3 IBGP邻居Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:2::2"
    expected_result: DUT3邻居状态=Established
    trace_refs:
      - DR-01

  - step_id: STEP-003
    step_name: DUT2通告路由MED=100, DUT3通告路由MED=50
    target: DUT2, DUT3
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        prefix: "2001:db8:100::/64"
        med:
          DUT2: 100
          DUT3: 50
    expected_result: 路由配置成功
    trace_refs:
      - DR-01

  - step_id: STEP-004
    step_name: DUT1验证选路结果选择MED=50路径
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:100::/64"
    expected_result: 优选路径为DUT3方向 (MED=50)
    trace_refs:
      - DR-01
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认DUT1-DUT2 IBGP Established | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 状态=Established |
| 2 | 确认DUT1-DUT3 IBGP Established | DUT1 | get-bgp6neighbor id=2001:db8:2::2 | 状态=Established |
| 3 | DUT2 MED=100, DUT3 MED=50通告同一路由 | DUT2, DUT3 | fw_config_bgp_global | 配置成功 |
| 4 | DUT1验证优选MED=50 | DUT1 | fw_verify_bgp6_route | 优选DUT3, MED=50 |

### expected_results

1. DUT1 收到两条 IBGP 路径: MED=100 和 MED=50
2. BGP 选路算法比较 MED，选择较小的 MED=50
3. 路由表中优选下一跳指向 DUT3

---

## PC-RC-PAR-002：IBGP vs EBGP 选EBGP

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RC-PAR-002 |
| `logic_case_id` | LC-RC-001 |
| `requirement_ids` | SR-RC-001 |
| `feature_tags` | BGP4+, 路由选路, IBGP, EBGP, AS路径 |
| `case_title` | 验证同一条路由IBGP和EBGP同时存在时优选EBGP路径 |
| `priority` | P1 |
| `preconditions` | DUT1与DUT2 IBGP Established, DUT1与DUT3 EBGP Established |
| `graph_ref` | SP-01 |
| `coverage_goal` | 选路基线: EBGP优先于IBGP |
| `trigger_data` | 路径来源=IBGP vs EBGP |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-02 |
| `scenario_refs` | S02 |
| `scenario_chain_refs` | S02 normal_path |
| `action_source_refs` | DR-02 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认DUT1-DUT2 IBGP Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: DUT2 IBGP邻居状态=Established
    trace_refs:
      - DR-02

  - step_id: STEP-002
    step_name: 确认DUT1-DUT3 EBGP Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:3::2"
    expected_result: DUT3 EBGP邻居状态=Established, AS不同
    trace_refs:
      - DR-02

  - step_id: STEP-003
    step_name: DUT2和DUT3同时通告同一路由
    target: DUT2, DUT3
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        prefix: "2001:db8:200::/64"
    expected_result: 路由配置成功
    trace_refs:
      - DR-02

  - step_id: STEP-004
    step_name: DUT1验证选路结果为EBGP
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:200::/64"
    expected_result: 优选路径为EBGP (DUT3方向)
    trace_refs:
      - DR-02
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认DUT1-DUT2 IBGP Established | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 状态=Established |
| 2 | 确认DUT1-DUT3 EBGP Established | DUT1 | get-bgp6neighbor id=2001:db8:3::2 | 状态=Established |
| 3 | 两端通告同一路由 | DUT2, DUT3 | fw_config_bgp_global | 配置成功 |
| 4 | DUT1验证优选EBGP | DUT1 | fw_verify_bgp6_route | 优选DUT3, 下一跳=EBGP |

### expected_results

1. DUT1 同时收到 IBGP 和 EBGP 路径
2. BGP 选路规则 EBGP 优先级高于 IBGP
3. 路由表中优选下一跳指向 EBGP 邻居

---

## PC-RC-PAR-003：本地路由 vs BGP 本地优先

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RC-PAR-003 |
| `logic_case_id` | LC-RC-001 |
| `requirement_ids` | SR-RC-001 |
| `feature_tags` | BGP4+, 路由选路, 本地路由, 优先级 |
| `case_title` | 验证本地路由与BGP路由存在同一前缀时优先本地路由 |
| `priority` | P1 |
| `preconditions` | DUT1本地存在直连路由, DUT2通过IBGP通告相同前缀路由 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 选路基线: 本地路由优先于BGP路由 |
| `trigger_data` | 路径来源=本地 vs BGP |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-03 |
| `scenario_refs` | S02 |
| `scenario_chain_refs` | S02 normal_path |
| `action_source_refs` | DR-03 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认DUT1本地存在直连路由
    target: DUT1
    atomic_op:
      op_id: get-ipv6-route
      args:
        prefix: "2001:db8:300::/64"
    expected_result: 本地路由表中存在直连路由
    trace_refs:
      - DR-03

  - step_id: STEP-002
    step_name: DUT2通过IBGP通告相同前缀路由
    target: DUT2
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        prefix: "2001:db8:300::/64"
    expected_result: BGP路由配置成功
    trace_refs:
      - DR-03

  - step_id: STEP-003
    step_name: DUT1验证路由选择本地路由而非BGP
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:300::/64"
    expected_result: 本地直连路由优先, BGP路由未被优选
    trace_refs:
      - DR-03
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认DUT1本地存在直连路由 | DUT1 | get-ipv6-route prefix=2001:db8:300::/64 | 本地路由存在 |
| 2 | DUT2通告相同前缀BGP路由 | DUT2 | fw_config_bgp_global | 配置成功 |
| 3 | DUT1验证本地路由优先 | DUT1 | fw_verify_bgp6_route | 本地路由优先于BGP |

### expected_results

1. 本地路由（直连/静态）优先级高于 BGP 路由
2. 相同前缀存在时选择本地路由
3. BGP 路由在表中但未被优选

---

## PC-RC-PAR-004：EBGP断开后切IBGP

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RC-PAR-004 |
| `logic_case_id` | LC-RC-001 |
| `requirement_ids` | SR-RC-001 |
| `feature_tags` | BGP4+, 路由选路, 故障切换, EBGP, IBGP |
| `case_title` | 验证EBGP邻居断开后路由自动切换到IBGP路径 |
| `priority` | P1 |
| `preconditions` | DUT1 EBGP/IBGP双路径均Established, 当前优选EBGP |
| `graph_ref` | SP-01 |
| `coverage_goal` | 故障切换: EBGP断 → 切IBGP |
| `trigger_data` | EBGP断开 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-04 |
| `scenario_refs` | S02 |
| `scenario_chain_refs` | S02 failure_path |
| `action_source_refs` | DR-04 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认DUT1优选EBGP路径
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:200::/64"
    expected_result: 优选EBGP路径
    trace_refs:
      - DR-04

  - step_id: STEP-002
    step_name: 断开DUT1-DUT3 EBGP邻居
    target: DUT1
    atomic_op:
      op_id: fw_update_bgp6_neighbor
      args:
        id: "2001:db8:3::2"
        admin_status: down
    expected_result: EBGP邻居断开
    trace_refs:
      - DR-04

  - step_id: STEP-003
    step_name: DUT1验证路由切换至IBGP
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:200::/64"
    expected_result: 优选路径切换为IBGP (DUT2方向)
    trace_refs:
      - DR-04

  - step_id: STEP-004
    step_name: 恢复EBGP邻居
    target: DUT1
    atomic_op:
      op_id: fw_update_bgp6_neighbor
      args:
        id: "2001:db8:3::2"
        admin_status: up
    expected_result: EBGP邻居恢复Established
    trace_refs:
      - DR-04

  - step_id: STEP-005
    step_name: DUT1验证路由切回EBGP
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:200::/64"
    expected_result: 优选路径切回EBGP
    trace_refs:
      - DR-04
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认当前优选EBGP | DUT1 | fw_verify_bgp6_route | 优选EBGP |
| 2 | 断开EBGP邻居 | DUT1 | fw_update_bgp6_neighbor admin_status=down | EBGP断开 |
| 3 | 验证路由切至IBGP | DUT1 | fw_verify_bgp6_route | 优选IBGP |
| 4 | 恢复EBGP邻居 | DUT1 | fw_update_bgp6_neighbor admin_status=up | EBGP恢复 |
| 5 | 验证路由切回EBGP | DUT1 | fw_verify_bgp6_route | 切回EBGP |

### expected_results

1. EBGP 断开后 DUT1 自动将路由切换到 IBGP 路径
2. EBGP 恢复后路由自动切回 EBGP
3. 切换过程中流量不中断（或中断时间可接受）

---

## PC-RC-PAR-005：仅IBGP单路径

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RC-PAR-005 |
| `logic_case_id` | LC-RC-001 |
| `requirement_ids` | SR-RC-001 |
| `feature_tags` | BGP4+, 路由选路, IBGP, 单路径 |
| `case_title` | 验证仅一条IBGP路径时正常选路 |
| `priority` | P2 |
| `preconditions` | DUT1仅与DUT2 IBGP Established, DUT2通告路由 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 边界条件: 仅IBGP单路径 |
| `trigger_data` | 路径来源=IBGP, 单一路径 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-05 |
| `scenario_refs` | S02 |
| `scenario_chain_refs` | S02 boundary_path |
| `action_source_refs` | DR-05 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认DUT1仅与DUT2 IBGP Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args: {}
    expected_result: 仅有DUT2一个IBGP邻居Established
    trace_refs:
      - DR-05

  - step_id: STEP-002
    step_name: DUT2通告路由
    target: DUT2
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        prefix: "2001:db8:500::/64"
    expected_result: 路由配置成功
    trace_refs:
      - DR-05

  - step_id: STEP-003
    step_name: DUT1验证唯一IBGP路由被优选
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:500::/64"
    expected_result: 路由存在且为优选路径
    trace_refs:
      - DR-05
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认仅一条IBGP邻居 | DUT1 | get-bgp6neighbor | 仅DUT2 Established |
| 2 | DUT2通告路由 | DUT2 | fw_config_bgp_global | 配置成功 |
| 3 | DUT1验证路由优选 | DUT1 | fw_verify_bgp6_route | 路由存在且优选 |

### expected_results

1. 单条 IBGP 路径正常选路
2. 路由表中该路径为唯一优选
3. 无异常日志或错误

---

## PC-RC-PAR-006：仅EBGP单路径

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RC-PAR-006 |
| `logic_case_id` | LC-RC-001 |
| `requirement_ids` | SR-RC-001 |
| `feature_tags` | BGP4+, 路由选路, EBGP, 单路径 |
| `case_title` | 验证仅一条EBGP路径时正常选路 |
| `priority` | P2 |
| `preconditions` | DUT1仅与DUT3 EBGP Established, DUT3通告路由 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 边界条件: 仅EBGP单路径 |
| `trigger_data` | 路径来源=EBGP, 单一路径 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-06 |
| `scenario_refs` | S02 |
| `scenario_chain_refs` | S02 boundary_path |
| `action_source_refs` | DR-06 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认DUT1仅与DUT3 EBGP Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args: {}
    expected_result: 仅有DUT3一个EBGP邻居Established
    trace_refs:
      - DR-06

  - step_id: STEP-002
    step_name: DUT3通告路由
    target: DUT3
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        prefix: "2001:db8:600::/64"
    expected_result: 路由配置成功
    trace_refs:
      - DR-06

  - step_id: STEP-003
    step_name: DUT1验证唯一EBGP路由被优选
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:600::/64"
    expected_result: 路由存在且为优选路径
    trace_refs:
      - DR-06
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认仅一条EBGP邻居 | DUT1 | get-bgp6neighbor | 仅DUT3 Established |
| 2 | DUT3通告路由 | DUT3 | fw_config_bgp_global | 配置成功 |
| 3 | DUT1验证路由优选 | DUT1 | fw_verify_bgp6_route | 路由存在且优选 |

### expected_results

1. 单条 EBGP 路径正常选路
2. 路由表中该路径为唯一优选
3. 无异常日志或错误

---

## PC-RC-PAR-007：MED=0 边界验证选MED=0

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RC-PAR-007 |
| `logic_case_id` | LC-RC-001 |
| `requirement_ids` | SR-RC-001 |
| `feature_tags` | BGP4+, 路由选路, MED边界, IBGP |
| `case_title` | 验证MED=0与MED=50比较时优选MED=0 |
| `priority` | P2 |
| `preconditions` | DUT1与DUT2/DUT3 IBGP Established |
| `graph_ref` | SP-01 |
| `coverage_goal` | MED边界: MED=0选路验证 |
| `trigger_data` | MED_A=0, MED_C=50 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-07 |
| `scenario_refs` | S02 |
| `scenario_chain_refs` | S02 boundary_path |
| `action_source_refs` | DR-07 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认DUT1与DUT2/DUT3 IBGP Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args: {}
    expected_result: DUT2和DUT3邻居状态=Established
    trace_refs:
      - DR-07

  - step_id: STEP-002
    step_name: DUT2 MED=0, DUT3 MED=50通告同一路由
    target: DUT2, DUT3
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        prefix: "2001:db8:700::/64"
        med:
          DUT2: 0
          DUT3: 50
    expected_result: 路由配置成功
    trace_refs:
      - DR-07

  - step_id: STEP-003
    step_name: DUT1验证优选MED=0
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:700::/64"
    expected_result: 优选MED=0路径 (DUT2方向)
    trace_refs:
      - DR-07
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认IBGP邻居Established | DUT1 | get-bgp6neighbor | DUT2/DUT3 Established |
| 2 | DUT2 MED=0, DUT3 MED=50 | DUT2, DUT3 | fw_config_bgp_global | 配置成功 |
| 3 | DUT1验证优选MED=0 | DUT1 | fw_verify_bgp6_route | 优选DUT2, MED=0 |

### expected_results

1. MED=0 为最小合法值，选路算法正确比较
2. MED=0 路径被优选（MED 值越小越优先）
3. 边界行为与正常值一致

---

## PC-RC-PAR-008：MED反转验证 MED(A)=100, MED(C)=50

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RC-PAR-008 |
| `logic_case_id` | LC-RC-001 |
| `requirement_ids` | SR-RC-001 |
| `feature_tags` | BGP4+, 路由选路, MED反转 |
| `case_title` | 验证MED值反转后（A=100→50, C=50→100）选路结果正确反转 |
| `priority` | P2 |
| `preconditions` | DUT1与DUT2/DUT3 IBGP Established, 初始MED(A)=100, MED(C)=50 |
| `graph_ref` | SP-01 |
| `coverage_goal` | MED反转: 值互换后选路方向互换 |
| `trigger_data` | MED(A)=100, MED(C)=50, 反转后 MED(A)=50, MED(C)=100 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-08 |
| `scenario_refs` | S02 |
| `scenario_chain_refs` | S02 reversal_path |
| `action_source_refs` | DR-08 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 初始态: DUT2 MED=100, DUT3 MED=50
    target: DUT2, DUT3
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        prefix: "2001:db8:800::/64"
        med:
          DUT2: 100
          DUT3: 50
    expected_result: 路由配置成功
    trace_refs:
      - DR-08

  - step_id: STEP-002
    step_name: DUT1验证初始优选MED=50 (DUT3)
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:800::/64"
    expected_result: 优选DUT3, MED=50
    trace_refs:
      - DR-08

  - step_id: STEP-003
    step_name: 反转MED: DUT2 MED=50, DUT3 MED=100
    target: DUT2, DUT3
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        prefix: "2001:db8:800::/64"
        med:
          DUT2: 50
          DUT3: 100
    expected_result: MED值交换成功
    trace_refs:
      - DR-08

  - step_id: STEP-004
    step_name: DUT1验证选路反转优选DUT2
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:800::/64"
    expected_result: 优选DUT2, MED=50
    trace_refs:
      - DR-08
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 初始MED: DUT2=100, DUT3=50 | DUT2, DUT3 | fw_config_bgp_global | 配置成功 |
| 2 | 验证初始优选DUT3 MED=50 | DUT1 | fw_verify_bgp6_route | 优选DUT3 |
| 3 | 反转MED: DUT2=50, DUT3=100 | DUT2, DUT3 | fw_config_bgp_global | MED交换成功 |
| 4 | 验证选路反转优选DUT2 | DUT1 | fw_verify_bgp6_route | 优选DUT2 |

### expected_results

1. 初始 MED(A)=100, MED(C)=50 时优选 C
2. MED 值互换后选路结果正确反转，优选 A
3. 选路算法对 MED 值变化响应正确

---

## PC-RC-PAR-009：MED反转 MED(A)=50, MED(C)=100

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RC-PAR-009 |
| `logic_case_id` | LC-RC-001 |
| `requirement_ids` | SR-RC-001 |
| `feature_tags` | BGP4+, 路由选路, MED反转, 对向验证 |
| `case_title` | 验证MED初始值为A=50, C=100时反转后选路结果正确 |
| `priority` | P2 |
| `preconditions` | DUT1与DUT2/DUT3 IBGP Established |
| `graph_ref` | SP-01 |
| `coverage_goal` | MED反转: 对向初始值验证 |
| `trigger_data` | MED(A)=50, MED(C)=100, 反转后 MED(A)=100, MED(C)=50 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-09 |
| `scenario_refs` | S02 |
| `scenario_chain_refs` | S02 reversal_path |
| `action_source_refs` | DR-09 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 初始态: DUT2 MED=50, DUT3 MED=100
    target: DUT2, DUT3
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        prefix: "2001:db8:900::/64"
        med:
          DUT2: 50
          DUT3: 100
    expected_result: 路由配置成功
    trace_refs:
      - DR-09

  - step_id: STEP-002
    step_name: DUT1验证初始优选MED=50 (DUT2)
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:900::/64"
    expected_result: 优选DUT2, MED=50
    trace_refs:
      - DR-09

  - step_id: STEP-003
    step_name: 反转MED: DUT2 MED=100, DUT3 MED=50
    target: DUT2, DUT3
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        prefix: "2001:db8:900::/64"
        med:
          DUT2: 100
          DUT3: 50
    expected_result: MED值交换成功
    trace_refs:
      - DR-09

  - step_id: STEP-004
    step_name: DUT1验证选路反转优选DUT3
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:900::/64"
    expected_result: 优选DUT3, MED=50
    trace_refs:
      - DR-09
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 初始MED: DUT2=50, DUT3=100 | DUT2, DUT3 | fw_config_bgp_global | 配置成功 |
| 2 | 验证初始优选DUT2 MED=50 | DUT1 | fw_verify_bgp6_route | 优选DUT2 |
| 3 | 反转MED: DUT2=100, DUT3=50 | DUT2, DUT3 | fw_config_bgp_global | MED交换成功 |
| 4 | 验证选路反转优选DUT3 | DUT1 | fw_verify_bgp6_route | 优选DUT3 |

### expected_results

1. 初始 MED(A)=50, MED(C)=100 时优选 A
2. MED 值互换后选路结果正确反转为优选 C
3. 与 DR-08 形成对向验证覆盖

---

## PC-RC-PAR-010：MED=0 边界反转

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RC-PAR-010 |
| `logic_case_id` | LC-RC-001 |
| `requirement_ids` | SR-RC-001 |
| `feature_tags` | BGP4+, 路由选路, MED边界, 反转 |
| `case_title` | 验证MED=0到非零值反转时选路结果正确 |
| `priority` | P2 |
| `preconditions` | DUT1与DUT2/DUT3 IBGP Established |
| `graph_ref` | SP-01 |
| `coverage_goal` | MED边界: MED=0 参与反转验证 |
| `trigger_data` | 初始 MED(A)=0, MED(C)=100; 反转后 MED(A)=100, MED(C)=0 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | DR-10 |
| `scenario_refs` | S02 |
| `scenario_chain_refs` | S02 boundary_reversal_path |
| `action_source_refs` | DR-10 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 初始态: DUT2 MED=0, DUT3 MED=100
    target: DUT2, DUT3
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        prefix: "2001:db8:a00::/64"
        med:
          DUT2: 0
          DUT3: 100
    expected_result: 路由配置成功
    trace_refs:
      - DR-10

  - step_id: STEP-002
    step_name: DUT1验证初始优选MED=0 (DUT2)
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:a00::/64"
    expected_result: 优选DUT2, MED=0
    trace_refs:
      - DR-10

  - step_id: STEP-003
    step_name: 反转MED: DUT2 MED=100, DUT3 MED=0
    target: DUT2, DUT3
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        prefix: "2001:db8:a00::/64"
        med:
          DUT2: 100
          DUT3: 0
    expected_result: MED值交换成功
    trace_refs:
      - DR-10

  - step_id: STEP-004
    step_name: DUT1验证选路反转优选DUT3 MED=0
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        prefix: "2001:db8:a00::/64"
    expected_result: 优选DUT3, MED=0
    trace_refs:
      - DR-10
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 初始MED: DUT2=0, DUT3=100 | DUT2, DUT3 | fw_config_bgp_global | 配置成功 |
| 2 | 验证初始优选DUT2 MED=0 | DUT1 | fw_verify_bgp6_route | 优选DUT2 |
| 3 | 反转MED: DUT2=100, DUT3=0 | DUT2, DUT3 | fw_config_bgp_global | MED交换成功 |
| 4 | 验证选路反转优选DUT3 | DUT1 | fw_verify_bgp6_route | 优选DUT3 |

### expected_results

1. MED=0 与 MED=100 比较正确优选 MED=0
2. MED 值互换后选路结果正确反转
3. MED=0 边界值在选路中与正常值行为一致
