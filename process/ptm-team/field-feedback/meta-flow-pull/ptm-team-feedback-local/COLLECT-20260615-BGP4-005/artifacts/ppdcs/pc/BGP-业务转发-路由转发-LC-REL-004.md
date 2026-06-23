# BGP4+ 数据面/协议级可靠性 — 物理用例

> 生成时间: 2026-06-12 | 设计Skill: parameter-design
> 逻辑用例: LC-REL-004 | 目录: BGP-业务转发-路由转发

---

## PC-REL4-PAR-001：路由震荡50条×10轮 路由同步正确 bgpd不崩溃

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL4-PAR-001 |
| `logic_case_id` | LC-REL-004 |
| `requirement_ids` | SR-REL-004 |
| `feature_tags` | BGP4+, 协议可靠性, 路由震荡, 稳定性 |
| `case_title` | 验证50条路由×10轮震荡后路由同步正确且bgpd进程不崩溃 |
| `priority` | P1 |
| `preconditions` | DUT1-DUT2 IBGP邻居已Established, 50条BGP4+路由已通告 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 协议级可靠性: 路由震荡50条×10轮 |
| `trigger_data` | 路由震荡: 50条路由 × 10轮 withdraw/announce 循环 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M4-001 |
| `scenario_refs` | S-REL-004 |
| `scenario_chain_refs` | S-REL-004 normal_path 步骤1→2 |
| `action_source_refs` | S-REL-004-OP1 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认初始路由表和bgpd进程状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args: {}
    expected_result: 50条BGP4+路由已学习且正确, bgpd进程=running
    trace_refs:
      - TP-REL-M4-001

  - step_id: STEP-002
    step_name: 执行路由震荡: 50条路由 × 10轮 withdraw/announce
    target: DUT2
    atomic_op:
      op_id: fw_inject_bgp6_route
      args:
        route_count: 50
        ops: "withdraw"
        cycles: 10
    expected_result: 10轮震荡完成, 无异常
    trace_refs:
      - S-REL-004-OP1

  - step_id: STEP-003
    step_name: 验证bgpd进程未崩溃
    target: DUT1
    atomic_op:
      op_id: fw_verify_process
      args:
        process_name: "bgpd"
    expected_result: bgpd进程=running, 运行中无crash
    trace_refs:
      - TP-REL-M4-001

  - step_id: STEP-004
    step_name: 验证最终路由表正确
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args: {}
    expected_result: 路由表包含50条正确路由, 下一跳/AS-Path/属性一致
    trace_refs:
      - TP-REL-M4-001
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认初始路由和进程 | DUT1 | fw_verify_bgp6_route | 50条路由正确, bgpd=running |
| 2 | 执行50条×10轮震荡 | DUT2 | fw_inject_bgp6_route route_count=50, cycles=10 | 10轮完成 |
| 3 | 验证bgpd未崩溃 | DUT1 | fw_verify_process process_name=bgpd | bgpd=running |
| 4 | 验证最终路由正确 | DUT1 | fw_verify_bgp6_route | 50条路由正确 |

### expected_results

1. 10轮路由撤销/通告震荡后 bgpd 进程未崩溃
2. 每轮震荡后路由表正确更新 (撤销时移除, 通告时加入)
3. 最终路由表与预期一致, 无残留无效路由
4. 震荡期间邻居保持 Established

---

## PC-REL4-PAR-002：畸形报文注入 bgpd不崩溃

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL4-PAR-002 |
| `logic_case_id` | LC-REL-004 |
| `requirement_ids` | SR-REL-004 |
| `feature_tags` | BGP4+, 协议可靠性, 畸形报文, 鲁棒性 |
| `case_title` | 验证畸形BGP报文注入后bgpd进程不崩溃 |
| `priority` | P1 |
| `preconditions` | DUT1-DUT2 IBGP邻居已Established |
| `graph_ref` | SP-02 |
| `coverage_goal` | 协议级可靠性: 畸形报文注入, bgpd不崩溃 |
| `trigger_data` | 发送畸形BGP报文 (非法长度/非法类型/格式错误) |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M4-002 |
| `scenario_refs` | S-REL-004 |
| `scenario_chain_refs` | S-REL-004 normal_path 步骤3 |
| `action_source_refs` | S-REL-004-OP2 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认邻居Established且bgpd正常
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, bgpd=running
    trace_refs:
      - TP-REL-M4-002

  - step_id: STEP-002
    step_name: 发送畸形BGP报文 (非法长度/非法类型/格式错误)
    target: DUT1
    atomic_op:
      op_id: fw_inject_malformed_packet
      args:
        protocol: "bgp"
        malformed_types:
          - "invalid_length"
          - "invalid_type"
          - "malformed_attributes"
    expected_result: 畸形报文注入完成
    trace_refs:
      - S-REL-004-OP2

  - step_id: STEP-003
    step_name: 验证bgpd进程未崩溃
    target: DUT1
    atomic_op:
      op_id: fw_verify_process
      args:
        process_name: "bgpd"
    expected_result: bgpd进程=running, 未crash, 正确处理畸形报文
    trace_refs:
      - TP-REL-M4-002

  - step_id: STEP-004
    step_name: 验证邻居仍为Established
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, 或发送NOTIFICATION后重建
    trace_refs:
      - TP-REL-M4-002
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认邻居Established | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 状态=Established |
| 2 | 注入畸形报文 | DUT1 | fw_inject_malformed_packet protocol=bgp | 畸形报文注入 |
| 3 | 验证bgpd未崩溃 | DUT1 | fw_verify_process process_name=bgpd | bgpd=running |
| 4 | 验证邻居状态 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 状态=Established或重建后Established |

### expected_results

1. 畸形 BGP 报文必须被正确识别并发送 NOTIFICATION
2. bgpd 进程不因畸形报文崩溃或死循环
3. 如果有 NOTIFICATION, 需检查错误码是否一致
4. 可选: 验证畸形报文数统计计数器增加

---

## PC-REL4-PAR-003：24h长稳 内存<5% 邻居稳定

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL4-PAR-003 |
| `logic_case_id` | LC-REL-004 |
| `requirement_ids` | SR-REL-004 |
| `feature_tags` | BGP4+, 协议可靠性, 长稳, 内存 |
| `case_title` | 验证BGP4+ 24小时长稳运行后内存<5%且邻居稳定不震荡 |
| `priority` | P1 |
| `preconditions` | DUT1-DUT2 IBGP邻居已Established, 有稳定路由通告 |
| `graph_ref` | SP-03 |
| `coverage_goal` | 协议级可靠性: 24h长稳 |
| `trigger_data` | 运行24小时, 持续监控内存和邻居状态 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M4-003 |
| `scenario_refs` | S-REL-004 |
| `scenario_chain_refs` | S-REL-004 normal_path 步骤4 |
| `action_source_refs` | S-REL-004-OP3 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 记录初始内存和邻居状态基线
    target: DUT1
    atomic_op:
      op_id: get-system-memory
      args: {}
    expected_result: 记录bgpd内存初始值, 邻居=Established
    trace_refs:
      - TP-REL-M4-003

  - step_id: STEP-002
    step_name: 持续运行24小时, 周期性验证邻居和内存
    target: DUT1
    atomic_op:
      op_id: monitor-24h
      args:
        interval: 3600
        metrics:
          - bgp_neighbor_state
          - bgpd_memory
    expected_result: 每1小时记录一次, 24小时内邻居始终Established
    trace_refs:
      - S-REL-004-OP3

  - step_id: STEP-003
    step_name: 24h后验证bgpd内存增长
    target: DUT1
    atomic_op:
      op_id: get-system-memory
      args: {}
    expected_result: bgpd内存增长<5% 或 增长后稳定无持续泄漏
    trace_refs:
      - TP-REL-M4-003

  - step_id: STEP-004
    step_name: 验证邻居和路由表最终状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居=Established, 路由表与基线一致
    trace_refs:
      - TP-REL-M4-003
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 记录基线 | DUT1 | get-system-memory | 内存基线, 邻居=Established |
| 2 | 24h长稳运行 | DUT1 | monitor-24h interval=3600s | 邻居始终Established |
| 3 | 验证内存增长 | DUT1 | get-system-memory | bgpd内存增长<5% |
| 4 | 验证最终状态 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 邻居=Established |

### expected_results

1. 24小时连续运行邻居状态不震荡
2. bgpd 内存增长 <5%, 无持续内存泄漏
3. 路由表始终保持与预期一致
4. 无异常进程重启或 Watchfrr 干预

---

## PC-REL4-PAR-004：升级旧版本→C03 配置保留邻居重建

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL4-PAR-004 |
| `logic_case_id` | LC-REL-004 |
| `requirement_ids` | SR-REL-004 |
| `feature_tags` | BGP4+, 协议可靠性, 升级, 配置保留 |
| `case_title` | 验证从旧版本升级到C03后BGP配置保留且邻居正常重建 |
| `priority` | P1 |
| `preconditions` | DUT1运行旧版本固件, BGP4+邻居已Established, 配置已备份 |
| `graph_ref` | SP-04 |
| `coverage_goal` | 协议级可靠性: 旧版本→C03升级 |
| `trigger_data` | 升级固件: 旧版本→C03 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M4-004 |
| `scenario_refs` | S-REL-004 |
| `scenario_chain_refs` | S-REL-004 normal_path 步骤5→6 |
| `action_source_refs` | S-REL-004-OP4 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认旧版本固件及当前BGP配置
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: 旧版本固件版本号, BGP配置已记录, 邻居=Established
    trace_refs:
      - TP-REL-M4-004

  - step_id: STEP-002
    step_name: 升级固件到C03版本
    target: DUT1
    atomic_op:
      op_id: fw_upgrade_firmware
      args:
        version: "C03"
    expected_result: 固件升级完成, 设备重启, 新版本=C03
    trace_refs:
      - S-REL-004-OP4

  - step_id: STEP-003
    step_name: 验证BGP配置保留
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: BGP全局配置与升级前一致, AS号/路由器ID/邻居均保留
    trace_refs:
      - TP-REL-M4-004

  - step_id: STEP-004
    step_name: 验证邻居重建
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established
    trace_refs:
      - TP-REL-M4-004

  - step_id: STEP-005
    step_name: 验证路由表恢复
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args: {}
    expected_result: 路由表与升级前一致
    trace_refs:
      - TP-REL-M4-004
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认旧版本及配置 | DUT1 | fw_verify_bgp_global | 旧版本号, 配置已记录 |
| 2 | 升级到C03 | DUT1 | fw_upgrade_firmware version=C03 | 升级完成, 版本=C03 |
| 3 | 验证配置保留 | DUT1 | fw_verify_bgp_global | 配置与升级前一致 |
| 4 | 验证邻居重建 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 邻居=Established |
| 5 | 验证路由恢复 | DUT1 | fw_verify_bgp6_route | 路由与升级前一致 |

### expected_results

1. 升级到 C03 后 BGP 全局配置完整保留 (无丢失)
2. BGP4+ 邻居自动重建到 Established
3. 路由表恢复与升级前一致
4. 升级过程无配置兼容性问题

---

## PC-REL4-PAR-005：回滚C03→旧版本 配置恢复邻居重建

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL4-PAR-005 |
| `logic_case_id` | LC-REL-004 |
| `requirement_ids` | SR-REL-004 |
| `feature_tags` | BGP4+, 协议可靠性, 回滚, 配置恢复 |
| `case_title` | 验证从C03回滚到旧版本后BGP配置恢复且邻居正常重建 |
| `priority` | P1 |
| `preconditions` | DUT1运行C03版本, BGP4+邻居已Established |
| `graph_ref` | SP-05 |
| `coverage_goal` | 协议级可靠性: C03→旧版本回滚 |
| `trigger_data` | 回滚固件: C03→旧版本 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M4-005 |
| `scenario_refs` | S-REL-004 |
| `scenario_chain_refs` | S-REL-004 normal_path 步骤7→8 |
| `action_source_refs` | S-REL-004-OP5 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认C03版本及当前BGP配置
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: C03版本, BGP配置已记录, 邻居=Established
    trace_refs:
      - TP-REL-M4-005

  - step_id: STEP-002
    step_name: 回滚固件到旧版本
    target: DUT1
    atomic_op:
      op_id: fw_rollback_firmware
      args:
        version: "previous"
    expected_result: 固件回滚完成, 设备重启, 版本=旧版本
    trace_refs:
      - S-REL-004-OP5

  - step_id: STEP-003
    step_name: 验证BGP配置恢复
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: BGP配置恢复与回滚前一致
    trace_refs:
      - TP-REL-M4-005

  - step_id: STEP-004
    step_name: 验证邻居重建
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established
    trace_refs:
      - TP-REL-M4-005

  - step_id: STEP-005
    step_name: 验证路由表恢复
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args: {}
    expected_result: 路由表与回滚前一致
    trace_refs:
      - TP-REL-M4-005
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认C03版本及配置 | DUT1 | fw_verify_bgp_global | C03, 配置已记录 |
| 2 | 回滚到旧版本 | DUT1 | fw_rollback_firmware version=previous | 回滚完成, 版本=旧版本 |
| 3 | 验证配置恢复 | DUT1 | fw_verify_bgp_global | 配置一致 |
| 4 | 验证邻居重建 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 邻居=Established |
| 5 | 验证路由恢复 | DUT1 | fw_verify_bgp6_route | 路由一致 |

### expected_results

1. 回滚到旧版本后 BGP 配置完整恢复 (向后兼容)
2. BGP4+ 邻居自动重建到 Established
3. 路由表恢复与回滚前一致
4. 回滚过程中无配置兼容性问题

---

## PC-REL4-PAR-006：邻居震荡100次 bgpd不崩溃每次重建

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL4-PAR-006 |
| `logic_case_id` | LC-REL-004 |
| `requirement_ids` | SR-REL-004 |
| `feature_tags` | BGP4+, 协议可靠性, 邻居震荡, 重建 |
| `case_title` | 验证邻居震荡100次后bgpd不崩溃且每次正常重建 |
| `priority` | P2 |
| `preconditions` | DUT1-DUT2 IBGP邻居已Established |
| `graph_ref` | SP-06 |
| `coverage_goal` | 压力守卫: 邻居震荡100次 |
| `trigger_data` | 邻居震荡=100次 (admin up/down 或 TCP reset) |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M4-006 |
| `scenario_refs` | S-REL-004 |
| `scenario_chain_refs` | S-REL-004 normal_path 步骤9 |
| `action_source_refs` | S-REL-004-OP6 |
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
    expected_result: 邻居状态=Established
    trace_refs:
      - TP-REL-M4-006

  - step_id: STEP-002
    step_name: 触发邻居震荡100次 (通过链路flap或TCP reset)
    target: DUT1
    atomic_op:
      op_id: flap-neighbor
      args:
        id: "2001:db8:1::2"
        cycles: 100
    expected_result: 100次震荡执行完成
    trace_refs:
      - S-REL-004-OP6

  - step_id: STEP-003
    step_name: 验证bgpd进程未崩溃
    target: DUT1
    atomic_op:
      op_id: fw_verify_process
      args:
        process_name: "bgpd"
    expected_result: bgpd进程=running, 未crash, 无内存泄漏
    trace_refs:
      - TP-REL-M4-006

  - step_id: STEP-004
    step_name: 验证最终邻居状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, 每次flap后成功重建
    trace_refs:
      - TP-REL-M4-006
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认邻居Established | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 状态=Established |
| 2 | 触发邻居flap 100次 | DUT1 | flap-neighbor cycles=100 | 100次完成 |
| 3 | 验证bgpd未崩溃 | DUT1 | fw_verify_process process_name=bgpd | bgpd=running |
| 4 | 验证最终邻居 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 邻居=Established |

### expected_results

1. 100次邻居震荡后 bgpd 进程未崩溃
2. 每次震荡后邻居均能成功重建到 Established
3. 无状态机异常 (如卡在某个中间状态)
4. 无内存泄漏

---

## PC-REL4-PAR-007：路由变更50轮 路由表始终正确

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL4-PAR-007 |
| `logic_case_id` | LC-REL-004 |
| `requirement_ids` | SR-REL-004 |
| `feature_tags` | BGP4+, 协议可靠性, 路由变更, 正确性 |
| `case_title` | 验证50轮路由变更后路由表始终与预期一致 |
| `priority` | P2 |
| `preconditions` | DUT1-DUT2 IBGP邻居已Established, 初始路由表已知 |
| `graph_ref` | SP-07 |
| `coverage_goal` | 功能守卫: 路由变更50轮, 路由表始终正确 |
| `trigger_data` | 路由变更=50轮 (每轮增加/删除/修改路由属性) |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M4-007 |
| `scenario_refs` | S-REL-004 |
| `scenario_chain_refs` | S-REL-004 normal_path 步骤10 |
| `action_source_refs` | S-REL-004-OP7 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 记录初始路由表预期状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args: {}
    expected_result: 初始路由表与预期一致, 建立参考基线
    trace_refs:
      - TP-REL-M4-007

  - step_id: STEP-002
    step_name: 执行50轮路由变更 (增/删/改属性)
    target: DUT2
    atomic_op:
      op_id: fw_inject_bgp6_route
      args:
        ops: "add|delete|modify"
        cycles: 50
    expected_result: 50轮变更完成, 每轮操作确认
    trace_refs:
      - S-REL-004-OP7

  - step_id: STEP-003
    step_name: 每轮变更后验证路由表正确性
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args:
        check_per_cycle: true
    expected_result: 每轮变更后路由表与预期完全一致
    trace_refs:
      - TP-REL-M4-007

  - step_id: STEP-004
    step_name: 验证邻居持续Established
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居全程Established, 无断开重建
    trace_refs:
      - TP-REL-M4-007
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 记录基线路由表 | DUT1 | fw_verify_bgp6_route | 基线确定 |
| 2 | 执行50轮路由变更 | DUT2 | fw_inject_bgp6_route ops=add/delete/modify, cycles=50 | 50轮完成 |
| 3 | 每轮验证路由 | DUT1 | fw_verify_bgp6_route check_per_cycle=true | 每轮正确 |
| 4 | 验证邻居状态 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 全程Established |

### expected_results

1. 50轮路由变更中每一轮结束后路由表与预期一致
2. 路由新增/删除/属性修改后表项及时更新
3. 邻居在变更过程中保持 Established 不震荡
4. 无路由残留或丢失

---

## PC-REL4-PAR-008：配置变更100次 一致生效无残留

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL4-PAR-008 |
| `logic_case_id` | LC-REL-004 |
| `requirement_ids` | SR-REL-004 |
| `feature_tags` | BGP4+, 协议可靠性, 配置变更, 无残留 |
| `case_title` | 验证100次BGP配置变更后一致生效且无配置残留 |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+邻居已Established, 配置基线已知 |
| `graph_ref` | SP-08 |
| `coverage_goal` | 功能守卫: 配置变更100次, 一致生效无残留 |
| `trigger_data` | 配置变更=100次 (修改BGP全局属性、邻居属性) |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M4-008 |
| `scenario_refs` | S-REL-004 |
| `scenario_chain_refs` | S-REL-004 normal_path 步骤11 |
| `action_source_refs` | S-REL-004-OP8 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 记录BGP配置基线
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: BGP配置基线已记录
    trace_refs:
      - TP-REL-M4-008

  - step_id: STEP-002
    step_name: 执行100次BGP配置变更
    target: DUT1
    atomic_op:
      op_id: modify-bgpconfig
      args:
        ops: "modify"
        cycles: 100
        config_items:
          - asNumber
          - routerId
          - holdTime
          - keepaliveTime
          - ebgpMultihop
    expected_result: 100次变更全部成功执行
    trace_refs:
      - S-REL-004-OP8

  - step_id: STEP-003
    step_name: 验证每次变更后配置生效
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: 每次变更后配置与预期一致, 生效时间合理
    trace_refs:
      - TP-REL-M4-008

  - step_id: STEP-004
    step_name: 验证最终无配置残留
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: 最终配置与最后一次变更完全一致, 无旧值残留
    trace_refs:
      - TP-REL-M4-008

  - step_id: STEP-005
    step_name: 验证邻居状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, 或按配置预期行为
    trace_refs:
      - TP-REL-M4-008
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 记录配置基线 | DUT1 | fw_verify_bgp_global | 基线已记录 |
| 2 | 执行100次配置变更 | DUT1 | modify-bgpconfig cycles=100 | 100次成功 |
| 3 | 验证每次生效 | DUT1 | fw_verify_bgp_global | 每次一致 |
| 4 | 验证无残留 | DUT1 | fw_verify_bgp_global | 无旧值残留 |
| 5 | 验证邻居状态 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 邻居=Established |

### expected_results

1. 100次配置变更每次均正确生效 (running-config 与预期一致)
2. 配置变更后无旧值残留 (如旧 AS 号、旧 Hold Time)
3. 配置持久化 (frrinc.conf) 与 running-config 同步
4. 邻居根据配置变更做出正确响应

---
