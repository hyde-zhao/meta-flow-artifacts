# BGP4+ 全局使能与禁用 — 物理用例

> 生成时间: 2026-06-12 | 设计Skill: data-design
> 逻辑用例: LC-BPG-001 | 目录: BGP-配置管理-BPG配置

---

## PC-BPG-DAT-001：使能→禁用→恢复 (典型值)

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-001 |
| `logic_case_id` | LC-BPG-001 |
| `requirement_ids` | SR-002, SR-003, SR-004 |
| `feature_tags` | BGP4+, 全局配置, 使能控制 |
| `case_title` | 验证BGP4+全局使能→禁用→恢复典型流程且三层配置一致 |
| `priority` | P1 |
| `preconditions` | DUT1 接口 IPv4/IPv6 已配置；BGP4+ 全局未使能；ASN=100 未占用 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 主生命周期: 使能→禁用→恢复 (valid-typical 全覆盖) |
| `trigger_data` | enable=true→false→true, asn=100, routerId=1.1.1.1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-001, TP-M1-002 |
| `scenario_refs` | S01 |
| `scenario_chain_refs` | S01 normal_path 步骤1→2→3 |
| `action_source_refs` | S01-OP1, S01-OP2, S01-OP3 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: PUT使能BGP4+全局配置 (enable=true, ASN=100, RouterID=1.1.1.1)
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        enable: true
        asNumber: 100
        routerId: "1.1.1.1"
    expected_result: API返回success, BGP4+ 全局使能生效
    trace_refs:
      - TP-M1-001

  - step_id: STEP-002
    step_name: GET验证API层配置一致
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: enable=true, asNumber=100, routerId="1.1.1.1"
    trace_refs:
      - TP-M1-001

  - step_id: STEP-003
    step_name: PUT禁用BGP4+全局配置 (enable=false)
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        enable: false
        asNumber: 100
        routerId: "1.1.1.1"
    expected_result: API返回success, BGP4+ 全局禁用, 邻居断开
    trace_refs:
      - TP-M1-002

  - step_id: STEP-004
    step_name: GET验证禁用状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: enable=false, 邻居状态为空/Idle
    trace_refs:
      - TP-M1-002

  - step_id: STEP-005
    step_name: PUT重新使能BGP4+全局配置 (enable=true)
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        enable: true
        asNumber: 100
        routerId: "1.1.1.1"
    expected_result: API返回success, BGP4+ 恢复使能
    trace_refs:
      - TP-M1-001

  - step_id: STEP-006
    step_name: GET验证重新使能后配置一致
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: enable=true, asNumber=100, routerId="1.1.1.1", 邻居可重新建立
    trace_refs:
      - TP-M1-001
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | PUT使能BGP4+全局配置 | DUT1 | fw_config_bgp_global enable=true, asNumber=100, routerId="1.1.1.1" | API返回success |
| 2 | GET验证API层配置一致 | DUT1 | fw_verify_bgp_global | enable=true, asNumber=100, routerId="1.1.1.1" |
| 3 | PUT禁用BGP4+全局配置 | DUT1 | fw_config_bgp_global enable=false, asNumber=100, routerId="1.1.1.1" | API返回success, 邻居断开 |
| 4 | GET验证禁用状态 | DUT1 | fw_verify_bgp_global | enable=false |
| 5 | PUT重新使能BGP4+全局配置 | DUT1 | fw_config_bgp_global enable=true, asNumber=100, routerId="1.1.1.1" | API返回success |
| 6 | GET验证重新使能后配置一致 | DUT1 | fw_verify_bgp_global | enable=true, asNumber=100, routerId="1.1.1.1" |

### expected_results

1. BGP4+ 全局使能成功，API/etcd/FRR 三层配置一致
2. BGP4+ 全局禁用成功，已建立的邻居断开
3. 重新使能后配置恢复，邻居可重新建立
4. 整个过程中无配置残留或状态异常

---

## PC-BPG-DAT-002：最小 ASN=1 使能

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-002 |
| `logic_case_id` | LC-BPG-001 |
| `requirement_ids` | SR-003 |
| `feature_tags` | BGP4+, 全局配置, 边界值 |
| `case_title` | 验证BGP4+全局使能使用最小ASN=1成功 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 未使能 |
| `graph_ref` | SP-02 |
| `coverage_goal` | 边界守卫: ASN min-on=1 合法接收 |
| `trigger_data` | enable=true, asn=1, routerId=1.1.1.1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-003 |
| `scenario_refs` | S01 |
| `scenario_chain_refs` | S01 boundary_path "ASN=1" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: PUT使能BGP4+全局配置 (ASN=1)
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        enable: true
        asNumber: 1
        routerId: "1.1.1.1"
    expected_result: API返回success, 最小ASN=1被接受
    trace_refs:
      - TP-M1-003

  - step_id: STEP-002
    step_name: GET验证配置生效
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: enable=true, asNumber=1, routerId="1.1.1.1"
    trace_refs:
      - TP-M1-003

  - step_id: STEP-003
    step_name: 清理: 禁用BGP4+
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        enable: false
        asNumber: 1
        routerId: "1.1.1.1"
    expected_result: API返回success
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | PUT使能BGP4+全局配置 (ASN=1) | DUT1 | fw_config_bgp_global enable=true, asNumber=1, routerId="1.1.1.1" | API返回success |
| 2 | GET验证配置生效 | DUT1 | fw_verify_bgp_global | asNumber=1 生效 |
| 3 | 清理: 禁用BGP4+ | DUT1 | fw_config_bgp_global enable=false | API返回success |

### expected_results

1. 最小 ASN=1 被正常接受，BGP4+ 全局使能成功
2. 配置查询返回 asNumber=1

---

## PC-BPG-DAT-003：最大 ASN=4294967295 使能

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-003 |
| `logic_case_id` | LC-BPG-001 |
| `requirement_ids` | SR-003 |
| `feature_tags` | BGP4+, 全局配置, 边界值 |
| `case_title` | 验证BGP4+全局使能使用最大ASN=4294967295成功 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 未使能 |
| `graph_ref` | SP-03 |
| `coverage_goal` | 边界守卫: ASN max-on=4294967295 合法接收 |
| `trigger_data` | enable=true, asn=4294967295, routerId=1.1.1.1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-004 |
| `scenario_refs` | S01 |
| `scenario_chain_refs` | S01 boundary_path "ASN=max" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: PUT使能BGP4+全局配置 (ASN=4294967295)
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        enable: true
        asNumber: 4294967295
        routerId: "1.1.1.1"
    expected_result: API返回success, 最大ASN被接受
    trace_refs:
      - TP-M1-004

  - step_id: STEP-002
    step_name: GET验证配置生效
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: enable=true, asNumber=4294967295, routerId="1.1.1.1"
    trace_refs:
      - TP-M1-004

  - step_id: STEP-003
    step_name: 清理: 禁用BGP4+
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        enable: false
        asNumber: 4294967295
        routerId: "1.1.1.1"
    expected_result: API返回success
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | PUT使能BGP4+全局配置 (ASN=max) | DUT1 | fw_config_bgp_global enable=true, asNumber=4294967295, routerId="1.1.1.1" | API返回success |
| 2 | GET验证配置生效 | DUT1 | fw_verify_bgp_global | asNumber=4294967295 生效 |
| 3 | 清理: 禁用BGP4+ | DUT1 | fw_config_bgp_global enable=false | API返回success |

### expected_results

1. 最大 ASN=4294967295 被正常接受，BGP4+ 全局使能成功
2. 配置查询返回 asNumber=4294967295

---

## PC-BPG-DAT-004：ASN=0 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-004 |
| `logic_case_id` | LC-BPG-001 |
| `requirement_ids` | SR-003 |
| `feature_tags` | BGP4+, 全局配置, 边界值, 异常路径 |
| `case_title` | 验证BGP4+全局配置ASN=0被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 未使能 |
| `graph_ref` | SP-04 |
| `coverage_goal` | 边界守卫: ASN min-off=0 拒绝 |
| `trigger_data` | asn=0, routerId=1.1.1.1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-005 |
| `scenario_refs` | S01 |
| `scenario_chain_refs` | S01 abnormal_path "ASN=0" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: PUT配置BGP4+全局 (ASN=0)
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        enable: true
        asNumber: 0
        routerId: "1.1.1.1"
    expected_result: API返回错误, 提示ASN不能为0
    trace_refs:
      - TP-M1-005

  - step_id: STEP-002
    step_name: GET验证配置未生效
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: 全局配置未变更, BGP4+未使能或保持原值
    trace_refs:
      - TP-M1-005
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | PUT配置BGP4+全局 (ASN=0) | DUT1 | fw_config_bgp_global enable=true, asNumber=0, routerId="1.1.1.1" | API返回错误 |
| 2 | GET验证配置未生效 | DUT1 | fw_verify_bgp_global | 配置未变更 |

### expected_results

1. ASN=0 被 API 拒绝，返回明确错误信息
2. 已有配置不受影响，配置未变更

---

## PC-BPG-DAT-005：ASN=4294967296 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-005 |
| `logic_case_id` | LC-BPG-001 |
| `requirement_ids` | SR-003 |
| `feature_tags` | BGP4+, 全局配置, 边界值, 异常路径 |
| `case_title` | 验证BGP4+全局配置ASN=4294967296(超出uint32)被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 未使能 |
| `graph_ref` | SP-05 |
| `coverage_goal` | 边界守卫: ASN max-off=4294967296 拒绝 |
| `trigger_data` | asn=4294967296, routerId=1.1.1.1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-005 |
| `scenario_refs` | S01 |
| `scenario_chain_refs` | S01 abnormal_path "ASN超出范围" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: PUT配置BGP4+全局 (ASN=4294967296)
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        enable: true
        asNumber: 4294967296
        routerId: "1.1.1.1"
    expected_result: API返回错误, 提示ASN超出有效范围(1~4294967295)
    trace_refs:
      - TP-M1-005

  - step_id: STEP-002
    step_name: GET验证配置未生效
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: 配置未变更
    trace_refs:
      - TP-M1-005
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | PUT配置BGP4+全局 (ASN=超出范围) | DUT1 | fw_config_bgp_global enable=true, asNumber=4294967296, routerId="1.1.1.1" | API返回错误 |
| 2 | GET验证配置未生效 | DUT1 | fw_verify_bgp_global | 配置未变更 |

### expected_results

1. ASN=4294967296 被 API 拒绝，返回范围错误信息
2. 已有配置不受影响

---

## PC-BPG-DAT-006：Router ID=0.0.0.0 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-006 |
| `logic_case_id` | LC-BPG-001 |
| `requirement_ids` | SR-004 |
| `feature_tags` | BGP4+, 全局配置, 边界值, 异常路径 |
| `case_title` | 验证BGP4+全局配置Router ID=0.0.0.0被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 未使能 |
| `graph_ref` | SP-06 |
| `coverage_goal` | 边界守卫: Router ID 禁止值 0.0.0.0 拒绝 |
| `trigger_data` | asn=100, routerId=0.0.0.0 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-006 |
| `scenario_refs` | S01 |
| `scenario_chain_refs` | S01 abnormal_path "RouterID=0.0.0.0" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: PUT配置BGP4+全局 (RouterID=0.0.0.0)
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        enable: true
        asNumber: 100
        routerId: "0.0.0.0"
    expected_result: API返回错误, 提示Router ID不能为0.0.0.0
    trace_refs:
      - TP-M1-006

  - step_id: STEP-002
    step_name: GET验证配置未生效
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: 配置未变更
    trace_refs:
      - TP-M1-006
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | PUT配置BGP4+全局 (RouterID=0.0.0.0) | DUT1 | fw_config_bgp_global enable=true, asNumber=100, routerId="0.0.0.0" | API返回错误 |
| 2 | GET验证配置未生效 | DUT1 | fw_verify_bgp_global | 配置未变更 |

### expected_results

1. Router ID=0.0.0.0 被 API 拒绝
2. 已有配置不受影响

---

## PC-BPG-DAT-007：Router ID=224.0.0.1 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-007 |
| `logic_case_id` | LC-BPG-001 |
| `requirement_ids` | SR-004 |
| `feature_tags` | BGP4+, 全局配置, 异常路径 |
| `case_title` | 验证BGP4+全局配置Router ID=224.0.0.1(组播地址)被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 未使能 |
| `graph_ref` | SP-07 |
| `coverage_goal` | 异常守卫: 组播地址 Router ID 拒绝 |
| `trigger_data` | asn=100, routerId=224.0.0.1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | — |
| `scenario_refs` | S01 |
| `scenario_chain_refs` | S01 abnormal_path "RouterID=组播" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: PUT配置BGP4+全局 (RouterID=224.0.0.1)
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        enable: true
        asNumber: 100
        routerId: "224.0.0.1"
    expected_result: API返回错误, 提示Router ID不能使用组播地址
    trace_refs:
      - TP-M1-006

  - step_id: STEP-002
    step_name: GET验证配置未生效
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: 配置未变更
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | PUT配置BGP4+全局 (RouterID=组播) | DUT1 | fw_config_bgp_global enable=true, asNumber=100, routerId="224.0.0.1" | API返回错误 |
| 2 | GET验证配置未生效 | DUT1 | fw_verify_bgp_global | 配置未变更 |

### expected_results

1. Router ID=224.0.0.1 (组播地址) 被 API 拒绝
2. 已有配置不受影响

---

## PC-BPG-DAT-008：ASN=-1 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-008 |
| `logic_case_id` | LC-BPG-001 |
| `requirement_ids` | SR-003 |
| `feature_tags` | BGP4+, 全局配置, 异常路径 |
| `case_title` | 验证BGP4+全局配置ASN=-1(负数)被拒绝 |
| `priority` | P3 |
| `preconditions` | DUT1 BGP4+ 未使能 |
| `graph_ref` | SP-08 |
| `coverage_goal` | 异常守卫: 负数 ASN 拒绝 |
| `trigger_data` | asn=-1, routerId=1.1.1.1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | — |
| `scenario_refs` | S01 |
| `scenario_chain_refs` | S01 abnormal_path "ASN负数" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: PUT配置BGP4+全局 (ASN=-1)
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        enable: true
        asNumber: -1
        routerId: "1.1.1.1"
    expected_result: API返回错误, 提示ASN值非法
    trace_refs:
      - TP-M1-005

  - step_id: STEP-002
    step_name: GET验证配置未生效
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: 配置未变更
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | PUT配置BGP4+全局 (ASN=-1) | DUT1 | fw_config_bgp_global enable=true, asNumber=-1, routerId="1.1.1.1" | API返回错误 |
| 2 | GET验证配置未生效 | DUT1 | fw_verify_bgp_global | 配置未变更 |

### expected_results

1. ASN=-1 (负数) 被 API 拒绝
2. 已有配置不受影响

---

## PC-BPG-DAT-009：enable=null 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-009 |
| `logic_case_id` | LC-BPG-001 |
| `requirement_ids` | SR-002 |
| `feature_tags` | BGP4+, 全局配置, 异常路径 |
| `case_title` | 验证BGP4+全局配置enable字段为null时被拒绝或取默认值 |
| `priority` | P3 |
| `preconditions` | DUT1 BGP4+ 未使能 |
| `graph_ref` | SP-09 |
| `coverage_goal` | 异常守卫: enable 空值行为 |
| `trigger_data` | enable=null, asn=100, routerId=1.1.1.1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | — |
| `scenario_refs` | S01 |
| `scenario_chain_refs` | S01 abnormal_path "enable为空" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: PUT配置BGP4+全局 (enable=null)
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        enable: null
        asNumber: 100
        routerId: "1.1.1.1"
    expected_result: API返回错误(必填校验) 或 默认取false保持未使能
    trace_refs:
      - TP-M1-001

  - step_id: STEP-002
    step_name: GET验证状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: BGP4+未使能(enable=false 或 配置未变更)
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | PUT配置BGP4+全局 (enable=null) | DUT1 | fw_config_bgp_global enable=null, asNumber=100, routerId="1.1.1.1" | 返回错误或取默认值 |
| 2 | GET验证状态 | DUT1 | fw_verify_bgp_global | BGP4+未使能 |

### expected_results

1. enable=null 被拒绝（必填校验）或默认取 false 保持未使能
2. 系统状态不受影响

---

## PC-BPG-DAT-010：Router ID 格式错误拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-010 |
| `logic_case_id` | LC-BPG-001 |
| `requirement_ids` | SR-004 |
| `feature_tags` | BGP4+, 全局配置, 异常路径 |
| `case_title` | 验证BGP4+全局配置Router ID为非IPv4格式字符串时被拒绝 |
| `priority` | P3 |
| `preconditions` | DUT1 BGP4+ 未使能 |
| `graph_ref` | SP-10 |
| `coverage_goal` | 异常守卫: Router ID 格式错误拒绝 |
| `trigger_data` | asn=100, routerId="abc" |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | — |
| `scenario_refs` | S01 |
| `scenario_chain_refs` | S01 abnormal_path "RouterID格式错误" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: PUT配置BGP4+全局 (RouterID="abc")
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        enable: true
        asNumber: 100
        routerId: "abc"
    expected_result: API返回错误, 提示Router ID格式非法
    trace_refs:
      - TP-M1-006

  - step_id: STEP-002
    step_name: GET验证配置未生效
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: 配置未变更
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | PUT配置BGP4+全局 (RouterID格式错误) | DUT1 | fw_config_bgp_global enable=true, asNumber=100, routerId="abc" | API返回错误 |
| 2 | GET验证配置未生效 | DUT1 | fw_verify_bgp_global | 配置未变更 |

### expected_results

1. Router ID 非 IPv4 格式字符串被 API 拒绝
2. 已有配置不受影响

---

## PC-BPG-DAT-011：Router ID=127.0.0.1 [待确认]

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-011 |
| `logic_case_id` | LC-BPG-001 |
| `requirement_ids` | SR-004 |
| `feature_tags` | BGP4+, 全局配置, 边界值 |
| `case_title` | 验证BGP4+全局配置Router ID=127.0.0.1(回环地址)的行为 [待确认] |
| `priority` | P3 |
| `preconditions` | DUT1 BGP4+ 未使能 |
| `graph_ref` | SP-11 |
| `coverage_goal` | 边界探查: 回环地址是否可作 Router ID |
| `trigger_data` | asn=100, routerId=127.0.0.1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | — |
| `scenario_refs` | S01 |
| `scenario_chain_refs` | S01 abnormal_path "RouterID=127.0.0.1" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | LC-BPG-001-GAP-RID-LOOPBACK |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: PUT配置BGP4+全局 (RouterID=127.0.0.1)
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp_global
      args:
        enable: true
        asNumber: 100
        routerId: "127.0.0.1"
    expected_result: "[待确认] API可能接受(回环地址未明确禁止) 或 拒绝(视为无效Router ID)"
    trace_refs: []
    confirmation_gap_refs:
      - LC-BPG-001-GAP-RID-LOOPBACK

  - step_id: STEP-002
    step_name: GET验证配置结果
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: "[待确认] 若接受则 routerId=127.0.0.1; 若拒绝则配置未变更"
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | PUT配置BGP4+全局 (RouterID=127.0.0.1) | DUT1 | fw_config_bgp_global enable=true, asNumber=100, routerId="127.0.0.1" | [待确认] |
| 2 | GET验证配置结果 | DUT1 | fw_verify_bgp_global | [待确认] |

### expected_results

1. [待确认] 回环地址 127.0.0.1 是否允许作为 Router ID，取决于实现
2. 场景A: 接受为合法 Router ID
3. 场景B: 拒绝，提示 Router ID 不能为回环地址

---
