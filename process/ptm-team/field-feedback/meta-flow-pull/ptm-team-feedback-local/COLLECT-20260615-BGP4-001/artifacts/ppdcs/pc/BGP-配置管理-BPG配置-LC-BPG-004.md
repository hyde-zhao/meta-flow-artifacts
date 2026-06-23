# BGP4+ MD5 认证邻居 — 物理用例

> 生成时间: 2026-06-12 | 设计Skill: state-design
> 逻辑用例: LC-BPG-004 | 目录: BGP-配置管理-BPG配置

---

## PC-BPG-004-01：无MD5邻居启用MD5认证成功

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-004-01 |
| `logic_case_id` | LC-BPG-004 |
| `requirement_ids` | SR-007e |
| `feature_tags` | BGP4+, MD5认证, 邻居安全 |
| `case_title` | 验证无MD5认证的BGP4+邻居两端同时启用MD5且密码一致时保持Established |
| `priority` | P0 |
| `preconditions` | DUT1/DUT2 BGP4+全局使能 (asNumber=100); 接口IPv6直连互通; IBGP邻居已Established (无MD5) |
| `graph_ref` | SP-01 |
| `coverage_goal` | 主生命周期: T1 迁移 (S0→S1), 合法迁移全覆盖 |
| `trigger_data` | authEnable=true, md5Password="TestPass123" (两端一致) |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-016, S07-OP2 |
| `scenario_refs` | S07 |
| `scenario_chain_refs` | S07 normal_path 步骤1→2 |
| `action_source_refs` | S07-OP1, S07-OP2 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认DUT1-DUT2 IBGP邻居已Established且无MD5认证
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, authEnable=false
    trace_refs:
      - TP-M1-016
      - S07-OP1

  - step_id: STEP-002
    step_name: DUT1启用MD5认证并设置密码
    target: DUT1
    atomic_op:
      op_id: put-bgp6neighbor-auth
      args:
        id: "2001:db8:1::2"
        authEnable: true
        md5Password: "TestPass123"
    expected_result: API返回success, 配置生效
    trace_refs:
      - S07-OP2

  - step_id: STEP-003
    step_name: DUT2启用MD5认证并设置相同密码
    target: DUT2
    atomic_op:
      op_id: put-bgp6neighbor-auth
      args:
        id: "2001:db8:1::1"
        authEnable: true
        md5Password: "TestPass123"
    expected_result: API返回success, 配置生效
    trace_refs:
      - S07-OP2

  - step_id: STEP-004
    step_name: 验证邻居保持Established且MD5认证已生效
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, authEnable=true, TCP MD5 Signature Option生效
    trace_refs:
      - TP-M1-016

  - step_id: STEP-005
    step_name: 清理: 两端禁用MD5认证
    target: DUT1, DUT2
    atomic_op:
      op_id: put-bgp6neighbor-auth
      args:
        authEnable: false
    expected_result: 恢复无MD5认证, 邻居保持Established
    trace_refs:
      - S07 exit_action
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认DUT1-DUT2 IBGP邻居已Established且无MD5认证 | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 邻居状态=Established, authEnable=false |
| 2 | DUT1启用MD5认证并设置密码 | DUT1 | put-bgp6neighbor-auth authEnable=true, md5Password="TestPass123" | API返回success |
| 3 | DUT2启用MD5认证并设置相同密码 | DUT2 | put-bgp6neighbor-auth authEnable=true, md5Password="TestPass123" | API返回success |
| 4 | 验证邻居保持Established且MD5认证已生效 | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 邻居状态=Established, authEnable=true |
| 5 | 清理: 两端禁用MD5认证 | DUT1, DUT2 | put-bgp6neighbor-auth authEnable=false | 恢复无MD5认证, 邻居保持Established |

### expected_results

1. 无MD5认证邻居正常 Established
2. 两端同时启用 MD5 且密码一致，邻居保持 Established
3. MD5 认证在 TCP 层生效（通过抓包验证 TCP MD5 Signature Option）
4. 邻居可正常交换 UPDATE 消息

---

## PC-BPG-004-02：MD5密码不一致导致邻居断开

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-004-02 |
| `logic_case_id` | LC-BPG-004 |
| `requirement_ids` | SR-007e |
| `feature_tags` | BGP4+, MD5认证, 异常路径 |
| `case_title` | 验证MD5认证建立后修改一端密码不一致导致邻居断开 |
| `priority` | P0 |
| `preconditions` | DUT1/DUT2 BGP4+ MD5认证已启用且密码一致 ("TestPass123"), 邻居 Established |
| `graph_ref` | SP-02 |
| `coverage_goal` | 异常回退: T2 迁移 (S1→S2) |
| `trigger_data` | DUT2 md5Password 改为 "WrongPass" (与 DUT1 "TestPass123" 不一致) |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-017, S07-OP3 |
| `scenario_refs` | S07 |
| `scenario_chain_refs` | S07 normal_path 步骤4 |
| `action_source_refs` | S07-OP3 |
| `confirmation_gap_refs` | GAP-MD5-001: 断开触发机制 (立即/Hold Timer超时) |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认MD5认证邻居已Established (密码一致)
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, authEnable=true
    trace_refs:
      - TP-M1-016

  - step_id: STEP-002
    step_name: 修改DUT2 MD5密码为不一致的值
    target: DUT2
    atomic_op:
      op_id: put-bgp6neighbor-auth
      args:
        id: "2001:db8:1::1"
        md5Password: "WrongPass"
    expected_result: API返回success, 密码已修改
    trace_refs:
      - S07-OP3

  - step_id: STEP-003
    step_name: 观察邻居状态变化
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: "[待确认] 邻居立即断开或Hold Timer超时(180s)后断开 → 状态变为Idle"
    trace_refs:
      - TP-M1-017
    confirmation_gap_refs:
      - GAP-MD5-001

  - step_id: STEP-004
    step_name: 清理: 恢复密码一致
    target: DUT2
    atomic_op:
      op_id: put-bgp6neighbor-auth
      args:
        id: "2001:db8:1::1"
        md5Password: "TestPass123"
    expected_result: 邻居重新建立 Established
    trace_refs:
      - S07-OP4
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认MD5认证邻居已Established | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 邻居状态=Established |
| 2 | 修改DUT2 MD5密码为不一致的值 | DUT2 | put-bgp6neighbor-auth md5Password="WrongPass" | API返回success |
| 3 | 观察邻居状态变化 | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | [待确认] 断开时机, 最终 Idle |
| 4 | 清理: 恢复密码一致 | DUT2 | put-bgp6neighbor-auth md5Password="TestPass123" | 邻居重新 Established |

### expected_results

1. 修改对端密码不一致后，邻居无法通过 MD5 校验
2. 邻居断开（Idle），无法重新建立（持续认证失败）
3. 恢复密码一致后，邻居重新建立 Established

---

## PC-BPG-004-05：MD5密码边界 — 空密码

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-004-05 |
| `logic_case_id` | LC-BPG-004 |
| `requirement_ids` | SR-007e |
| `feature_tags` | BGP4+, MD5认证, 边界值 |
| `case_title` | 验证启用MD5时设置空密码被拒绝 |
| `priority` | P1 |
| `preconditions` | DUT1/DUT2 IBGP邻居已Established (无MD5) |
| `graph_ref` | SP-05 |
| `coverage_goal` | 边界守卫: 空密码 T7 非法迁移 |
| `trigger_data` | authEnable=true, md5Password="" |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-018 |
| `scenario_refs` | S07 |
| `scenario_chain_refs` | S07 abnormal_path "密码为空" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 尝试以空密码启用MD5认证
    target: DUT1
    atomic_op:
      op_id: put-bgp6neighbor-auth
      args:
        id: "2001:db8:1::2"
        authEnable: true
        md5Password: ""
    expected_result: API返回错误, 提示密码不能为空
    trace_refs:
      - TP-M1-018

  - step_id: STEP-002
    step_name: 验证邻居状态未变化
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居仍为 Established, authEnable=false (MD5未启用)
    trace_refs:
      - TP-M1-018
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 尝试以空密码启用MD5认证 | DUT1 | put-bgp6neighbor-auth authEnable=true, md5Password="" | API返回错误 |
| 2 | 验证邻居状态未变化 | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 邻居Established, authEnable=false |

### expected_results

1. 空密码被 API 拒绝，返回明确错误信息
2. 邻居保持原状，不进入 MD5 认证模式

---

## PC-BPG-004-08：MD5密码边界 — 特殊字符合法

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-004-08 |
| `logic_case_id` | LC-BPG-004 |
| `requirement_ids` | SR-007e |
| `feature_tags` | BGP4+, MD5认证, 边界值 |
| `case_title` | 验证MD5密码包含特殊字符时正常建立认证 |
| `priority` | P1 |
| `preconditions` | DUT1/DUT2 IBGP邻居已Established (无MD5) |
| `graph_ref` | SP-08 |
| `coverage_goal` | 边界守卫: 特殊字符合法通过 |
| `trigger_data` | authEnable=true, md5Password="!@#$%^&*()" |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-M1-018 |
| `scenario_refs` | S07 |
| `scenario_chain_refs` | S07 abnormal_path "密码含特殊字符" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 两端同时启用MD5, 密码使用特殊字符
    target: DUT1, DUT2
    atomic_op:
      op_id: put-bgp6neighbor-auth
      args:
        authEnable: true
        md5Password: "!@#$%^&*()"
    expected_result: API返回success, 两端均接受特殊字符密码
    trace_refs:
      - TP-M1-018

  - step_id: STEP-002
    step_name: 验证邻居保持Established
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, authEnable=true
    trace_refs:
      - TP-M1-018

  - step_id: STEP-003
    step_name: 清理: 禁用MD5
    target: DUT1, DUT2
    atomic_op:
      op_id: put-bgp6neighbor-auth
      args:
        authEnable: false
    expected_result: 恢复无MD5, 邻居保持Established
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 两端启用MD5, 密码使用特殊字符 | DUT1, DUT2 | put-bgp6neighbor-auth authEnable=true, md5Password="!@#$%^&*()" | API返回success |
| 2 | 验证邻居保持Established | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | 邻居Established, authEnable=true |
| 3 | 清理: 禁用MD5 | DUT1, DUT2 | put-bgp6neighbor-auth authEnable=false | 恢复无MD5 |

### expected_results

1. 特殊字符密码被正常接受（非注入攻击）
2. MD5 认证正常建立，邻居保持 Established

---

## PC-BPG-004-10：非法迁移 — 单端启用MD5 [待确认]

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-BPG-004-10 |
| `logic_case_id` | LC-BPG-004 |
| `requirement_ids` | SR-007e |
| `feature_tags` | BGP4+, MD5认证, 非法迁移 |
| `case_title` | 验证仅一端启用MD5认证时邻居行为 [待确认] |
| `priority` | P2 |
| `preconditions` | DUT1/DUT2 IBGP邻居已Established (无MD5) |
| `graph_ref` | SP-06 |
| `coverage_goal` | 非法迁移: T5 (S0→S0) 单端MD5 |
| `trigger_data` | DUT1: authEnable=true, md5Password="TestPass123"; DUT2: authEnable=false |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-Q-SEC-001 |
| `scenario_refs` | S07 |
| `scenario_chain_refs` | — |
| `action_source_refs` | — |
| `confirmation_gap_refs` | GAP-MD5-003: 单端MD5行为待确认 |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: DUT1启用MD5认证, DUT2保持无MD5
    target: DUT1
    atomic_op:
      op_id: put-bgp6neighbor-auth
      args:
        id: "2001:db8:1::2"
        authEnable: true
        md5Password: "TestPass123"
    expected_result: "[待确认] DUT1发送带MD5签名的TCP报文; DUT2行为待确认"
    trace_refs:
      - TP-Q-SEC-001

  - step_id: STEP-002
    step_name: 观察邻居状态变化
    target: DUT1
    atomic_op:
      op_id: get-bgp6neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: "[待确认] 邻居保持Established(无MD5端不验证) 或 断开(校验失败)"
    trace_refs:
      - TP-Q-SEC-001
    confirmation_gap_refs:
      - GAP-MD5-003
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | DUT1启用MD5, DUT2保持无MD5 | DUT1 | put-bgp6neighbor-auth authEnable=true, md5Password="TestPass123" | [待确认] |
| 2 | 观察邻居状态变化 | DUT1 | get-bgp6neighbor id=2001:db8:1::2 | [待确认] 保持或断开 |

### expected_results

[待确认] 根据实现不同：
- 场景A: DUT2 (无MD5) 忽略 MD5 Signature Option，邻居保持 Established
- 场景B: DUT1 (有MD5) 期望对端也带 MD5，校验失败断开
