# LC-RT-002 BGP4+ 网络宣告 — 物理用例

> 生成时间: 2026-06-12 | 设计Skill: data-design
> 逻辑用例: LC-RT-002 | 目录: BGP-协议协商-路由通告

---

## PC-RT-DAT-001：标准掩码 /64 网络宣告

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-DAT-001 |
| `logic_case_id` | LC-RT-002 |
| `requirement_ids` | SR-BGP6-NETWORK |
| `feature_tags` | BGP4+, 网络宣告, IPv6 CIDR, 标准掩码 |
| `case_title` | 验证标准 /64 掩码的 IPv6 CIDR 网络宣告成功 |
| `priority` | P1 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 接口 IPv6 直连互通 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 主生命周期: 标准掩码 CRUD 全覆盖 — 宣告 / 验证 / 删除 |
| `trigger_data` | cidr="2001:db8:100::/64", maskType="standard" |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-RT-DAT-001 |
| `scenario_refs` | S-RT-01 |
| `scenario_chain_refs` | S-RT-01 normal_path 步骤1→2→3 |
| `action_source_refs` | S-RT-01-OP1, S-RT-01-OP2, S-RT-01-OP3 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 宣告标准 /64 掩码的 IPv6 CIDR 网络
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_network
      args:
        cidr: "2001:db8:100::/64"
        maskType: "standard"
    expected_result: API 返回 success, 网络宣告成功
    trace_refs:
      - TP-RT-DAT-001
      - S-RT-01-OP1

  - step_id: STEP-002
    step_name: 验证网络宣告在 BGP 路由表中存在
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_network
      args:
        cidr: "2001:db8:100::/64"
    expected_result: BGP 路由表中存在 2001:db8:100::/64, maskType=standard
    trace_refs:
      - TP-RT-DAT-001
      - S-RT-01-OP2

  - step_id: STEP-003
    step_name: 删除网络宣告
    target: DUT1
    atomic_op:
      op_id: fw_delete_bgp6_network
      args:
        cidr: "2001:db8:100::/64"
    expected_result: API 返回 success, 网络宣告已删除
    trace_refs:
      - TP-RT-DAT-001
      - S-RT-01-OP3

  - step_id: STEP-004
    step_name: 验证网络宣告已从路由表中移除
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_network
      args:
        cidr: "2001:db8:100::/64"
    expected_result: BGP 路由表中不存在 2001:db8:100::/64
    trace_refs:
      - TP-RT-DAT-001
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 宣告标准 /64 掩码的 IPv6 CIDR 网络 | DUT1 | fw_config_bgp6_network cidr="2001:db8:100::/64", maskType="standard" | API 返回 success |
| 2 | 验证网络宣告在 BGP 路由表中存在 | DUT1 | fw_verify_bgp6_network cidr="2001:db8:100::/64" | 路由表中存在 /64 |
| 3 | 删除网络宣告 | DUT1 | fw_delete_bgp6_network cidr="2001:db8:100::/64" | API 返回 success |
| 4 | 验证网络宣告已从路由表中移除 | DUT1 | fw_verify_bgp6_network cidr="2001:db8:100::/64" | 路由表中不存在 |

### expected_results

1. 标准 /64 掩码 IPv6 CIDR 能够正常宣告
2. 宣告后 BGP 路由表中可查询到该网络
3. 删除后路由表中不再存在该网络
4. CRUD 全生命周期操作均正常

---

## PC-RT-DAT-002：标准掩码 /48 网络宣告

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-DAT-002 |
| `logic_case_id` | LC-RT-002 |
| `requirement_ids` | SR-BGP6-NETWORK |
| `feature_tags` | BGP4+, 网络宣告, IPv6 CIDR, 标准掩码 |
| `case_title` | 验证标准 /48 掩码的 IPv6 CIDR 网络宣告成功 |
| `priority` | P1 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 接口 IPv6 直连互通 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 主生命周期: 标准掩码 /48 宣告 |
| `trigger_data` | cidr="2001:db8::/48", maskType="standard" |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-RT-DAT-002 |
| `scenario_refs` | S-RT-01 |
| `scenario_chain_refs` | S-RT-01 normal_path 步骤1→2 |
| `action_source_refs` | S-RT-01-OP1, S-RT-01-OP2 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 宣告标准 /48 掩码的 IPv6 CIDR 网络
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_network
      args:
        cidr: "2001:db8::/48"
        maskType: "standard"
    expected_result: API 返回 success, 网络宣告成功
    trace_refs:
      - TP-RT-DAT-002
      - S-RT-01-OP1

  - step_id: STEP-002
    step_name: 验证网络宣告在 BGP 路由表中存在
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_network
      args:
        cidr: "2001:db8::/48"
    expected_result: BGP 路由表中存在 2001:db8::/48
    trace_refs:
      - TP-RT-DAT-002
      - S-RT-01-OP2

  - step_id: STEP-003
    step_name: 清理: 删除网络宣告
    target: DUT1
    atomic_op:
      op_id: fw_delete_bgp6_network
      args:
        cidr: "2001:db8::/48"
    expected_result: API 返回 success
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 宣告标准 /48 掩码的 IPv6 CIDR 网络 | DUT1 | fw_config_bgp6_network cidr="2001:db8::/48", maskType="standard" | API 返回 success |
| 2 | 验证网络宣告在 BGP 路由表中存在 | DUT1 | fw_verify_bgp6_network cidr="2001:db8::/48" | 路由表中存在 /48 |
| 3 | 清理: 删除网络宣告 | DUT1 | fw_delete_bgp6_network cidr="2001:db8::/48" | API 返回 success |

### expected_results

1. 标准 /48 掩码 IPv6 CIDR 能够正常宣告
2. 宣告后 BGP 路由表中可查询到该网络

---

## PC-RT-DAT-003：标准掩码 /32 网络宣告

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-DAT-003 |
| `logic_case_id` | LC-RT-002 |
| `requirement_ids` | SR-BGP6-NETWORK |
| `feature_tags` | BGP4+, 网络宣告, IPv6 CIDR, 标准掩码 |
| `case_title` | 验证标准 /32 掩码的 IPv6 CIDR 网络宣告成功 |
| `priority` | P1 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 接口 IPv6 直连互通 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 主生命周期: 标准掩码 /32 宣告 |
| `trigger_data` | cidr="2001:db8::/32", maskType="standard" |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-RT-DAT-003 |
| `scenario_refs` | S-RT-01 |
| `scenario_chain_refs` | S-RT-01 normal_path 步骤1→2 |
| `action_source_refs` | S-RT-01-OP1, S-RT-01-OP2 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 宣告标准 /32 掩码的 IPv6 CIDR 网络
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_network
      args:
        cidr: "2001:db8::/32"
        maskType: "standard"
    expected_result: API 返回 success, 网络宣告成功
    trace_refs:
      - TP-RT-DAT-003
      - S-RT-01-OP1

  - step_id: STEP-002
    step_name: 验证网络宣告在 BGP 路由表中存在
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_network
      args:
        cidr: "2001:db8::/32"
    expected_result: BGP 路由表中存在 2001:db8::/32
    trace_refs:
      - TP-RT-DAT-003
      - S-RT-01-OP2

  - step_id: STEP-003
    step_name: 清理: 删除网络宣告
    target: DUT1
    atomic_op:
      op_id: fw_delete_bgp6_network
      args:
        cidr: "2001:db8::/32"
    expected_result: API 返回 success
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 宣告标准 /32 掩码的 IPv6 CIDR 网络 | DUT1 | fw_config_bgp6_network cidr="2001:db8::/32", maskType="standard" | API 返回 success |
| 2 | 验证网络宣告在 BGP 路由表中存在 | DUT1 | fw_verify_bgp6_network cidr="2001:db8::/32" | 路由表中存在 /32 |
| 3 | 清理: 删除网络宣告 | DUT1 | fw_delete_bgp6_network cidr="2001:db8::/32" | API 返回 success |

### expected_results

1. 标准 /32 掩码 IPv6 CIDR 能够正常宣告
2. 宣告后 BGP 路由表中可查询到该网络

---

## PC-RT-DAT-004：非标准包含掩码宣告

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-DAT-004 |
| `logic_case_id` | LC-RT-002 |
| `requirement_ids` | SR-BGP6-NETWORK |
| `feature_tags` | BGP4+, 网络宣告, IPv6 CIDR, 非标准包含掩码 |
| `case_title` | 验证非标准包含掩码（/96）的 IPv6 CIDR 网络宣告成功 |
| `priority` | P1 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 接口 IPv6 直连互通 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 主生命周期: 非标准包含掩码宣告 |
| `trigger_data` | cidr="2001:db8:100::/96", maskType="nonstandard_include" |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-RT-DAT-004 |
| `scenario_refs` | S-RT-01 |
| `scenario_chain_refs` | S-RT-01 normal_path 步骤1→2 |
| `action_source_refs` | S-RT-01-OP1, S-RT-01-OP2 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 宣告非标准包含掩码 /96 的 IPv6 CIDR 网络
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_network
      args:
        cidr: "2001:db8:100::/96"
        maskType: "nonstandard_include"
    expected_result: API 返回 success, 网络宣告成功
    trace_refs:
      - TP-RT-DAT-004
      - S-RT-01-OP1

  - step_id: STEP-002
    step_name: 验证网络宣告在 BGP 路由表中存在
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_network
      args:
        cidr: "2001:db8:100::/96"
    expected_result: BGP 路由表中存在 2001:db8:100::/96
    trace_refs:
      - TP-RT-DAT-004
      - S-RT-01-OP2

  - step_id: STEP-003
    step_name: 清理: 删除网络宣告
    target: DUT1
    atomic_op:
      op_id: fw_delete_bgp6_network
      args:
        cidr: "2001:db8:100::/96"
    expected_result: API 返回 success
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 宣告非标准包含掩码 /96 的 IPv6 CIDR 网络 | DUT1 | fw_config_bgp6_network cidr="2001:db8:100::/96", maskType="nonstandard_include" | API 返回 success |
| 2 | 验证网络宣告在 BGP 路由表中存在 | DUT1 | fw_verify_bgp6_network cidr="2001:db8:100::/96" | 路由表中存在 /96 |
| 3 | 清理: 删除网络宣告 | DUT1 | fw_delete_bgp6_network cidr="2001:db8:100::/96" | API 返回 success |

### expected_results

1. 非标准包含掩码（/96）IPv6 CIDR 能够正常宣告
2. 宣告后 BGP 路由表中可查询到该网络

---

## PC-RT-DAT-005：非标准不包含掩码宣告

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-DAT-005 |
| `logic_case_id` | LC-RT-002 |
| `requirement_ids` | SR-BGP6-NETWORK |
| `feature_tags` | BGP4+, 网络宣告, IPv6 CIDR, 非标准不包含掩码 |
| `case_title` | 验证非标准不包含掩码（/80）的 IPv6 CIDR 网络宣告成功 |
| `priority` | P1 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 接口 IPv6 直连互通 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 主生命周期: 非标准不包含掩码宣告 |
| `trigger_data` | cidr="2001:db8:100::/80", maskType="nonstandard_exclude" |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-RT-DAT-005 |
| `scenario_refs` | S-RT-01 |
| `scenario_chain_refs` | S-RT-01 normal_path 步骤1→2 |
| `action_source_refs` | S-RT-01-OP1, S-RT-01-OP2 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 宣告非标准不包含掩码 /80 的 IPv6 CIDR 网络
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_network
      args:
        cidr: "2001:db8:100::/80"
        maskType: "nonstandard_exclude"
    expected_result: API 返回 success, 网络宣告成功
    trace_refs:
      - TP-RT-DAT-005
      - S-RT-01-OP1

  - step_id: STEP-002
    step_name: 验证网络宣告在 BGP 路由表中存在
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_network
      args:
        cidr: "2001:db8:100::/80"
    expected_result: BGP 路由表中存在 2001:db8:100::/80
    trace_refs:
      - TP-RT-DAT-005
      - S-RT-01-OP2

  - step_id: STEP-003
    step_name: 清理: 删除网络宣告
    target: DUT1
    atomic_op:
      op_id: fw_delete_bgp6_network
      args:
        cidr: "2001:db8:100::/80"
    expected_result: API 返回 success
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 宣告非标准不包含掩码 /80 的 IPv6 CIDR 网络 | DUT1 | fw_config_bgp6_network cidr="2001:db8:100::/80", maskType="nonstandard_exclude" | API 返回 success |
| 2 | 验证网络宣告在 BGP 路由表中存在 | DUT1 | fw_verify_bgp6_network cidr="2001:db8:100::/80" | 路由表中存在 /80 |
| 3 | 清理: 删除网络宣告 | DUT1 | fw_delete_bgp6_network cidr="2001:db8:100::/80" | API 返回 success |

### expected_results

1. 非标准不包含掩码（/80）IPv6 CIDR 能够正常宣告
2. 宣告后 BGP 路由表中可查询到该网络

---

## PC-RT-DAT-006：主机路由 /128 宣告

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-DAT-006 |
| `logic_case_id` | LC-RT-002 |
| `requirement_ids` | SR-BGP6-NETWORK |
| `feature_tags` | BGP4+, 网络宣告, IPv6 CIDR, /128 主机路由 |
| `case_title` | 验证 /128 主机路由的 IPv6 CIDR 网络宣告成功 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 接口 IPv6 直连互通 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 边界值: 最大掩码 /128 主机路由宣告 |
| `trigger_data` | cidr="2001:db8:100::1/128", maskType="standard" |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-RT-DAT-006 |
| `scenario_refs` | S-RT-01 |
| `scenario_chain_refs` | S-RT-01 normal_path 步骤1→2 |
| `action_source_refs` | S-RT-01-OP1, S-RT-01-OP2 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 宣告 /128 主机路由的 IPv6 CIDR 网络
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_network
      args:
        cidr: "2001:db8:100::1/128"
        maskType: "standard"
    expected_result: API 返回 success, 主机路由宣告成功
    trace_refs:
      - TP-RT-DAT-006
      - S-RT-01-OP1

  - step_id: STEP-002
    step_name: 验证主机路由在 BGP 路由表中存在
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_network
      args:
        cidr: "2001:db8:100::1/128"
    expected_result: BGP 路由表中存在 2001:db8:100::1/128
    trace_refs:
      - TP-RT-DAT-006
      - S-RT-01-OP2

  - step_id: STEP-003
    step_name: 清理: 删除主机路由宣告
    target: DUT1
    atomic_op:
      op_id: fw_delete_bgp6_network
      args:
        cidr: "2001:db8:100::1/128"
    expected_result: API 返回 success
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 宣告 /128 主机路由的 IPv6 CIDR 网络 | DUT1 | fw_config_bgp6_network cidr="2001:db8:100::1/128", maskType="standard" | API 返回 success |
| 2 | 验证主机路由在 BGP 路由表中存在 | DUT1 | fw_verify_bgp6_network cidr="2001:db8:100::1/128" | 路由表中存在 /128 |
| 3 | 清理: 删除主机路由宣告 | DUT1 | fw_delete_bgp6_network cidr="2001:db8:100::1/128" | API 返回 success |

### expected_results

1. /128 主机路由能够正常宣告
2. 宣告后 BGP 路由表中可查询到该主机路由

---

## PC-RT-DAT-007：极短前缀 /1 宣告 [待确认]

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-DAT-007 |
| `logic_case_id` | LC-RT-002 |
| `requirement_ids` | SR-BGP6-NETWORK |
| `feature_tags` | BGP4+, 网络宣告, IPv6 CIDR, /1 极短前缀 |
| `case_title` | 验证 /1 极短前缀的 IPv6 CIDR 网络宣告行为 [待确认] |
| `priority` | P3 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 接口 IPv6 直连互通 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 边界值: 极短前缀 /1 宣告行为 |
| `trigger_data` | cidr="8000::/1", maskType="standard" |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-RT-DAT-007 |
| `scenario_refs` | S-RT-01 |
| `scenario_chain_refs` | — |
| `action_source_refs` | — |
| `confirmation_gap_refs` | GAP-RT-DAT-007: /1 极短前缀宣告是否被支持 |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 尝试宣告 /1 极短前缀的 IPv6 CIDR 网络
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_network
      args:
        cidr: "8000::/1"
        maskType: "standard"
    expected_result: "[待确认] API 返回 success 或拒绝; 若成功则路由表中应存在 8000::/1"
    trace_refs:
      - TP-RT-DAT-007
    confirmation_gap_refs:
      - GAP-RT-DAT-007

  - step_id: STEP-002
    step_name: 若成功则验证路由存在, 并清理
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_network
      args:
        cidr: "8000::/1"
    expected_result: "[待确认] 若支持则路由存在; 若不支持则路由不存在"
    trace_refs:
      - TP-RT-DAT-007
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 尝试宣告 /1 极短前缀的 IPv6 CIDR 网络 | DUT1 | fw_config_bgp6_network cidr="8000::/1", maskType="standard" | [待确认] 支持或拒绝 |
| 2 | 验证路由存在（若宣告成功） | DUT1 | fw_verify_bgp6_network cidr="8000::/1" | [待确认] |

### expected_results

[待确认] 根据实现不同：
- 场景A: /1 极短前缀被接受并正常宣告
- 场景B: /1 极短前缀被拒绝（需确认最小支持前缀长度）

---

## PC-RT-DAT-008：::/0 默认路由宣告拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-DAT-008 |
| `logic_case_id` | LC-RT-002 |
| `requirement_ids` | SR-BGP6-NETWORK |
| `feature_tags` | BGP4+, 网络宣告, IPv6 CIDR, ::/0 默认路由 |
| `case_title` | 验证 ::/0 默认路由的 IPv6 CIDR 网络宣告被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 接口 IPv6 直连互通 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 异常拒绝: ::/0 默认路由不应通过网络宣告方式注入 |
| `trigger_data` | cidr="::/0", maskType="standard" |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-RT-DAT-008 |
| `scenario_refs` | S-RT-01 |
| `scenario_chain_refs` | S-RT-01 abnormal_path "::/0 拒绝" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 尝试宣告 ::/0 默认路由
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_network
      args:
        cidr: "::/0"
        maskType: "standard"
    expected_result: API 返回错误, 提示 ::/0 不允许通过网络宣告方式注入
    trace_refs:
      - TP-RT-DAT-008

  - step_id: STEP-002
    step_name: 验证 ::/0 未出现在 BGP 路由表中
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_network
      args:
        cidr: "::/0"
    expected_result: BGP 路由表中不存在 ::/0
    trace_refs:
      - TP-RT-DAT-008
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 尝试宣告 ::/0 默认路由 | DUT1 | fw_config_bgp6_network cidr="::/0", maskType="standard" | API 返回错误 |
| 2 | 验证 ::/0 未出现在 BGP 路由表中 | DUT1 | fw_verify_bgp6_network cidr="::/0" | 路由表中不存在 |

### expected_results

1. ::/0 默认路由被 API 拒绝
2. 返回明确错误提示
3. 路由表中不会出现 ::/0

---

## PC-RT-DAT-009：ff00::/64 多播地址宣告拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-DAT-009 |
| `logic_case_id` | LC-RT-002 |
| `requirement_ids` | SR-BGP6-NETWORK |
| `feature_tags` | BGP4+, 网络宣告, IPv6 CIDR, 多播地址 |
| `case_title` | 验证多播地址 ff00::/64 的 IPv6 CIDR 网络宣告被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 接口 IPv6 直连互通 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 异常拒绝: 多播地址不应作为单播路由宣告 |
| `trigger_data` | cidr="ff00::/64", maskType="standard" |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-RT-DAT-009 |
| `scenario_refs` | S-RT-01 |
| `scenario_chain_refs` | S-RT-01 abnormal_path "多播地址拒绝" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 尝试宣告多播地址 ff00::/64
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_network
      args:
        cidr: "ff00::/64"
        maskType: "standard"
    expected_result: API 返回错误, 提示多播地址不允许宣告
    trace_refs:
      - TP-RT-DAT-009

  - step_id: STEP-002
    step_name: 验证 ff00::/64 未出现在 BGP 路由表中
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_network
      args:
        cidr: "ff00::/64"
    expected_result: BGP 路由表中不存在 ff00::/64
    trace_refs:
      - TP-RT-DAT-009
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 尝试宣告多播地址 ff00::/64 | DUT1 | fw_config_bgp6_network cidr="ff00::/64", maskType="standard" | API 返回错误 |
| 2 | 验证 ff00::/64 未出现在 BGP 路由表中 | DUT1 | fw_verify_bgp6_network cidr="ff00::/64" | 路由表中不存在 |

### expected_results

1. 多播地址 ff00::/64 被 API 拒绝
2. 返回明确错误提示
3. 路由表中不会出现多播地址

---

## PC-RT-DAT-010：::1/128 环回地址宣告拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-DAT-010 |
| `logic_case_id` | LC-RT-002 |
| `requirement_ids` | SR-BGP6-NETWORK |
| `feature_tags` | BGP4+, 网络宣告, IPv6 CIDR, 环回地址 |
| `case_title` | 验证环回地址 ::1/128 的 IPv6 CIDR 网络宣告被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 接口 IPv6 直连互通 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 异常拒绝: 环回地址不应宣告 |
| `trigger_data` | cidr="::1/128", maskType="standard" |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-RT-DAT-010 |
| `scenario_refs` | S-RT-01 |
| `scenario_chain_refs` | S-RT-01 abnormal_path "环回地址拒绝" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 尝试宣告环回地址 ::1/128
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_network
      args:
        cidr: "::1/128"
        maskType: "standard"
    expected_result: API 返回错误, 提示环回地址不允许宣告
    trace_refs:
      - TP-RT-DAT-010

  - step_id: STEP-002
    step_name: 验证 ::1/128 未出现在 BGP 路由表中
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_network
      args:
        cidr: "::1/128"
    expected_result: BGP 路由表中不存在 ::1/128
    trace_refs:
      - TP-RT-DAT-010
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 尝试宣告环回地址 ::1/128 | DUT1 | fw_config_bgp6_network cidr="::1/128", maskType="standard" | API 返回错误 |
| 2 | 验证 ::1/128 未出现在 BGP 路由表中 | DUT1 | fw_verify_bgp6_network cidr="::1/128" | 路由表中不存在 |

### expected_results

1. 环回地址 ::1/128 被 API 拒绝
2. 返回明确错误提示
3. 路由表中不会出现环回地址

---

## PC-RT-DAT-011：未指定地址 :: 宣告拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-DAT-011 |
| `logic_case_id` | LC-RT-002 |
| `requirement_ids` | SR-BGP6-NETWORK |
| `feature_tags` | BGP4+, 网络宣告, IPv6 CIDR, :: 未指定地址 |
| `case_title` | 验证未指定地址 :: 的 IPv6 CIDR 网络宣告被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 接口 IPv6 直连互通 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 异常拒绝: 未指定地址不应宣告 |
| `trigger_data` | cidr="::", maskType="standard" |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-RT-DAT-011 |
| `scenario_refs` | S-RT-01 |
| `scenario_chain_refs` | S-RT-01 abnormal_path ":: 拒绝" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 尝试宣告未指定地址 ::
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_network
      args:
        cidr: "::"
        maskType: "standard"
    expected_result: API 返回错误, 提示未指定地址不允许宣告
    trace_refs:
      - TP-RT-DAT-011

  - step_id: STEP-002
    step_name: 验证 :: 未出现在 BGP 路由表中
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_network
      args:
        cidr: "::"
    expected_result: BGP 路由表中不存在 ::
    trace_refs:
      - TP-RT-DAT-011
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 尝试宣告未指定地址 :: | DUT1 | fw_config_bgp6_network cidr="::", maskType="standard" | API 返回错误 |
| 2 | 验证 :: 未出现在 BGP 路由表中 | DUT1 | fw_verify_bgp6_network cidr="::" | 路由表中不存在 |

### expected_results

1. 未指定地址 :: 被 API 拒绝
2. 返回明确错误提示

---

## PC-RT-DAT-012：IPv4 CIDR 网络宣告拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-DAT-012 |
| `logic_case_id` | LC-RT-002 |
| `requirement_ids` | SR-BGP6-NETWORK |
| `feature_tags` | BGP4+, 网络宣告, IPv6 CIDR, IPv4 拒绝 |
| `case_title` | 验证 IPv4 CIDR 地址被 BGP4+ 网络宣告拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 接口 IPv6 直连互通 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 异常拒绝: IPv4 地址不应通过 BGP4+ 网络宣告接口注入 |
| `trigger_data` | cidr="192.168.1.0/24", maskType="standard" |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-RT-DAT-012 |
| `scenario_refs` | S-RT-01 |
| `scenario_chain_refs` | S-RT-01 abnormal_path "IPv4 CIDR 拒绝" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 尝试通过 BGP4+ 网络宣告接口宣告 IPv4 CIDR
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_network
      args:
        cidr: "192.168.1.0/24"
        maskType: "standard"
    expected_result: API 返回错误, 提示仅支持 IPv6 CIDR
    trace_refs:
      - TP-RT-DAT-012

  - step_id: STEP-002
    step_name: 验证 IPv4 CIDR 未出现在 BGP 路由表中
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_network
      args:
        cidr: "192.168.1.0/24"
    expected_result: BGP 路由表中不存在该 IPv4 CIDR
    trace_refs:
      - TP-RT-DAT-012
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 尝试通过 BGP4+ 网络宣告接口宣告 IPv4 CIDR | DUT1 | fw_config_bgp6_network cidr="192.168.1.0/24", maskType="standard" | API 返回错误 |
| 2 | 验证 IPv4 CIDR 未出现在 BGP 路由表中 | DUT1 | fw_verify_bgp6_network cidr="192.168.1.0/24" | 路由表中不存在 |

### expected_results

1. IPv4 CIDR 被 BGP4+ 接口拒绝
2. 返回明确错误提示（仅支持 IPv6）

---

## PC-RT-DAT-013：/129 前缀长度溢出拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-DAT-013 |
| `logic_case_id` | LC-RT-002 |
| `requirement_ids` | SR-BGP6-NETWORK |
| `feature_tags` | BGP4+, 网络宣告, IPv6 CIDR, /129 溢出 |
| `case_title` | 验证 /129 前缀长度溢出的 IPv6 CIDR 网络宣告被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 接口 IPv6 直连互通 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 边界拒绝: IPv6 前缀长度上限 128, /129 为非法值 |
| `trigger_data` | cidr="2001:db8:100::/129", maskType="standard" |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-RT-DAT-013 |
| `scenario_refs` | S-RT-01 |
| `scenario_chain_refs` | S-RT-01 abnormal_path "/129 溢出拒绝" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 尝试宣告前缀长度 /129 的 IPv6 CIDR
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_network
      args:
        cidr: "2001:db8:100::/129"
        maskType: "standard"
    expected_result: API 返回错误, 提示前缀长度超出范围(0-128)
    trace_refs:
      - TP-RT-DAT-013

  - step_id: STEP-002
    step_name: 验证非法 CIDR 未出现在 BGP 路由表中
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_network
      args:
        cidr: "2001:db8:100::/129"
    expected_result: BGP 路由表中不存在该 CIDR
    trace_refs:
      - TP-RT-DAT-013
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 尝试宣告前缀长度 /129 的 IPv6 CIDR | DUT1 | fw_config_bgp6_network cidr="2001:db8:100::/129", maskType="standard" | API 返回错误 |
| 2 | 验证非法 CIDR 未出现在 BGP 路由表中 | DUT1 | fw_verify_bgp6_network cidr="2001:db8:100::/129" | 路由表中不存在 |

### expected_results

1. /129 前缀长度溢出被 API 拒绝
2. 返回明确错误提示（前缀长度 0-128）

---

## PC-RT-DAT-014：缺少前缀（仅斜杠）宣告拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-DAT-014 |
| `logic_case_id` | LC-RT-002 |
| `requirement_ids` | SR-BGP6-NETWORK |
| `feature_tags` | BGP4+, 网络宣告, IPv6 CIDR, 缺前缀 |
| `case_title` | 验证缺少前缀长度的 IPv6 CIDR 网络宣告被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 接口 IPv6 直连互通 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 格式拒绝: 缺少前缀长度的 CIDR 不合法 |
| `trigger_data` | cidr="2001:db8:100::", maskType="standard" |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-RT-DAT-014 |
| `scenario_refs` | S-RT-01 |
| `scenario_chain_refs` | S-RT-01 abnormal_path "缺前缀拒绝" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 尝试宣告缺少前缀长度的 IPv6 地址
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_network
      args:
        cidr: "2001:db8:100::"
        maskType: "standard"
    expected_result: API 返回错误, 提示缺少前缀长度(/N)
    trace_refs:
      - TP-RT-DAT-014

  - step_id: STEP-002
    step_name: 验证该地址未出现在 BGP 路由表中
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_network
      args:
        cidr: "2001:db8:100::"
    expected_result: BGP 路由表中不存在该地址
    trace_refs:
      - TP-RT-DAT-014
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 尝试宣告缺少前缀长度的 IPv6 地址 | DUT1 | fw_config_bgp6_network cidr="2001:db8:100::", maskType="standard" | API 返回错误 |
| 2 | 验证该地址未出现在 BGP 路由表中 | DUT1 | fw_verify_bgp6_network cidr="2001:db8:100::" | 路由表中不存在 |

### expected_results

1. 缺少前缀长度的 CIDR 被 API 拒绝
2. 返回明确错误提示

---

## PC-RT-DAT-015：NaN 前缀宣告 [待确认]

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-DAT-015 |
| `logic_case_id` | LC-RT-002 |
| `requirement_ids` | SR-BGP6-NETWORK |
| `feature_tags` | BGP4+, 网络宣告, IPv6 CIDR, NaN 前缀 |
| `case_title` | 验证 NaN 格式前缀的 IPv6 CIDR 网络宣告行为 [待确认] |
| `priority` | P3 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 接口 IPv6 直连互通 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 格式拒绝: NaN 非数字前缀 |
| `trigger_data` | cidr="NaN", maskType="standard" |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-RT-DAT-015 |
| `scenario_refs` | S-RT-01 |
| `scenario_chain_refs` | — |
| `action_source_refs` | — |
| `confirmation_gap_refs` | GAP-RT-DAT-015: NaN 前缀处理行为待确认 |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 尝试宣告 NaN 格式前缀
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_network
      args:
        cidr: "NaN"
        maskType: "standard"
    expected_result: "[待确认] API 返回格式错误或系统异常; 不应导致服务崩溃"
    trace_refs:
      - TP-RT-DAT-015
    confirmation_gap_refs:
      - GAP-RT-DAT-015

  - step_id: STEP-002
    step_name: 验证系统状态正常
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_network
      args:
        cidr: "2001:db8:100::/64"
    expected_result: "[待确认] 已有合法宣告不受影响, 系统状态正常"
    trace_refs:
      - TP-RT-DAT-015
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 尝试宣告 NaN 格式前缀 | DUT1 | fw_config_bgp6_network cidr="NaN", maskType="standard" | [待确认] 格式错误或异常 |
| 2 | 验证系统状态正常，已有宣告不受影响 | DUT1 | fw_verify_bgp6_network cidr="2001:db8:100::/64" | [待确认] |

### expected_results

[待确认] 根据实现不同：
- 场景A: NaN 被格式校验拦截，返回明确错误
- 场景B: NaN 导致系统异常（需确认是否有容错机制）

---

## PC-RT-DAT-016：空掩码类型宣告

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-DAT-016 |
| `logic_case_id` | LC-RT-002 |
| `requirement_ids` | SR-BGP6-NETWORK |
| `feature_tags` | BGP4+, 网络宣告, IPv6 CIDR, 空掩码类型 |
| `case_title` | 验证缺少掩码类型参数的 IPv6 CIDR 网络宣告被拒绝 |
| `priority` | P3 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 接口 IPv6 直连互通 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 必填参数校验: maskType 为空 |
| `trigger_data` | cidr="2001:db8:100::/64", maskType="" |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-RT-DAT-016 |
| `scenario_refs` | S-RT-01 |
| `scenario_chain_refs` | S-RT-01 abnormal_path "空掩码类型拒绝" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 尝试宣告缺少掩码类型参数的 IPv6 CIDR
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_network
      args:
        cidr: "2001:db8:100::/64"
        maskType: ""
    expected_result: API 返回错误, 提示 maskType 不能为空
    trace_refs:
      - TP-RT-DAT-016

  - step_id: STEP-002
    step_name: 验证未产生意外宣告
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_network
      args:
        cidr: "2001:db8:100::/64"
    expected_result: BGP 路由表中不存在 2001:db8:100::/64
    trace_refs:
      - TP-RT-DAT-016
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 尝试宣告缺少掩码类型参数的 IPv6 CIDR | DUT1 | fw_config_bgp6_network cidr="2001:db8:100::/64", maskType="" | API 返回错误 |
| 2 | 验证未产生意外宣告 | DUT1 | fw_verify_bgp6_network cidr="2001:db8:100::/64" | 路由表中不存在 |

### expected_results

1. 空掩码类型被 API 拒绝
2. 返回明确错误提示
3. 不会产生意外的默认行为宣告

---

## PC-RT-DAT-017：link-local 地址宣告 [待确认]

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-RT-DAT-017 |
| `logic_case_id` | LC-RT-002 |
| `requirement_ids` | SR-BGP6-NETWORK |
| `feature_tags` | BGP4+, 网络宣告, IPv6 CIDR, link-local |
| `case_title` | 验证 link-local 地址 fe80::/10 的 IPv6 CIDR 网络宣告行为 [待确认] |
| `priority` | P3 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 接口 IPv6 直连互通 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 边界行为: link-local 地址是否允许宣告 |
| `trigger_data` | cidr="fe80::/64", maskType="standard" |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-RT-DAT-017 |
| `scenario_refs` | S-RT-01 |
| `scenario_chain_refs` | — |
| `action_source_refs` | — |
| `confirmation_gap_refs` | GAP-RT-DAT-017: link-local 地址宣告行为待确认 |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 尝试宣告 link-local 地址 fe80::/64
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_network
      args:
        cidr: "fe80::/64"
        maskType: "standard"
    expected_result: "[待确认] API 返回 success 或拒绝; link-local 地址通常不应在 BGP 中宣告"
    trace_refs:
      - TP-RT-DAT-017
    confirmation_gap_refs:
      - GAP-RT-DAT-017

  - step_id: STEP-002
    step_name: 验证路由表状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_network
      args:
        cidr: "fe80::/64"
    expected_result: "[待确认] 若支持则路由存在; 若拒绝则路由不存在"
    trace_refs:
      - TP-RT-DAT-017
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 尝试宣告 link-local 地址 fe80::/64 | DUT1 | fw_config_bgp6_network cidr="fe80::/64", maskType="standard" | [待确认] 支持或拒绝 |
| 2 | 验证路由表状态 | DUT1 | fw_verify_bgp6_network cidr="fe80::/64" | [待确认] |

### expected_results

[待确认] 根据实现不同：
- 场景A: link-local 地址被拒绝（BGP 最佳实践）
- 场景B: link-local 地址被接受并宣告
