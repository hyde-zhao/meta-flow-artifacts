# LC-DFX-001 BGP4+ 日志与可维护性 — 物理用例

> 生成时间: 2026-06-12 | 设计Skill: data-design
> 逻辑用例: LC-DFX-001 | 目录: BGP-管理维护-DFX

---

## PC-DFX-DAT-001：邻居变化日志查询

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-DFX-DAT-001 |
| `logic_case_id` | LC-DFX-001 |
| `requirement_ids` | SR-BGP6-DFX |
| `feature_tags` | BGP4+, 日志查询, 邻居变化, 可维护性 |
| `case_title` | 验证邻居变化类型日志查询成功返回记录 |
| `priority` | P1 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 已有邻居建立和断开事件发生; 日志系统正常运行 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 主生命周期: 日志类型 neighbor_change 查询成功 |
| `trigger_data` | type="neighbor_change", page=1, size=10 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-DFX-DAT-001 |
| `scenario_refs` | S-DFX-01 |
| `scenario_chain_refs` | S-DFX-01 normal_path 步骤1→2 |
| `action_source_refs` | S-DFX-01-OP1, S-DFX-01-OP2 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 执行邻居变化日志查询
    target: DUT1
    atomic_op:
      op_id: fw_capture_bgp6_log
      args:
        type: "neighbor_change"
        page: 1
        size: 10
    expected_result: API 返回 success, 返回邻居变化日志列表, total>0
    trace_refs:
      - TP-DFX-DAT-001
      - S-DFX-01-OP1

  - step_id: STEP-002
    step_name: 验证日志内容符合邻居变化类型
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_log
      args:
        type: "neighbor_change"
    expected_result: 日志条目 type 均为 neighbor_change, 包含时间戳、邻居地址、状态变化详情
    trace_refs:
      - TP-DFX-DAT-001
      - S-DFX-01-OP2
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 执行邻居变化日志查询 | DUT1 | fw_capture_bgp6_log type="neighbor_change", page=1, size=10 | API 返回 success, 日志列表非空 |
| 2 | 验证日志内容符合邻居变化类型 | DUT1 | fw_verify_bgp6_log type="neighbor_change" | 日志类型正确, 内容完整 |

### expected_results

1. 邻居变化日志查询成功返回记录
2. 日志条目包含必要字段（时间戳、邻居地址、事件类型、状态变化）
3. 分页参数生效

---

## PC-DFX-DAT-002：路由添加日志查询

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-DFX-DAT-002 |
| `logic_case_id` | LC-DFX-001 |
| `requirement_ids` | SR-BGP6-DFX |
| `feature_tags` | BGP4+, 日志查询, 路由添加, 可维护性 |
| `case_title` | 验证路由添加类型日志查询成功返回记录 |
| `priority` | P1 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 已有路由添加事件发生; 日志系统正常运行 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 主生命周期: 日志类型 route_add 查询成功 |
| `trigger_data` | type="route_add", page=1, size=10 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-DFX-DAT-002 |
| `scenario_refs` | S-DFX-01 |
| `scenario_chain_refs` | S-DFX-01 normal_path 步骤1→2 |
| `action_source_refs` | S-DFX-01-OP1, S-DFX-01-OP2 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 执行路由添加日志查询
    target: DUT1
    atomic_op:
      op_id: fw_capture_bgp6_log
      args:
        type: "route_add"
        page: 1
        size: 10
    expected_result: API 返回 success, 返回路由添加日志列表, total>0
    trace_refs:
      - TP-DFX-DAT-002
      - S-DFX-01-OP1

  - step_id: STEP-002
    step_name: 验证日志内容符合路由添加类型
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_log
      args:
        type: "route_add"
    expected_result: 日志条目 type 均为 route_add, 包含时间戳、路由前缀、下一跳、AS Path
    trace_refs:
      - TP-DFX-DAT-002
      - S-DFX-01-OP2
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 执行路由添加日志查询 | DUT1 | fw_capture_bgp6_log type="route_add", page=1, size=10 | API 返回 success, 日志列表非空 |
| 2 | 验证日志内容符合路由添加类型 | DUT1 | fw_verify_bgp6_log type="route_add" | 日志类型正确, 包含路由详细信息 |

### expected_results

1. 路由添加日志查询成功返回记录
2. 日志条目包含必要字段（时间戳、路由前缀、下一跳、AS Path）
3. 分页参数生效

---

## PC-DFX-DAT-003：路由删除日志查询

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-DFX-DAT-003 |
| `logic_case_id` | LC-DFX-001 |
| `requirement_ids` | SR-BGP6-DFX |
| `feature_tags` | BGP4+, 日志查询, 路由删除, 可维护性 |
| `case_title` | 验证路由删除类型日志查询成功返回记录 |
| `priority` | P1 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 已有路由删除事件发生; 日志系统正常运行 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 主生命周期: 日志类型 route_delete 查询成功 |
| `trigger_data` | type="route_delete", page=1, size=10 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-DFX-DAT-003 |
| `scenario_refs` | S-DFX-01 |
| `scenario_chain_refs` | S-DFX-01 normal_path 步骤1→2 |
| `action_source_refs` | S-DFX-01-OP1, S-DFX-01-OP2 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 执行路由删除日志查询
    target: DUT1
    atomic_op:
      op_id: fw_capture_bgp6_log
      args:
        type: "route_delete"
        page: 1
        size: 10
    expected_result: API 返回 success, 返回路由删除日志列表, total>0
    trace_refs:
      - TP-DFX-DAT-003
      - S-DFX-01-OP1

  - step_id: STEP-002
    step_name: 验证日志内容符合路由删除类型
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_log
      args:
        type: "route_delete"
    expected_result: 日志条目 type 均为 route_delete, 包含时间戳、被删除路由前缀、撤销原因
    trace_refs:
      - TP-DFX-DAT-003
      - S-DFX-01-OP2
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 执行路由删除日志查询 | DUT1 | fw_capture_bgp6_log type="route_delete", page=1, size=10 | API 返回 success, 日志列表非空 |
| 2 | 验证日志内容符合路由删除类型 | DUT1 | fw_verify_bgp6_log type="route_delete" | 日志类型正确, 包含撤销详情 |

### expected_results

1. 路由删除日志查询成功返回记录
2. 日志条目包含必要字段（时间戳、路由前缀、撤销原因）
3. 分页参数生效

---

## PC-DFX-DAT-004：认证日志查询

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-DFX-DAT-004 |
| `logic_case_id` | LC-DFX-001 |
| `requirement_ids` | SR-BGP6-DFX |
| `feature_tags` | BGP4+, 日志查询, 认证, 可维护性 |
| `case_title` | 验证认证类型日志查询成功返回记录 |
| `priority` | P1 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 已有 MD5 认证成功/失败事件发生; 日志系统正常运行 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 主生命周期: 日志类型 auth 查询成功 |
| `trigger_data` | type="auth", page=1, size=10 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-DFX-DAT-004 |
| `scenario_refs` | S-DFX-01 |
| `scenario_chain_refs` | S-DFX-01 normal_path 步骤1→2 |
| `action_source_refs` | S-DFX-01-OP1, S-DFX-01-OP2 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 执行认证日志查询
    target: DUT1
    atomic_op:
      op_id: fw_capture_bgp6_log
      args:
        type: "auth"
        page: 1
        size: 10
    expected_result: API 返回 success, 返回认证日志列表, total>0
    trace_refs:
      - TP-DFX-DAT-004
      - S-DFX-01-OP1

  - step_id: STEP-002
    step_name: 验证日志内容符合认证类型
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_log
      args:
        type: "auth"
    expected_result: 日志条目 type 均为 auth, 包含时间戳、邻居地址、认证结果(成功/失败)
    trace_refs:
      - TP-DFX-DAT-004
      - S-DFX-01-OP2
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 执行认证日志查询 | DUT1 | fw_capture_bgp6_log type="auth", page=1, size=10 | API 返回 success, 日志列表非空 |
| 2 | 验证日志内容符合认证类型 | DUT1 | fw_verify_bgp6_log type="auth" | 日志类型正确, 包含认证结果 |

### expected_results

1. 认证日志查询成功返回记录
2. 日志条目包含必要字段（时间戳、邻居地址、认证结果）

---

## PC-DFX-DAT-005：分页 size=1 最小边界

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-DFX-DAT-005 |
| `logic_case_id` | LC-DFX-001 |
| `requirement_ids` | SR-BGP6-DFX |
| `feature_tags` | BGP4+, 日志查询, 分页, 边界值, 可维护性 |
| `case_title` | 验证分页 size=1 最小边界值查询成功返回单条记录 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 日志系统存在多条邻居变化记录; 日志系统正常运行 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 边界值: size 最小值 1, 返回恰好 1 条记录 |
| `trigger_data` | type="neighbor_change", page=1, size=1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-DFX-DAT-005 |
| `scenario_refs` | S-DFX-01 |
| `scenario_chain_refs` | S-DFX-01 normal_path 步骤1→2 |
| `action_source_refs` | S-DFX-01-OP1 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 执行 size=1 分页查询
    target: DUT1
    atomic_op:
      op_id: fw_capture_bgp6_log
      args:
        type: "neighbor_change"
        page: 1
        size: 1
    expected_result: API 返回 success, 返回恰好 1 条日志记录, total>=1
    trace_refs:
      - TP-DFX-DAT-005

  - step_id: STEP-002
    step_name: 验证分页元数据正确
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_log
      args:
        type: "neighbor_change"
    expected_result: 返回记录数 = 1, total 值 >= 1, page=1
    trace_refs:
      - TP-DFX-DAT-005
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 执行 size=1 分页查询 | DUT1 | fw_capture_bgp6_log type="neighbor_change", page=1, size=1 | API 返回 success, 1 条记录 |
| 2 | 验证分页元数据正确 | DUT1 | fw_verify_bgp6_log type="neighbor_change" | 记录数 = 1, total >= 1 |

### expected_results

1. size=1 分页查询正常返回
2. 返回恰好 1 条记录
3. 分页元数据（total, page, size）正确

---

## PC-DFX-DAT-006：分页 page=100 最后一页

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-DFX-DAT-006 |
| `logic_case_id` | LC-DFX-001 |
| `requirement_ids` | SR-BGP6-DFX |
| `feature_tags` | BGP4+, 日志查询, 分页, 边界值, 可维护性 |
| `case_title` | 验证分页 page=100 查询最后一页行为正确 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 日志系统存在足够记录使最后一页可能非满页; 日志系统正常运行 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 边界值: 较大 page 值, 验证最后一页可能不满 |
| `trigger_data` | type="neighbor_change", page=100, size=10 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-DFX-DAT-006 |
| `scenario_refs` | S-DFX-01 |
| `scenario_chain_refs` | S-DFX-01 normal_path 步骤1→2 |
| `action_source_refs` | S-DFX-01-OP1 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 执行 page=100 分页查询
    target: DUT1
    atomic_op:
      op_id: fw_capture_bgp6_log
      args:
        type: "neighbor_change"
        page: 100
        size: 10
    expected_result: API 返回 success; 若有足够数据则返回记录; 若超出范围则返回空列表
    trace_refs:
      - TP-DFX-DAT-006

  - step_id: STEP-002
    step_name: 验证返回结果合理性
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_log
      args:
        type: "neighbor_change"
    expected_result: 返回记录数 <= size; page 信息正确
    trace_refs:
      - TP-DFX-DAT-006
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 执行 page=100 分页查询 | DUT1 | fw_capture_bgp6_log type="neighbor_change", page=100, size=10 | API 返回 success |
| 2 | 验证返回结果合理性 | DUT1 | fw_verify_bgp6_log type="neighbor_change" | 记录数 <= size, 无异常 |

### expected_results

1. page=100 查询不产生错误
2. 若超出数据范围则返回空列表
3. 若在范围内正常返回，且可能不满页

---

## PC-DFX-DAT-007：分页 page=101 超范围

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-DFX-DAT-007 |
| `logic_case_id` | LC-DFX-001 |
| `requirement_ids` | SR-BGP6-DFX |
| `feature_tags` | BGP4+, 日志查询, 分页, 超范围, 可维护性 |
| `case_title` | 验证分页 page=101 超出数据范围时返回空列表 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 日志记录总页数 < 101; 日志系统正常运行 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 超范围: page 超出总页数时应返回空列表 |
| `trigger_data` | type="neighbor_change", page=101, size=10 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-DFX-DAT-007 |
| `scenario_refs` | S-DFX-01 |
| `scenario_chain_refs` | S-DFX-01 abnormal_path "超范围分页" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 执行 page=101 超出范围分页查询
    target: DUT1
    atomic_op:
      op_id: fw_capture_bgp6_log
      args:
        type: "neighbor_change"
        page: 101
        size: 10
    expected_result: API 返回 success, 返回空列表 [], total 显示实际总记录数
    trace_refs:
      - TP-DFX-DAT-007

  - step_id: STEP-002
    step_name: 验证分页元数据正确
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_log
      args:
        type: "neighbor_change"
    expected_result: 返回记录数 = 0, total 值正常, 无错误
    trace_refs:
      - TP-DFX-DAT-007
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 执行 page=101 超出范围分页查询 | DUT1 | fw_capture_bgp6_log type="neighbor_change", page=101, size=10 | API 返回 success, 空列表 |
| 2 | 验证分页元数据正确 | DUT1 | fw_verify_bgp6_log type="neighbor_change" | 记录数 = 0, total 正常 |

### expected_results

1. 超出总页数的查询返回空列表，不应报错
2. total 仍显示实际总记录数
3. 无异常或崩溃

---

## PC-DFX-DAT-008：分页 page=99999 溢出

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-DFX-DAT-008 |
| `logic_case_id` | LC-DFX-001 |
| `requirement_ids` | SR-BGP6-DFX |
| `feature_tags` | BGP4+, 日志查询, 分页, 溢出, 可维护性 |
| `case_title` | 验证分页 page=99999 极大值溢出查询不导致异常 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 日志记录总页数远小于 99999; 日志系统正常运行 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 溢出边界: page 极大值是否导致整数溢出 |
| `trigger_data` | type="neighbor_change", page=99999, size=10 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-DFX-DAT-008 |
| `scenario_refs` | S-DFX-01 |
| `scenario_chain_refs` | S-DFX-01 abnormal_path "page 溢出" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 执行 page=99999 极大值分页查询
    target: DUT1
    atomic_op:
      op_id: fw_capture_bgp6_log
      args:
        type: "neighbor_change"
        page: 99999
        size: 10
    expected_result: API 返回 success, 返回空列表或参数被截断, 不导致系统异常
    trace_refs:
      - TP-DFX-DAT-008

  - step_id: STEP-002
    step_name: 验证系统状态正常
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_log
      args:
        type: "neighbor_change"
    expected_result: 日志查询功能正常, 无 crash 或 OOM
    trace_refs:
      - TP-DFX-DAT-008
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 执行 page=99999 极大值分页查询 | DUT1 | fw_capture_bgp6_log type="neighbor_change", page=99999, size=10 | API 返回 success, 空列表 |
| 2 | 验证系统状态正常 | DUT1 | fw_verify_bgp6_log type="neighbor_change" | 功能正常, 无崩溃 |

### expected_results

1. 极大 page 值不会导致整数溢出或系统异常
2. 返回空列表，total 正常
3. 系统日志查询功能不受影响

---

## PC-DFX-DAT-009：分页 page=0 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-DFX-DAT-009 |
| `logic_case_id` | LC-DFX-001 |
| `requirement_ids` | SR-BGP6-DFX |
| `feature_tags` | BGP4+, 日志查询, 分页, 非法值, 可维护性 |
| `case_title` | 验证分页 page=0 非法值被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 日志系统正常运行 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 非法值: page=0 不在合法范围 [1, N] |
| `trigger_data` | type="neighbor_change", page=0, size=10 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-DFX-DAT-009 |
| `scenario_refs` | S-DFX-01 |
| `scenario_chain_refs` | S-DFX-01 abnormal_path "page=0 拒绝" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 执行 page=0 非法分页查询
    target: DUT1
    atomic_op:
      op_id: fw_capture_bgp6_log
      args:
        type: "neighbor_change"
        page: 0
        size: 10
    expected_result: API 返回错误, 提示 page 必须 >= 1
    trace_refs:
      - TP-DFX-DAT-009

  - step_id: STEP-002
    step_name: 验证未返回意外数据
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_log
      args:
        type: "neighbor_change"
    expected_result: 查询被拒绝, 无意外数据泄露
    trace_refs:
      - TP-DFX-DAT-009
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 执行 page=0 非法分页查询 | DUT1 | fw_capture_bgp6_log type="neighbor_change", page=0, size=10 | API 返回错误 |
| 2 | 验证未返回意外数据 | DUT1 | fw_verify_bgp6_log type="neighbor_change" | 无数据泄露 |

### expected_results

1. page=0 被 API 拒绝
2. 返回明确错误提示（page 必须 >= 1）

---

## PC-DFX-DAT-010：分页 size=0 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-DFX-DAT-010 |
| `logic_case_id` | LC-DFX-001 |
| `requirement_ids` | SR-BGP6-DFX |
| `feature_tags` | BGP4+, 日志查询, 分页, 非法值, 可维护性 |
| `case_title` | 验证分页 size=0 非法值被拒绝 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 日志系统正常运行 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 非法值: size=0 不在合法范围 [1, N] |
| `trigger_data` | type="neighbor_change", page=1, size=0 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-DFX-DAT-010 |
| `scenario_refs` | S-DFX-01 |
| `scenario_chain_refs` | S-DFX-01 abnormal_path "size=0 拒绝" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 执行 size=0 非法分页查询
    target: DUT1
    atomic_op:
      op_id: fw_capture_bgp6_log
      args:
        type: "neighbor_change"
        page: 1
        size: 0
    expected_result: API 返回错误, 提示 size 必须 >= 1
    trace_refs:
      - TP-DFX-DAT-010

  - step_id: STEP-002
    step_name: 验证未返回意外数据
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_log
      args:
        type: "neighbor_change"
    expected_result: 查询被拒绝, 无意外数据泄露
    trace_refs:
      - TP-DFX-DAT-010
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 执行 size=0 非法分页查询 | DUT1 | fw_capture_bgp6_log type="neighbor_change", page=1, size=0 | API 返回错误 |
| 2 | 验证未返回意外数据 | DUT1 | fw_verify_bgp6_log type="neighbor_change" | 无数据泄露 |

### expected_results

1. size=0 被 API 拒绝
2. 返回明确错误提示（size 必须 >= 1）

---

## PC-DFX-DAT-011：分页 page=-1 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-DFX-DAT-011 |
| `logic_case_id` | LC-DFX-001 |
| `requirement_ids` | SR-BGP6-DFX |
| `feature_tags` | BGP4+, 日志查询, 分页, 负数, 可维护性 |
| `case_title` | 验证分页 page=-1 负数值被拒绝 |
| `priority` | P3 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 日志系统正常运行 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 非法值: page 为负数, 不应被接受 |
| `trigger_data` | type="neighbor_change", page=-1, size=10 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-DFX-DAT-011 |
| `scenario_refs` | S-DFX-01 |
| `scenario_chain_refs` | S-DFX-01 abnormal_path "page=-1 拒绝" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 执行 page=-1 负数分页查询
    target: DUT1
    atomic_op:
      op_id: fw_capture_bgp6_log
      args:
        type: "neighbor_change"
        page: -1
        size: 10
    expected_result: API 返回错误, 提示 page 必须为正整数
    trace_refs:
      - TP-DFX-DAT-011

  - step_id: STEP-002
    step_name: 验证未返回意外数据
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_log
      args:
        type: "neighbor_change"
    expected_result: 查询被拒绝, 无异常数据返回
    trace_refs:
      - TP-DFX-DAT-011
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 执行 page=-1 负数分页查询 | DUT1 | fw_capture_bgp6_log type="neighbor_change", page=-1, size=10 | API 返回错误 |
| 2 | 验证未返回意外数据 | DUT1 | fw_verify_bgp6_log type="neighbor_change" | 无数据泄露 |

### expected_results

1. page=-1 被 API 拒绝
2. 返回明确错误提示（page 必须为正整数）

---

## PC-DFX-DAT-012：分页 size=-1 拒绝

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-DFX-DAT-012 |
| `logic_case_id` | LC-DFX-001 |
| `requirement_ids` | SR-BGP6-DFX |
| `feature_tags` | BGP4+, 日志查询, 分页, 负数, 可维护性 |
| `case_title` | 验证分页 size=-1 负数值被拒绝 |
| `priority` | P3 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 日志系统正常运行 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 非法值: size 为负数, 不应被接受 |
| `trigger_data` | type="neighbor_change", page=1, size=-1 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-DFX-DAT-012 |
| `scenario_refs` | S-DFX-01 |
| `scenario_chain_refs` | S-DFX-01 abnormal_path "size=-1 拒绝" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 执行 size=-1 负数分页查询
    target: DUT1
    atomic_op:
      op_id: fw_capture_bgp6_log
      args:
        type: "neighbor_change"
        page: 1
        size: -1
    expected_result: API 返回错误, 提示 size 必须为正整数
    trace_refs:
      - TP-DFX-DAT-012

  - step_id: STEP-002
    step_name: 验证未返回意外数据
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_log
      args:
        type: "neighbor_change"
    expected_result: 查询被拒绝, 无异常数据返回
    trace_refs:
      - TP-DFX-DAT-012
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 执行 size=-1 负数分页查询 | DUT1 | fw_capture_bgp6_log type="neighbor_change", page=1, size=-1 | API 返回错误 |
| 2 | 验证未返回意外数据 | DUT1 | fw_verify_bgp6_log type="neighbor_change" | 无数据泄露 |

### expected_results

1. size=-1 被 API 拒绝
2. 返回明确错误提示（size 必须为正整数）

---

## PC-DFX-DAT-013：空日志类型查询

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-DFX-DAT-013 |
| `logic_case_id` | LC-DFX-001 |
| `requirement_ids` | SR-BGP6-DFX |
| `feature_tags` | BGP4+, 日志查询, 空类型, 可维护性 |
| `case_title` | 验证空日志类型查询被拒绝 |
| `priority` | P3 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 日志系统正常运行 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 必填参数校验: type 为空 |
| `trigger_data` | type="", page=1, size=10 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-DFX-DAT-013 |
| `scenario_refs` | S-DFX-01 |
| `scenario_chain_refs` | S-DFX-01 abnormal_path "空类型拒绝" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 执行空日志类型查询
    target: DUT1
    atomic_op:
      op_id: fw_capture_bgp6_log
      args:
        type: ""
        page: 1
        size: 10
    expected_result: API 返回错误, 提示 type 不能为空, 可选类型: neighbor_change, route_add, route_delete, auth
    trace_refs:
      - TP-DFX-DAT-013

  - step_id: STEP-002
    step_name: 验证未返回意外数据
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_log
      args:
        type: ""
    expected_result: 查询被拒绝, 无意外数据返回
    trace_refs:
      - TP-DFX-DAT-013
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 执行空日志类型查询 | DUT1 | fw_capture_bgp6_log type="", page=1, size=10 | API 返回错误 |
| 2 | 验证未返回意外数据 | DUT1 | fw_verify_bgp6_log type="" | 无数据泄露 |

### expected_results

1. 空日志类型被 API 拒绝
2. 返回明确错误提示，列出可用类型

---

## PC-DFX-DAT-014：非法日志类型 debug

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-DFX-DAT-014 |
| `logic_case_id` | LC-DFX-001 |
| `requirement_ids` | SR-BGP6-DFX |
| `feature_tags` | BGP4+, 日志查询, 非法类型, 可维护性 |
| `case_title` | 验证非法日志类型 debug 被拒绝 |
| `priority` | P3 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 日志系统正常运行 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 非法值: type 为未定义的 debug |
| `trigger_data` | type="debug", page=1, size=10 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-DFX-DAT-014 |
| `scenario_refs` | S-DFX-01 |
| `scenario_chain_refs` | S-DFX-01 abnormal_path "非法类型拒绝" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 执行非法日志类型 debug 查询
    target: DUT1
    atomic_op:
      op_id: fw_capture_bgp6_log
      args:
        type: "debug"
        page: 1
        size: 10
    expected_result: API 返回错误, 提示不支持的类型 debug, 可选类型: neighbor_change, route_add, route_delete, auth
    trace_refs:
      - TP-DFX-DAT-014

  - step_id: STEP-002
    step_name: 验证未返回意外数据
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_log
      args:
        type: "debug"
    expected_result: 查询被拒绝, 无意外数据返回
    trace_refs:
      - TP-DFX-DAT-014
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 执行非法日志类型 debug 查询 | DUT1 | fw_capture_bgp6_log type="debug", page=1, size=10 | API 返回错误 |
| 2 | 验证未返回意外数据 | DUT1 | fw_verify_bgp6_log type="debug" | 无数据泄露 |

### expected_results

1. 非法类型 debug 被 API 拒绝
2. 返回明确错误提示，列出可用类型

---

## PC-DFX-DAT-015：分页 size=99999 极大值 [待确认]

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-DFX-DAT-015 |
| `logic_case_id` | LC-DFX-001 |
| `requirement_ids` | SR-BGP6-DFX |
| `feature_tags` | BGP4+, 日志查询, 分页, 极大值, 可维护性 |
| `case_title` | 验证分页 size=99999 极大值查询行为 [待确认] |
| `priority` | P3 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 日志系统正常运行 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 边界行为: size 极大值是否被截断或接受 |
| `trigger_data` | type="neighbor_change", page=1, size=99999 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-DFX-DAT-015 |
| `scenario_refs` | S-DFX-01 |
| `scenario_chain_refs` | — |
| `action_source_refs` | — |
| `confirmation_gap_refs` | GAP-DFX-DAT-015: size 极大值处理策略待确认 |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 执行 size=99999 极大值分页查询
    target: DUT1
    atomic_op:
      op_id: fw_capture_bgp6_log
      args:
        type: "neighbor_change"
        page: 1
        size: 99999
    expected_result: "[待确认] API 返回 success 并截断到上限; 或返回错误提示; 不导致 OOM"
    trace_refs:
      - TP-DFX-DAT-015
    confirmation_gap_refs:
      - GAP-DFX-DAT-015

  - step_id: STEP-002
    step_name: 验证系统状态正常
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_log
      args:
        type: "neighbor_change"
    expected_result: "[待确认] 日志查询功能正常, 无 crash 或 OOM"
    trace_refs:
      - TP-DFX-DAT-015
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 执行 size=99999 极大值分页查询 | DUT1 | fw_capture_bgp6_log type="neighbor_change", page=1, size=99999 | [待确认] |
| 2 | 验证系统状态正常 | DUT1 | fw_verify_bgp6_log type="neighbor_change" | [待确认] |

### expected_results

[待确认] 根据实现不同：
- 场景A: 系统截断 size 到内部上限（如 1000），正常返回
- 场景B: 返回错误，要求 size 不超过某上限
- 场景C: size 无上限，可能导致 OOM（需确认防护机制）

---

## PC-DFX-DAT-016：分页 page=null 查询 [待确认]

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-DFX-DAT-016 |
| `logic_case_id` | LC-DFX-001 |
| `requirement_ids` | SR-BGP6-DFX |
| `feature_tags` | BGP4+, 日志查询, 分页, null, 可维护性 |
| `case_title` | 验证分页 page=null 查询行为 [待确认] |
| `priority` | P3 |
| `preconditions` | DUT1 BGP4+ 全局使能 (asNumber=100); 日志系统正常运行 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 空值行为: page 为 null 时的处理策略 |
| `trigger_data` | type="neighbor_change", page=null, size=10 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-DFX-DAT-016 |
| `scenario_refs` | S-DFX-01 |
| `scenario_chain_refs` | — |
| `action_source_refs` | — |
| `confirmation_gap_refs` | GAP-DFX-DAT-016: null page 处理策略待确认 |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 执行 page=null 分页查询
    target: DUT1
    atomic_op:
      op_id: fw_capture_bgp6_log
      args:
        type: "neighbor_change"
        page: null
        size: 10
    expected_result: "[待确认] API 返回错误(NullPointer/参数缺失); 或默认 page=1; 不导致系统异常"
    trace_refs:
      - TP-DFX-DAT-016
    confirmation_gap_refs:
      - GAP-DFX-DAT-016

  - step_id: STEP-002
    step_name: 验证系统状态正常
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_log
      args:
        type: "neighbor_change"
    expected_result: "[待确认] 日志查询功能正常, 无 crash"
    trace_refs:
      - TP-DFX-DAT-016
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 执行 page=null 分页查询 | DUT1 | fw_capture_bgp6_log type="neighbor_change", page=null, size=10 | [待确认] |
| 2 | 验证系统状态正常 | DUT1 | fw_verify_bgp6_log type="neighbor_change" | [待确认] |

### expected_results

[待确认] 根据实现不同：
- 场景A: API 层拦截，返回参数缺失错误
- 场景B: 后端默认 page=1，正常返回首页
- 场景C: 导致 NullPointerException / 500 错误
