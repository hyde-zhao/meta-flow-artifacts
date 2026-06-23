# BGP4+ 进程级可靠性 — 物理用例

> 生成时间: 2026-06-12 | 设计Skill: parameter-design
> 逻辑用例: LC-REL-001 | 目录: BGP-协议协商-状态协商

---

## PC-REL1-PAR-001：kill bgpd 后 Watchfrr 自动拉起邻居重建 ≤5s

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL1-PAR-001 |
| `logic_case_id` | LC-REL-001 |
| `requirement_ids` | SR-REL-001 |
| `feature_tags` | BGP4+, 进程可靠性, Watchfrr, 自动恢复 |
| `case_title` | 验证kill bgpd进程后Watchfrr自动拉起, 邻居重建时间≤5s |
| `priority` | P1 |
| `preconditions` | DUT1/DUT2 BGP4+邻居已Established, Watchfrr监控bgpd正常运行 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 进程级可靠性: kill bgpd → Watchfrr自动恢复, 邻居重建 |
| `trigger_data` | kill bgpd进程, Watchfrr检测并拉起, 恢复时间阈值≤5s |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M1-001 |
| `scenario_refs` | S-REL-001 |
| `scenario_chain_refs` | S-REL-001 normal_path 步骤1→2→3 |
| `action_source_refs` | S-REL-001-OP1, S-REL-001-OP2 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认bgpd进程运行且邻居Established
    target: DUT1
    atomic_op:
      op_id: fw_verify_process
      args:
        process_name: "bgpd"
    expected_result: bgpd进程状态=running, 邻居状态=Established
    trace_refs:
      - TP-REL-M1-001

  - step_id: STEP-002
    step_name: kill bgpd进程
    target: DUT1
    atomic_op:
      op_id: fw_kill_process
      args:
        process_name: "bgpd"
    expected_result: bgpd进程被终止
    trace_refs:
      - S-REL-001-OP1

  - step_id: STEP-003
    step_name: 验证Watchfrr自动拉起bgpd
    target: DUT1
    atomic_op:
      op_id: fw_verify_process
      args:
        process_name: "bgpd"
    expected_result: bgpd进程由Watchfrr自动拉起, 状态=running
    trace_refs:
      - TP-REL-M1-001

  - step_id: STEP-004
    step_name: 验证邻居重建且恢复时间≤5s
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, 从kill到重建总时间≤5s
    trace_refs:
      - TP-REL-M1-001
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认bgpd进程运行且邻居Established | DUT1 | fw_verify_process process_name=bgpd | bgpd=running, 邻居=Established |
| 2 | kill bgpd进程 | DUT1 | fw_kill_process process_name=bgpd | bgpd被终止 |
| 3 | 验证Watchfrr自动拉起bgpd | DUT1 | fw_verify_process process_name=bgpd | bgpd自动拉起=running |
| 4 | 验证邻居重建且≤5s | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 邻居=Established, ≤5s |

### expected_results

1. Watchfrr 在检测到 bgpd 异常终止后自动拉起 bgpd 进程
2. bgpd 重新启动后保留原有 BGP 配置
3. BGP4+ 邻居自动重建到 Established
4. 从 kill 到邻居 Established 总恢复时间 ≤5s

---

## PC-REL1-PAR-002：kill FRR 后 Watchfrr 拉起从 frrinc.conf 恢复配置

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL1-PAR-002 |
| `logic_case_id` | LC-REL-001 |
| `requirement_ids` | SR-REL-001 |
| `feature_tags` | BGP4+, 进程可靠性, FRR, Watchfrr, 配置恢复 |
| `case_title` | 验证kill FRR进程后Watchfrr拉起并从frrinc.conf恢复BGP配置 |
| `priority` | P1 |
| `preconditions` | DUT1 BGP4+已配置(邻居+路由策略), frrinc.conf持久化正常 |
| `graph_ref` | SP-02 |
| `coverage_goal` | 进程级可靠性: kill FRR → Watchfrr恢复, 配置从frrinc.conf恢复 |
| `trigger_data` | kill FRR进程(含bgpd), Watchfrr检测并拉起, 从frrinc.conf恢复配置 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M1-002 |
| `scenario_refs` | S-REL-001 |
| `scenario_chain_refs` | S-REL-001 normal_path 步骤4→5 |
| `action_source_refs` | S-REL-001-OP3, S-REL-001-OP4 |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认FRR和bgpd进程运行, BGP配置已写入frrinc.conf
    target: DUT1
    atomic_op:
      op_id: fw_verify_process
      args:
        process_name: "frr"
    expected_result: FRR进程=running, bgpd进程=running, frrinc.conf含BGP配置
    trace_refs:
      - TP-REL-M1-002

  - step_id: STEP-002
    step_name: kill FRR进程
    target: DUT1
    atomic_op:
      op_id: fw_kill_process
      args:
        process_name: "frr"
    expected_result: FRR (含bgpd)进程被终止
    trace_refs:
      - S-REL-001-OP3

  - step_id: STEP-003
    step_name: 验证Watchfrr自动拉起FRR
    target: DUT1
    atomic_op:
      op_id: fw_verify_process
      args:
        process_name: "frr"
    expected_result: FRR进程由Watchfrr自动拉起, bgpd=running
    trace_refs:
      - TP-REL-M1-002

  - step_id: STEP-004
    step_name: 验证BGP配置从frrinc.conf恢复
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp_global
      args: {}
    expected_result: BGP全局配置与frrinc.conf一致, AS号/路由器ID/邻居配置恢复
    trace_refs:
      - S-REL-001-OP4

  - step_id: STEP-005
    step_name: 验证邻居自动重建
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: 邻居状态=Established, 路由恢复
    trace_refs:
      - TP-REL-M1-002
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认FRR运行且配置持久化 | DUT1 | fw_verify_process process_name=frr | FRR=running, frrinc.conf含BGP配置 |
| 2 | kill FRR进程 | DUT1 | fw_kill_process process_name=frr | FRR被终止 |
| 3 | 验证Watchfrr拉起FRR | DUT1 | fw_verify_process process_name=frr | FRR自动拉起=running |
| 4 | 验证BGP配置恢复 | DUT1 | fw_verify_bgp_global | 配置与frrinc.conf一致 |
| 5 | 验证邻居重建 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | 邻居=Established |

### expected_results

1. Watchfrr 在 FRR 终止后自动拉起 FRR 进程
2. bgpd 从 frrinc.conf 恢复所有 BGP 配置 (AS号、邻居、路由策略)
3. BGP4+ 邻居自动重建到 Established
4. 路由表恢复正常

---

## PC-REL1-PAR-003：kill bgpd 恢复时间 >5s 边界 [待确认]

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL1-PAR-003 |
| `logic_case_id` | LC-REL-001 |
| `requirement_ids` | SR-REL-001 |
| `feature_tags` | BGP4+, 进程可靠性, 恢复时间, 边界 |
| `case_title` | 验证kill bgpd后恢复时间>5s边界行为 [待确认] |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+邻居已Established, 大配置量/大路由量加大恢复时间 |
| `graph_ref` | SP-03 |
| `coverage_goal` | 边界守卫: 恢复时间>5s [待确认] |
| `trigger_data` | kill bgpd进程, 通过大配置量/大路由量模拟恢复时间>5s |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M1-003 |
| `scenario_refs` | S-REL-001 |
| `scenario_chain_refs` | S-REL-001 abnormal_path "恢复时间>5s" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | GAP-REL-001: 恢复时间>5s 行为待确认 |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: "[待确认] 配置大量邻居和路由以加大bgpd重启时间"
    target: DUT1
    atomic_op:
      op_id: fw_create_bgp6_neighbor
      args:
        count: "[待确认: 多个邻居+多条路由]"
    expected_result: "[待确认] 邻居和路由已配置, 重启时间>5s"
    trace_refs:
      - TP-REL-M1-003
    confirmation_gap_refs:
      - GAP-REL-001

  - step_id: STEP-002
    step_name: "[待确认] kill bgpd进程并计时恢复"
    target: DUT1
    atomic_op:
      op_id: fw_kill_process
      args:
        process_name: "bgpd"
    expected_result: "[待确认] 恢复时间>5s, 邻居重建中"
    trace_refs:
      - TP-REL-M1-003
    confirmation_gap_refs:
      - GAP-REL-001

  - step_id: STEP-003
    step_name: "[待确认] 观察恢复时间>5s时的系统行为"
    target: DUT1
    atomic_op:
      op_id: fw_verify_bgp6_neighbor
      args:
        id: "2001:db8:1::2"
    expected_result: "[待确认] 恢复时间>5s时邻居最终仍能建立, 或进入异常状态"
    trace_refs:
      - TP-REL-M1-003
    confirmation_gap_refs:
      - GAP-REL-001
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | [待确认] 配置大容量邻居/路由 | DUT1 | fw_create_bgp6_neighbor count=[待确认] | [待确认] 恢复时间>5s |
| 2 | [待确认] kill bgpd并计时 | DUT1 | fw_kill_process process_name=bgpd | [待确认] 恢复时间>5s |
| 3 | [待确认] 观察超时行为 | DUT1 | fw_verify_bgp6_neighbor id=2001:db8:1::2 | [待确认] |

### expected_results

[待确认] 需确认恢复时间>5s时的系统行为：
- 邻居最终是否仍能成功建立 Established
- 是否有告警或日志记录
- 恢复时间超长 (>30s) 时 Watchfrr 是否持续重试

---

## PC-REL1-PAR-004：Watchfrr 异常时 kill bgpd 不恢复 [待确认]

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-REL1-PAR-004 |
| `logic_case_id` | LC-REL-001 |
| `requirement_ids` | SR-REL-001 |
| `feature_tags` | BGP4+, 进程可靠性, Watchfrr异常, 边界 |
| `case_title` | 验证Watchfrr异常时kill bgpd不自动恢复 [待确认] |
| `priority` | P2 |
| `preconditions` | DUT1 BGP4+邻居已Established, Watchfrr状态可控 |
| `graph_ref` | SP-04 |
| `coverage_goal` | 异常路径: Watchfrr异常 → kill bgpd不恢复 [待确认] |
| `trigger_data` | kill Watchfrr 或使其异常, 再kill bgpd |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-REL-M1-004 |
| `scenario_refs` | S-REL-001 |
| `scenario_chain_refs` | S-REL-001 abnormal_path "Watchfrr异常" |
| `action_source_refs` | — |
| `confirmation_gap_refs` | GAP-REL-002: Watchfrr异常时 bgpd 不恢复行为待确认 |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认Watchfrr正常运行且监控bgpd
    target: DUT1
    atomic_op:
      op_id: fw_verify_process
      args:
        process_name: "watchfrr"
    expected_result: Watchfrr进程=running, 监控bgpd
    trace_refs:
      - TP-REL-M1-004

  - step_id: STEP-002
    step_name: "[待确认] 使Watchfrr异常 (kill watchfrr或停止监控)"
    target: DUT1
    atomic_op:
      op_id: fw_kill_process
      args:
        process_name: "watchfrr"
    expected_result: "[待确认] Watchfrr进程异常/停止, bgpd失去守护"
    trace_refs:
      - TP-REL-M1-004
    confirmation_gap_refs:
      - GAP-REL-002

  - step_id: STEP-003
    step_name: "[待确认] kill bgpd进程并观察恢复行为"
    target: DUT1
    atomic_op:
      op_id: fw_kill_process
      args:
        process_name: "bgpd"
    expected_result: "[待确认] bgpd被kill后不自动恢复, 邻居永久中断"
    trace_refs:
      - TP-REL-M1-004
    confirmation_gap_refs:
      - GAP-REL-002

  - step_id: STEP-004
    step_name: "[待确认] 验证系统告警或日志记录"
    target: DUT1
    atomic_op:
      op_id: get-system-log
      args:
        filter: "bgpd|watchfrr"
    expected_result: "[待确认] 系统产生告警, 记录bgpd未恢复原因"
    trace_refs:
      - TP-REL-M1-004
    confirmation_gap_refs:
      - GAP-REL-002
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认Watchfrr正常运行 | DUT1 | fw_verify_process process_name=watchfrr | Watchfrr=running |
| 2 | [待确认] 使Watchfrr异常 | DUT1 | fw_kill_process process_name=watchfrr | [待确认] Watchfrr异常 |
| 3 | [待确认] kill bgpd观察恢复 | DUT1 | fw_kill_process process_name=bgpd | [待确认] bgpd不自动恢复 |
| 4 | [待确认] 验证告警 | DUT1 | get-system-log filter=bgpd|watchfrr | [待确认] 有告警记录 |

### expected_results

[待确认] 需确认 Watchfrr 异常时的系统行为：
- bgpd 被 kill 后是否完全无法自动恢复
- 系统是否产生告警通知运维人员
- 是否有手动恢复途径 (手动启动 bgpd)
- 恢复 Watchfrr 后 bgpd 是否能被重新拉起

---
