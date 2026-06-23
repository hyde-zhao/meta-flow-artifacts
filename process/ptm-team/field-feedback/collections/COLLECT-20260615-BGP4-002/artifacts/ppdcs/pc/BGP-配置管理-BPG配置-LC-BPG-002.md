# BGP4+ 优先级配置 — 物理用例

> 生成时间: 2026-06-12 | 设计Skill: data-design
> 逻辑用例: LC-BPG-002 | 目录: BGP-配置管理-BPG配置

---

## PC-BPG-DAT-012：默认优先级值 (20/200)

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-012 |
| `logic_case_id` | LC-BPG-002 |
| `requirement_ids` | SR-005 |
| `feature_tags` | BGP4+, 优先级, 管理距离 |
| `case_title` | 验证BGP4+优先级使用默认值EBGP=20/IBGP=200配置成功 |
| `priority` | P1 |
| `preconditions` | DUT1 BGP4+ 全局已使能 (ASN=100, RouterID=1.1.1.1) |
| `graph_ref` | SP-01 |
| `coverage_goal` | 典型值: 默认 EBGP/IBGP 管理距离配置与验证 |
| `trigger_data` | ebgp6Distance=20, ibgp6Distance=200 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-008, TP-M1-009 |
| `scenario_refs` | S02 |
| `scenario_chain_refs` | S02 normal_path 步骤1→2 |
| `action_source_refs` | S02-OP1 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: PUT配置BGP4+优先级为默认值
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_distance
      args:
        ebgp6Distance: 20
        ibgp6Distance: 200
    expected_result: API返回success, 默认值配置生效
    trace_refs:
      - TP-M1-008
      - TP-M1-009

  - step_id: STEP-002
    step_name: GET验证优先级配置
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_distance
      args: {}
    expected_result: ebgp6Distance=20, ibgp6Distance=200
    trace_refs:
      - TP-M1-008
      - TP-M1-009
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | PUT配置BGP4+优先级为默认值 | DUT1 | fw_config_bgp6_distance ebgp6Distance=20, ibgp6Distance=200 | API返回success |
| 2 | GET验证优先级配置 | DUT1 | fw_verify_bgp6_distance | ebgp6Distance=20, ibgp6Distance=200 |

### expected_results

1. EBGP/IBGP 默认管理距离配置成功
2. 查询返回配置值与写入一致

---

## PC-BPG-DAT-013：EBGP 最小边界 =1

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-013 |
| `logic_case_id` | LC-BPG-002 |
| `requirement_ids` | SR-005 |
| `feature_tags` | BGP4+, 优先级, 边界值 |
| `case_title` | 验证BGP4+ EBGP优先级使用最小边界值=1成功 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 全局已使能 (ASN=100, RouterID=1.1.1.1) |
| `graph_ref` | SP-02 |
| `coverage_goal` | 边界守卫: EBGP min-on=1 合法接收 |
| `trigger_data` | ebgp6Distance=1, ibgp6Distance=200 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-008 |
| `scenario_refs` | S02 |
| `scenario_chain_refs` | S02 boundary_path "EBGP=1" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: PUT配置EBGP优先级为最小值1
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_distance
      args:
        ebgp6Distance: 1
        ibgp6Distance: 200
    expected_result: API返回success, 最小EBGP优先级被接受
    trace_refs:
      - TP-M1-008

  - step_id: STEP-002
    step_name: GET验证优先级配置
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_distance
      args: {}
    expected_result: ebgp6Distance=1, ibgp6Distance=200
    trace_refs:
      - TP-M1-008

  - step_id: STEP-003
    step_name: 恢复默认值
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_distance
      args:
        ebgp6Distance: 20
        ibgp6Distance: 200
    expected_result: API返回success
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | PUT配置EBGP优先级为1 | DUT1 | fw_config_bgp6_distance ebgp6Distance=1, ibgp6Distance=200 | API返回success |
| 2 | GET验证优先级配置 | DUT1 | fw_verify_bgp6_distance | ebgp6Distance=1 |
| 3 | 恢复默认值 | DUT1 | fw_config_bgp6_distance ebgp6Distance=20 | API返回success |

### expected_results

1. EBGP 最小优先级=1 被正常接受
2. 配置查询返回 ebgp6Distance=1

---

## PC-BPG-DAT-014：EBGP 最大边界 =255

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-014 |
| `logic_case_id` | LC-BPG-002 |
| `requirement_ids` | SR-005 |
| `feature_tags` | BGP4+, 优先级, 边界值 |
| `case_title` | 验证BGP4+ EBGP优先级使用最大边界值=255成功 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 全局已使能 (ASN=100, RouterID=1.1.1.1) |
| `graph_ref` | SP-03 |
| `coverage_goal` | 边界守卫: EBGP max-on=255 合法接收 |
| `trigger_data` | ebgp6Distance=255, ibgp6Distance=200 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-008 |
| `scenario_refs` | S02 |
| `scenario_chain_refs` | S02 boundary_path "EBGP=255" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: PUT配置EBGP优先级为最大值255
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_distance
      args:
        ebgp6Distance: 255
        ibgp6Distance: 200
    expected_result: API返回success, 最大EBGP优先级被接受
    trace_refs:
      - TP-M1-008

  - step_id: STEP-002
    step_name: GET验证优先级配置
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_distance
      args: {}
    expected_result: ebgp6Distance=255, ibgp6Distance=200
    trace_refs:
      - TP-M1-008

  - step_id: STEP-003
    step_name: 恢复默认值
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_distance
      args:
        ebgp6Distance: 20
        ibgp6Distance: 200
    expected_result: API返回success
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | PUT配置EBGP优先级为255 | DUT1 | fw_config_bgp6_distance ebgp6Distance=255, ibgp6Distance=200 | API返回success |
| 2 | GET验证优先级配置 | DUT1 | fw_verify_bgp6_distance | ebgp6Distance=255 |
| 3 | 恢复默认值 | DUT1 | fw_config_bgp6_distance ebgp6Distance=20 | API返回success |

### expected_results

1. EBGP 最大优先级=255 被正常接受
2. 配置查询返回 ebgp6Distance=255

---

## PC-BPG-DAT-015：IBGP 最小边界 =1

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-015 |
| `logic_case_id` | LC-BPG-002 |
| `requirement_ids` | SR-005 |
| `feature_tags` | BGP4+, 优先级, 边界值 |
| `case_title` | 验证BGP4+ IBGP优先级使用最小边界值=1成功 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 全局已使能 (ASN=100, RouterID=1.1.1.1) |
| `graph_ref` | SP-04 |
| `coverage_goal` | 边界守卫: IBGP min-on=1 合法接收 |
| `trigger_data` | ebgp6Distance=20, ibgp6Distance=1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-009 |
| `scenario_refs` | S02 |
| `scenario_chain_refs` | S02 boundary_path "IBGP=1" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: PUT配置IBGP优先级为最小值1
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_distance
      args:
        ebgp6Distance: 20
        ibgp6Distance: 1
    expected_result: API返回success, 最小IBGP优先级被接受
    trace_refs:
      - TP-M1-009

  - step_id: STEP-002
    step_name: GET验证优先级配置
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_distance
      args: {}
    expected_result: ebgp6Distance=20, ibgp6Distance=1
    trace_refs:
      - TP-M1-009

  - step_id: STEP-003
    step_name: 恢复默认值
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_distance
      args:
        ebgp6Distance: 20
        ibgp6Distance: 200
    expected_result: API返回success
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | PUT配置IBGP优先级为1 | DUT1 | fw_config_bgp6_distance ebgp6Distance=20, ibgp6Distance=1 | API返回success |
| 2 | GET验证优先级配置 | DUT1 | fw_verify_bgp6_distance | ibgp6Distance=1 |
| 3 | 恢复默认值 | DUT1 | fw_config_bgp6_distance ibgp6Distance=200 | API返回success |

### expected_results

1. IBGP 最小优先级=1 被正常接受
2. 配置查询返回 ibgp6Distance=1

---

## PC-BPG-DAT-016：IBGP 最大边界 =255

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-016 |
| `logic_case_id` | LC-BPG-002 |
| `requirement_ids` | SR-005 |
| `feature_tags` | BGP4+, 优先级, 边界值 |
| `case_title` | 验证BGP4+ IBGP优先级使用最大边界值=255成功 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 全局已使能 (ASN=100, RouterID=1.1.1.1) |
| `graph_ref` | SP-05 |
| `coverage_goal` | 边界守卫: IBGP max-on=255 合法接收 |
| `trigger_data` | ebgp6Distance=20, ibgp6Distance=255 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-009 |
| `scenario_refs` | S02 |
| `scenario_chain_refs` | S02 boundary_path "IBGP=255" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: PUT配置IBGP优先级为最大值255
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_distance
      args:
        ebgp6Distance: 20
        ibgp6Distance: 255
    expected_result: API返回success, 最大IBGP优先级被接受
    trace_refs:
      - TP-M1-009

  - step_id: STEP-002
    step_name: GET验证优先级配置
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_distance
      args: {}
    expected_result: ebgp6Distance=20, ibgp6Distance=255
    trace_refs:
      - TP-M1-009

  - step_id: STEP-003
    step_name: 恢复默认值
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_distance
      args:
        ebgp6Distance: 20
        ibgp6Distance: 200
    expected_result: API返回success
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | PUT配置IBGP优先级为255 | DUT1 | fw_config_bgp6_distance ebgp6Distance=20, ibgp6Distance=255 | API返回success |
| 2 | GET验证优先级配置 | DUT1 | fw_verify_bgp6_distance | ibgp6Distance=255 |
| 3 | 恢复默认值 | DUT1 | fw_config_bgp6_distance ibgp6Distance=200 | API返回success |

### expected_results

1. IBGP 最大优先级=255 被正常接受
2. 配置查询返回 ibgp6Distance=255

---

## PC-BPG-DAT-017：EBGP=0 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-017 |
| `logic_case_id` | LC-BPG-002 |
| `requirement_ids` | SR-005 |
| `feature_tags` | BGP4+, 优先级, 边界值, 异常路径 |
| `case_title` | 验证BGP4+ EBGP优先级=0被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 全局已使能 (ASN=100, RouterID=1.1.1.1) |
| `graph_ref` | SP-06 |
| `coverage_goal` | 边界守卫: EBGP min-off=0 拒绝 |
| `trigger_data` | ebgp6Distance=0, ibgp6Distance=200 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-008 |
| `scenario_refs` | S02 |
| `scenario_chain_refs` | S02 abnormal_path "EBGP=0" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: PUT配置EBGP优先级为0
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_distance
      args:
        ebgp6Distance: 0
        ibgp6Distance: 200
    expected_result: API返回错误, 提示EBGP优先级范围1~255
    trace_refs:
      - TP-M1-008

  - step_id: STEP-002
    step_name: GET验证配置未变更
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_distance
      args: {}
    expected_result: ebgp6Distance保持原值(非0)
    trace_refs:
      - TP-M1-008
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | PUT配置EBGP优先级为0 | DUT1 | fw_config_bgp6_distance ebgp6Distance=0, ibgp6Distance=200 | API返回错误 |
| 2 | GET验证配置未变更 | DUT1 | fw_verify_bgp6_distance | ebgp6Distance保持不变 |

### expected_results

1. EBGP=0 被 API 拒绝，返回明确错误信息
2. 已有配置不受影响

---

## PC-BPG-DAT-018：EBGP=256 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-018 |
| `logic_case_id` | LC-BPG-002 |
| `requirement_ids` | SR-005 |
| `feature_tags` | BGP4+, 优先级, 边界值, 异常路径 |
| `case_title` | 验证BGP4+ EBGP优先级=256(超出uint8)被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 全局已使能 (ASN=100, RouterID=1.1.1.1) |
| `graph_ref` | SP-07 |
| `coverage_goal` | 边界守卫: EBGP max-off=256 拒绝 |
| `trigger_data` | ebgp6Distance=256, ibgp6Distance=200 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-008 |
| `scenario_refs` | S02 |
| `scenario_chain_refs` | S02 abnormal_path "EBGP=256" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: PUT配置EBGP优先级为256
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_distance
      args:
        ebgp6Distance: 256
        ibgp6Distance: 200
    expected_result: API返回错误, 提示EBGP优先级超出范围
    trace_refs:
      - TP-M1-008

  - step_id: STEP-002
    step_name: GET验证配置未变更
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_distance
      args: {}
    expected_result: ebgp6Distance保持原值
    trace_refs:
      - TP-M1-008
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | PUT配置EBGP优先级为256 | DUT1 | fw_config_bgp6_distance ebgp6Distance=256, ibgp6Distance=200 | API返回错误 |
| 2 | GET验证配置未变更 | DUT1 | fw_verify_bgp6_distance | ebgp6Distance保持不变 |

### expected_results

1. EBGP=256 被 API 拒绝，返回范围错误信息
2. 已有配置不受影响

---

## PC-BPG-DAT-019：IBGP=0 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-019 |
| `logic_case_id` | LC-BPG-002 |
| `requirement_ids` | SR-005 |
| `feature_tags` | BGP4+, 优先级, 边界值, 异常路径 |
| `case_title` | 验证BGP4+ IBGP优先级=0被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 全局已使能 (ASN=100, RouterID=1.1.1.1) |
| `graph_ref` | SP-08 |
| `coverage_goal` | 边界守卫: IBGP min-off=0 拒绝 |
| `trigger_data` | ebgp6Distance=20, ibgp6Distance=0 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-009 |
| `scenario_refs` | S02 |
| `scenario_chain_refs` | S02 abnormal_path "IBGP=0" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: PUT配置IBGP优先级为0
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_distance
      args:
        ebgp6Distance: 20
        ibgp6Distance: 0
    expected_result: API返回错误, 提示IBGP优先级范围1~255
    trace_refs:
      - TP-M1-009

  - step_id: STEP-002
    step_name: GET验证配置未变更
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_distance
      args: {}
    expected_result: ibgp6Distance保持原值(非0)
    trace_refs:
      - TP-M1-009
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | PUT配置IBGP优先级为0 | DUT1 | fw_config_bgp6_distance ebgp6Distance=20, ibgp6Distance=0 | API返回错误 |
| 2 | GET验证配置未变更 | DUT1 | fw_verify_bgp6_distance | ibgp6Distance保持不变 |

### expected_results

1. IBGP=0 被 API 拒绝，返回明确错误信息
2. 已有配置不受影响

---

## PC-BPG-DAT-020：IBGP=256 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-020 |
| `logic_case_id` | LC-BPG-002 |
| `requirement_ids` | SR-005 |
| `feature_tags` | BGP4+, 优先级, 边界值, 异常路径 |
| `case_title` | 验证BGP4+ IBGP优先级=256(超出uint8)被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 全局已使能 (ASN=100, RouterID=1.1.1.1) |
| `graph_ref` | SP-09 |
| `coverage_goal` | 边界守卫: IBGP max-off=256 拒绝 |
| `trigger_data` | ebgp6Distance=20, ibgp6Distance=256 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-009 |
| `scenario_refs` | S02 |
| `scenario_chain_refs` | S02 abnormal_path "IBGP=256" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: PUT配置IBGP优先级为256
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_distance
      args:
        ebgp6Distance: 20
        ibgp6Distance: 256
    expected_result: API返回错误, 提示IBGP优先级超出范围
    trace_refs:
      - TP-M1-009

  - step_id: STEP-002
    step_name: GET验证配置未变更
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_distance
      args: {}
    expected_result: ibgp6Distance保持原值
    trace_refs:
      - TP-M1-009
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | PUT配置IBGP优先级为256 | DUT1 | fw_config_bgp6_distance ebgp6Distance=20, ibgp6Distance=256 | API返回错误 |
| 2 | GET验证配置未变更 | DUT1 | fw_verify_bgp6_distance | ibgp6Distance保持不变 |

### expected_results

1. IBGP=256 被 API 拒绝，返回范围错误信息
2. 已有配置不受影响

---

## PC-BPG-DAT-021：EBGP=-1 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-021 |
| `logic_case_id` | LC-BPG-002 |
| `requirement_ids` | SR-005 |
| `feature_tags` | BGP4+, 优先级, 异常路径 |
| `case_title` | 验证BGP4+ EBGP优先级=-1(负数)被拒绝 |
| `priority` | P3 |
| `preconditions` | DUT1 BGP4+ 全局已使能 (ASN=100, RouterID=1.1.1.1) |
| `graph_ref` | SP-10 |
| `coverage_goal` | 异常守卫: 负数 EBGP 拒绝 |
| `trigger_data` | ebgp6Distance=-1, ibgp6Distance=200 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | — |
| `scenario_refs` | S02 |
| `scenario_chain_refs` | S02 abnormal_path "EBGP负数" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: PUT配置EBGP优先级为-1
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_distance
      args:
        ebgp6Distance: -1
        ibgp6Distance: 200
    expected_result: API返回错误, 提示值非法
    trace_refs:
      - TP-M1-008

  - step_id: STEP-002
    step_name: GET验证配置未变更
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_distance
      args: {}
    expected_result: ebgp6Distance保持原值
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | PUT配置EBGP优先级为-1 | DUT1 | fw_config_bgp6_distance ebgp6Distance=-1, ibgp6Distance=200 | API返回错误 |
| 2 | GET验证配置未变更 | DUT1 | fw_verify_bgp6_distance | ebgp6Distance保持不变 |

### expected_results

1. EBGP=-1 (负数) 被 API 拒绝
2. 已有配置不受影响

---

## PC-BPG-DAT-022：IBGP=-1 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-022 |
| `logic_case_id` | LC-BPG-002 |
| `requirement_ids` | SR-005 |
| `feature_tags` | BGP4+, 优先级, 异常路径 |
| `case_title` | 验证BGP4+ IBGP优先级=-1(负数)被拒绝 |
| `priority` | P3 |
| `preconditions` | DUT1 BGP4+ 全局已使能 (ASN=100, RouterID=1.1.1.1) |
| `graph_ref` | SP-11 |
| `coverage_goal` | 异常守卫: 负数 IBGP 拒绝 |
| `trigger_data` | ebgp6Distance=20, ibgp6Distance=-1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | — |
| `scenario_refs` | S02 |
| `scenario_chain_refs` | S02 abnormal_path "IBGP负数" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: PUT配置IBGP优先级为-1
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_distance
      args:
        ebgp6Distance: 20
        ibgp6Distance: -1
    expected_result: API返回错误, 提示值非法
    trace_refs:
      - TP-M1-009

  - step_id: STEP-002
    step_name: GET验证配置未变更
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_distance
      args: {}
    expected_result: ibgp6Distance保持原值
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | PUT配置IBGP优先级为-1 | DUT1 | fw_config_bgp6_distance ebgp6Distance=20, ibgp6Distance=-1 | API返回错误 |
| 2 | GET验证配置未变更 | DUT1 | fw_verify_bgp6_distance | ibgp6Distance保持不变 |

### expected_results

1. IBGP=-1 (负数) 被 API 拒绝
2. 已有配置不受影响

---

## PC-BPG-DAT-023：EBGP=null 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-DAT-023 |
| `logic_case_id` | LC-BPG-002 |
| `requirement_ids` | SR-005 |
| `feature_tags` | BGP4+, 优先级, 异常路径 |
| `case_title` | 验证BGP4+ EBGP优先级=null时被拒绝或取默认值 |
| `priority` | P3 |
| `preconditions` | DUT1 BGP4+ 全局已使能 (ASN=100, RouterID=1.1.1.1) |
| `graph_ref` | SP-12 |
| `coverage_goal` | 异常守卫: EBGP 空值行为 |
| `trigger_data` | ebgp6Distance=null, ibgp6Distance=200 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | — |
| `scenario_refs` | S02 |
| `scenario_chain_refs` | S02 abnormal_path "EBGP为空" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: PUT配置EBGP优先级为null
    target: DUT1
    atomic_op:
      op_id: fw_config_bgp6_distance
      args:
        ebgp6Distance: null
        ibgp6Distance: 200
    expected_result: API返回错误(必填校验) 或 默认取20
    trace_refs:
      - TP-M1-008

  - step_id: STEP-002
    step_name: GET验证状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_distance
      args: {}
    expected_result: ebgp6Distance保持有效值
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | PUT配置EBGP优先级为null | DUT1 | fw_config_bgp6_distance ebgp6Distance=null, ibgp6Distance=200 | 返回错误或取默认值20 |
| 2 | GET验证状态 | DUT1 | fw_verify_bgp6_distance | ebgp6Distance保持有效值 |

### expected_results

1. EBGP=null 被拒绝或默认取 20
2. 已有配置不受影响

---
