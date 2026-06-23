# BGP4+ 设备级可靠性 — 物理用例

> 生成时间: 2026-06-12 | 设计Skill: parameter-design
> 逻辑用例: LC-REL-002 | 目录: BGP-配置管理-BPG配置

---

## PC-REL2-PAR-001：冷重启完整恢复 配置+邻居+路由

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL2-PAR-001 |
| `logic_case_id` | LC-REL-002 |
| `requirement_ids` | SR-REL-002 |
| `feature_tags` | BGP4+, 设备可靠性, 冷重启, 配置持久化 |
| `case_title` | 验证设备冷重启后BGP配置、邻居、路由完整恢复 |
| `priority` | P1 |
| `preconditions` | DUT1 BGP4+邻居已Established, 路由已学习, 配置已写入持久化存储 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 设备级可靠性: 冷重启完整恢复 |
| `trigger_data` | 冷重启设备, 验证配置+邻居+路由全部恢复 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M2-001 |
| `scenario_refs` | S-REL-002 |
| `scenario_chain_refs` | S-REL-002 normal_path 步骤1→2→3 |
| `action_source_refs` | S-REL-002-OP1, S-REL-002-OP2, S-REL-002-OP3 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认当前BGP配置快照
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: BGP全局使能, AS号=100, 邻居列表已记录, 路由表快照已保存
    trace_refs:
      - TP-REL-M2-001

  - step_id: STEP-002
    step_name: 对DUT1执行冷重启
    target: DUT1
    atomic_op:
      op_id: fw_reboot_device
      args:
        mode: cold
    expected_result: DUT1断电重启, 系统重新加载
    trace_refs:
      - S-REL-002-OP1

  - step_id: STEP-003
    step_name: 验证设备重启后BGP全局配置恢复
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: BGP全局配置完整恢复, AS号/路由器ID/邻居配置与重启前一致
    trace_refs:
      - S-REL-002-OP2

  - step_id: STEP-004
    step_name: 验证BGP4+邻居自动重建
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established
    trace_refs:
      - TP-REL-M2-001

  - step_id: STEP-005
    step_name: 验证路由表恢复
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args: {}
    expected_result: BGP路由表与重启前快照一致, 路由前缀/下一跳/属性完整
    trace_refs:
      - S-REL-002-OP3
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 保存当前BGP配置快照 | DUT1 | fw_verify_bgp_global | 配置/邻居/路由已记录 |
| 2 | 执行冷重启 | DUT1 | fw_reboot_device mode=cold | 设备重启 |
| 3 | 验证BGP全局配置恢复 | DUT1 | fw_verify_bgp_global | 配置完整恢复 |
| 4 | 验证邻居重建 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 邻居=Established |
| 5 | 验证路由表恢复 | DUT1 | fw_verify_bgp6_route | 路由与快照一致 |

### expected_results

1. 冷重启后 BGP 全局配置从持久化存储完整恢复 (AS号、RouterID、邻居、路由策略)
2. BGP4+ 邻居自动重建到 Established
3. 路由表恢复到重启前状态, 前缀/下一跳/属性完整
4. 数据面转发恢复正常

---

## PC-REL2-PAR-002：enable true↔false 循环50次

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL2-PAR-002 |
| `logic_case_id` | LC-REL-002 |
| `requirement_ids` | SR-REL-002 |
| `feature_tags` | BGP4+, 设备可靠性, enable循环, 反复启停 |
| `case_title` | 验证BGP enable true↔false反复切换50次后配置和邻居正常 |
| `priority` | P1 |
| `preconditions` | DUT1 BGP4+邻居已Established, BGP配置稳定 |
| `graph_ref` | SP-02 |
| `coverage_goal` | 设备级可靠性: 反复启停50次压力 |
| `trigger_data` | BGP enable true↔false 循环50次 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M2-002 |
| `scenario_refs` | S-REL-002 |
| `scenario_chain_refs` | S-REL-002 normal_path 步骤4→5 |
| `action_source_refs` | S-REL-002-OP4, S-REL-002-OP5 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认初始状态: BGP enable=true, 邻居Established
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: BGP enable=true, 邻居=Established
    trace_refs:
      - TP-REL-M2-002

  - step_id: STEP-002
    step_name: 循环执行 enable=false → enable=true 50次
    target: DUT1
    atomic_op:
      op_id: fw_toggle_bgp_enable
      args:
        cycles: 50
        enable_state: "false→true"
    expected_result: 50次启停均成功, 无异常报错
    trace_refs:
      - S-REL-002-OP4

  - step_id: STEP-003
    step_name: 验证最终BGP配置完整性
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: BGP配置完整, enable=true, AS号/路由器ID/邻居未丢失
    trace_refs:
      - TP-REL-M2-002

  - step_id: STEP-004
    step_name: 验证邻居最终状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, 无状态残留或异常
    trace_refs:
      - S-REL-002-OP5

  - step_id: STEP-005
    step_name: 验证路由表正常
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_route
      args: {}
    expected_result: 路由表正常, 无残留或异常路由
    trace_refs:
      - TP-REL-M2-002
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认初始状态 | DUT1 | fw_verify_bgp_global | enable=true, 邻居=Established |
| 2 | 循环启停50次 | DUT1 | fw_toggle_bgp_enable cycles=50 | 50次成功, 无异常 |
| 3 | 验证最终配置 | DUT1 | fw_verify_bgp_global | 配置完整无丢失 |
| 4 | 验证邻居 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 邻居=Established |
| 5 | 验证路由表 | DUT1 | fw_verify_bgp6_route | 路由正常无残留 |

### expected_results

1. 50次 enable true↔false 循环全部成功
2. 每次 enable=true 后 BGP4+ 邻居能正常重建
3. 最终 BGP 配置完整无丢失
4. 无内存泄漏、无配置残留、无状态异常

---

## PC-REL2-PAR-003：清除配置后备份恢复再重启 bgpd

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL2-PAR-003 |
| `logic_case_id` | LC-REL-002 |
| `requirement_ids` | SR-REL-002 |
| `feature_tags` | BGP4+, 设备可靠性, 配置清除, 备份恢复 |
| `case_title` | 验证清除BGP配置后从备份恢复, 重启bgpd邻居重建 |
| `priority` | P1 |
| `preconditions` | DUT1 BGP4+配置已备份 (配置文件名及内容已知) |
| `graph_ref` | SP-03 |
| `coverage_goal` | 设备级可靠性: 清除配置→备份恢复→重启bgpd |
| `trigger_data` | 清除BGP配置→从备份文件恢复→重启bgpd进程 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M2-003 |
| `scenario_refs` | S-REL-002 |
| `scenario_chain_refs` | S-REL-002 normal_path 步骤6→7→8 |
| `action_source_refs` | S-REL-002-OP6, S-REL-002-OP7, S-REL-002-OP8 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认BGP配置已备份
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: BGP全局配置已保存, 备份文件存在
    trace_refs:
      - TP-REL-M2-003

  - step_id: STEP-002
    step_name: 清除BGP配置
    target: DUT1
    atomic_op:
      op_id: delete-bgpconfig
      args: {}
    expected_result: BGP配置全量清除, 邻居全部消失
    trace_refs:
      - S-REL-002-OP6

  - step_id: STEP-003
    step_name: 从备份恢复BGP配置
    target: DUT1
    atomic_op:
      op_id: restore-bgpconfig
      args:
        source: "备份文件"
    expected_result: BGP配置从备份完整恢复
    trace_refs:
      - S-REL-002-OP7

  - step_id: STEP-004
    step_name: 重启bgpd进程使配置生效
    target: DUT1
    atomic_op:
      op_id: fw_restart_process
      args:
        process_name: "bgpd"
    expected_result: bgpd重新启动, 恢复的配置生效
    trace_refs:
      - S-REL-002-OP8

  - step_id: STEP-005
    step_name: 验证邻居重建
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, 路由恢复
    trace_refs:
      - TP-REL-M2-003
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认备份存在 | DUT1 | fw_verify_bgp_global | 备份文件存在 |
| 2 | 清除BGP配置 | DUT1 | delete-bgpconfig | 配置全量清除 |
| 3 | 从备份恢复 | DUT1 | restore-bgpconfig source=备份文件 | 配置完整恢复 |
| 4 | 重启bgpd | DUT1 | fw_restart_process process_name=bgpd | bgpd启动, 配置生效 |
| 5 | 验证邻居重建 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 邻居=Established |

### expected_results

1. 清除 BGP 配置后所有邻居断开
2. 备份恢复后配置完整还原
3. 重启 bgpd 后邻居自动重建 Established
4. 路由表恢复正常

---

## PC-REL2-PAR-004：反复启停异常路径 配置残留 [待确认]

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL2-PAR-004 |
| `logic_case_id` | LC-REL-002 |
| `requirement_ids` | SR-REL-002 |
| `feature_tags` | BGP4+, 设备可靠性, 反复启停, 配置残留, 异常路径 |
| `case_title` | 验证设备反复异常启停后配置残留行为 [待确认] |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+邻居已Established, 设备启停可控 |
| `graph_ref` | SP-04 |
| `coverage_goal` | 异常路径: 反复启停 → 配置残留 [待确认] |
| `trigger_data` | 反复异常启停设备 (模拟异常断电/强制关机场景) |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M2-004 |
| `scenario_refs` | S-REL-002 |
| `scenario_chain_refs` | S-REL-002 abnormal_path "反复启停异常" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | GAP-REL-003: 反复启停异常路径配置残留行为待确认 |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认初始状态
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: BGP配置稳定, 邻居=Established
    trace_refs:
      - TP-REL-M2-004

  - step_id: STEP-002
    step_name: "[待确认] 模拟反复异常启停 (强制断电/reboot不保存)"
    target: DUT1
    atomic_op:
      op_id: fw_reboot_device
      args:
        mode: "[待确认: force/hard]"
        cycles: "[待确认: 多次]"
    expected_result: "[待确认] 设备反复异常重启, 观察配置持久化行为"
    trace_refs:
      - TP-REL-M2-004
    confirmation_gap_refs:
      - GAP-REL-003

  - step_id: STEP-003
    step_name: "[待确认] 检查配置残留情况"
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: "[待确认] BGP配置是否残留, 是否有部分配置丢失或损坏"
    trace_refs:
      - TP-REL-M2-004
    confirmation_gap_refs:
      - GAP-REL-003

  - step_id: STEP-004
    step_name: "[待确认] 验证邻居状态"
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: "[待确认] 邻居是否能正常建立或存在异常"
    trace_refs:
      - TP-REL-M2-004
    confirmation_gap_refs:
      - GAP-REL-003
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认初始状态 | DUT1 | fw_verify_bgp_global | 配置稳定 |
| 2 | [待确认] 反复异常启停 | DUT1 | fw_reboot_device mode=[待确认] | [待确认] |
| 3 | [待确认] 检查配置残留 | DUT1 | fw_verify_bgp_global | [待确认] |
| 4 | [待确认] 验证邻居状态 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | [待确认] |

### expected_results

[待确认] 需确认反复异常启停后的系统行为：
- 配置文件是否完整 (无写入到一半的损坏配置)
- 是否存在 "幽灵配置" (已删除但仍在内存中生效)
- 是否有配置不一致 (内存 vs 持久化存储)
- 异常启停后是否需要手动清理或恢复

---
