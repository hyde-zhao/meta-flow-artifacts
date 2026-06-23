# BGP4+ 路由冗余 — 静态路由与 BGP4+ 互备 — 物理用例

> 生成时间: 2026-06-12 | 设计Skill: state-design
> 逻辑用例: LC-FW-002 | 目录: BGP-业务转发-路由转发

---

## PC-FW-002-01：静态路由在线 — AD优先级验证

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW-002-01 |
| `logic_case_id` | LC-FW-002 |
| `requirement_ids` | SR-012 |
| `feature_tags` | BGP4+, 路由冗余, AD优先级, 互备 |
| `case_title` | 验证同目的静态路由(AD=1)和IBGP路由(AD=200)同时存在时优选静态路由 |
| `priority` | P0 |
| `preconditions` | DUT1-DUT2 IBGP邻居Established; DUT1已配置静态路由和IBGP路由指向相同目的 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 主生命周期: AD优先级验证 |
| `trigger_data` | 静态AD=1, IBGP AD=200, 同目的前缀 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-F-M2-STATIC-PRI-001, LC-FW-002 P1前段 |
| `scenario_refs` | S10 |
| `scenario_chain_refs` | LC-FW-002 P1前段 |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: DUT1配置静态IPv6路由指向目的网络
    target: DUT1
    atomic_op:
      op_id: add-static-route
      args:
        prefix: "2001:db8:200::/48"
        nextHop: "<TG_IPv6>"
        adminDistance: 1
    expected_result: 静态路由添加成功
    trace_refs:
      - TP-F-M2-STATIC-PRI-001

  - step_id: STEP-002
    step_name: DUT2宣告相同目的网络的BGP路由
    target: DUT2
    atomic_op:
      op_id: post-bgp6network
      args:
        network: "2001:db8:200::/48"
    expected_result: BGP路由通告成功, DUT1学到IBGP路由(AD=200)
    trace_refs:
      - TP-F-M2-STATIC-PRI-001

  - step_id: STEP-003
    step_name: 查看DUT1路由表验证静态路由优先
    target: DUT1
    atomic_op:
      op_id: get-route-table
      args:
        prefix: "2001:db8:200::/48"
    expected_result: 路由表中仅有静态路由(AD=1)生效, BGP路由(AD=200)为备份(不生效)
    trace_refs:
      - TP-F-M2-STATIC-PRI-001

  - step_id: STEP-004
    step_name: TG发送流量验证走静态路由转发
    target: TG
    atomic_op:
      op_id: send-traffic
      args:
        dst: "2001:db8:200::1"
    expected_result: 流量通过静态路由下一跳转发, 到达TG
    trace_refs:
      - LC-FW-002 P1
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 配置静态路由(AD=1) | DUT1 | add-static-route prefix=2001:db8:200::/48, nextHop=<TG_IPv6> | 添加成功 |
| 2 | DUT2宣告相同BGP路由 | DUT2 | post-bgp6network network=2001:db8:200::/48 | BGP通告成功 |
| 3 | 验证静态路由优先 | DUT1 | get-route-table prefix=2001:db8:200::/48 | 静态生效, BGP备份 |
| 4 | 流量走静态转发 | TG | send-traffic dst=2001:db8:200::1 | 流量到达TG |

### expected_results

1. 同目的静态(AD=1)和IBGP(AD=200)路由同时存在于RIB
2. 路由表优选静态路由(AD最小)
3. 流量通过静态路由下一跳转发

---

## PC-FW-002-02：静态失效 — 切换BGP转发

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW-002-02 |
| `logic_case_id` | LC-FW-002 |
| `requirement_ids` | SR-012 |
| `feature_tags` | BGP4+, 路由冗余, 故障切换 |
| `case_title` | 验证删除静态路由后自动切换到BGP路由转发, 中断≤5s |
| `priority` | P0 |
| `preconditions` | 静态路由(AD=1)在线, IBGP路由(AD=200)在RIB中备份 |
| `graph_ref` | SP-02 |
| `coverage_goal` | 回退切换: T1 (S0→S1) |
| `trigger_data` | 删除静态路由 |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | TP-Q-REL-014, LC-FW-002 P1 |
| `scenario_refs` | S10, S08 |
| `scenario_chain_refs` | LC-FW-002 P1 |
| `action_source_refs` | — |
| `confirmation_gap_refs` | GAP-FW2-001: 中断≤5s, GAP-FW2-002: BGP预装 |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认静态路由在线且流量走静态转发
    target: DUT1, TG
    atomic_op:
      op_id: get-route-table-and-traffic
      args:
        prefix: "2001:db8:200::/48"
    expected_result: FIB下一跳=静态路由, TG流量走静态
    trace_refs:
      - LC-FW-002 P1

  - step_id: STEP-002
    step_name: TG开始持续发送流量并记录时间T0
    target: TG
    atomic_op:
      op_id: start-continuous-traffic
      args:
        dst: "2001:db8:200::1"
    expected_result: 持续流量已启动, 记录T0
    trace_refs:
      - LC-FW-002 P1

  - step_id: STEP-003
    step_name: 删除静态路由
    target: DUT1
    atomic_op:
      op_id: delete-static-route
      args:
        prefix: "2001:db8:200::/48"
    expected_result: 静态路由删除成功
    trace_refs:
      - LC-FW-002 P1

  - step_id: STEP-004
    step_name: 验证路由表切换到BGP路由
    target: DUT1
    atomic_op:
      op_id: get-route-table
      args:
        prefix: "2001:db8:200::/48"
    expected_result: FIB下一跳= BGP邻居, 路由来源=BGP(AD=200)
    trace_refs:
      - LC-FW-002 P1

  - step_id: STEP-005
    step_name: 验证流量恢复且中断时间≤5s
    target: TG
    atomic_op:
      op_id: check-traffic-recovery
      args:
        dst: "2001:db8:200::1"
    expected_result: "[待确认] 流量恢复, 中断时长 ≤ 5s"
    trace_refs:
      - TP-Q-REL-014
      - LC-FW-002 P1
    confirmation_gap_refs:
      - GAP-FW2-001
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认静态在线 | DUT1, TG | get-route-table-and-traffic | 静态转发 |
| 2 | TG持续流量+记录T0 | TG | start-continuous-traffic | T0记录 |
| 3 | 删除静态路由 | DUT1 | delete-static-route prefix=2001:db8:200::/48 | 删除成功 |
| 4 | 验证切BGP | DUT1 | get-route-table | FIB=BGP邻居 |
| 5 | 验证中断≤5s | TG | check-traffic-recovery | [待确认] 中断≤5s |

### expected_results

1. 删除静态路由后RIB移除静态路由条目
2. RIB安装次优BGP路由(AD=200)到FIB
3. 流量从静态下一跳切换到BGP下一跳
4. [待确认] 流量中断时间 ≤ 5s

---

## PC-FW-002-03：静态恢复 — 切回静态转发

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW-002-03 |
| `logic_case_id` | LC-FW-002 |
| `requirement_ids` | SR-012 |
| `feature_tags` | BGP4+, 路由冗余, 恢复切换 |
| `case_title` | 验证重新添加静态路由后路由表切回静态, 流量恢复走静态转发 |
| `priority` | P0 |
| `preconditions` | 静态路由已删除, 当前走BGP路由转发 |
| `graph_ref` | SP-03 |
| `coverage_goal` | 恢复切换: T2 (S1→S0) |
| `trigger_data` | 重新添加静态路由 (同目的) |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | LC-FW-002 P2 |
| `scenario_refs` | S10 |
| `scenario_chain_refs` | LC-FW-002 P2 |
| `action_source_refs` | — |
| `confirmation_gap_refs` | GAP-FW2-001 |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认当前走BGP路由转发
    target: DUT1
    atomic_op:
      op_id: get-route-table
      args:
        prefix: "2001:db8:200::/48"
    expected_result: FIB下一跳=BGP邻居
    trace_refs:
      - LC-FW-002 P2

  - step_id: STEP-002
    step_name: 重新添加静态路由(同目的, AD=1)
    target: DUT1
    atomic_op:
      op_id: add-static-route
      args:
        prefix: "2001:db8:200::/48"
        nextHop: "<TG_IPv6>"
        adminDistance: 1
    expected_result: 静态路由添加成功
    trace_refs:
      - LC-FW-002 P2

  - step_id: STEP-003
    step_name: 验证路由表立即切回静态
    target: DUT1
    atomic_op:
      op_id: get-route-table
      args:
        prefix: "2001:db8:200::/48"
    expected_result: FIB下一跳=静态路由下一跳, BGP路由退回备份
    trace_refs:
      - LC-FW-002 P2

  - step_id: STEP-004
    step_name: 验证流量恢复走静态转发
    target: TG
    atomic_op:
      op_id: send-traffic
      args:
        dst: "2001:db8:200::1"
    expected_result: 流量通过静态路由转发到达TG
    trace_refs:
      - LC-FW-002 P2
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认BGP转发 | DUT1 | get-route-table prefix=2001:db8:200::/48 | FIB=BGP |
| 2 | 重新添加静态路由 | DUT1 | add-static-route prefix=2001:db8:200::/48, AD=1 | 添加成功 |
| 3 | 验证切回静态 | DUT1 | get-route-table | FIB=静态下一跳 |
| 4 | 验证流量走静态 | TG | send-traffic dst=2001:db8:200::1 | 流量到达TG |

### expected_results

1. 重新添加静态路由后，RIB检测到AD=1的静态路由
2. AD=1 < AD=200，静态路由替换BGP路由安装到FIB
3. 流量自动从BGP下一跳切回静态下一跳

---

## PC-FW-002-07：异常退化 — 静态失效+BGP断开

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-FW-002-07 |
| `logic_case_id` | LC-FW-002 |
| `requirement_ids` | SR-012 |
| `feature_tags` | BGP4+, 路由冗余, 异常退化 |
| `case_title` | 验证静态路由失效且BGP邻居断开时导致路由黑洞 |
| `priority` | P2 |
| `preconditions` | 静态路由已删除, 当前走BGP路由转发, BGP邻居 Established |
| `graph_ref` | SP-05 |
| `coverage_goal` | 异常退化: T3 (S1→S2) |
| `trigger_data` | BGP邻居断开 (enable=false或链路down) |
| `topology_binding_refs` | node3_dut2_tg1_link3 |
| `trace_refs` | LC-FW-002 P3扩展 |
| `scenario_refs` | S08 |
| `scenario_chain_refs` | — |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认静态已删除, BGP转发中
    target: DUT1
    atomic_op:
      op_id: get-route-table
      args:
        prefix: "2001:db8:200::/48"
    expected_result: FIB=BGP邻居
    trace_refs:
      - LC-FW-002 P3

  - step_id: STEP-002
    step_name: 断开BGP邻居
    target: DUT1
    atomic_op:
      op_id: put-bgpconfig
      args:
        enable: false
    expected_result: BGP邻居断开, BGP路由撤销
    trace_refs:
      - LC-FW-002 P3

  - step_id: STEP-003
    step_name: 验证路由表无该目的路由
    target: DUT1
    atomic_op:
      op_id: get-route-table
      args:
        prefix: "2001:db8:200::/48"
    expected_result: 路由表中无2001:db8:200::/48条目 (路由黑洞)
    trace_refs:
      - LC-FW-002 P3

  - step_id: STEP-004
    step_name: TG发送流量验证丢包
    target: TG
    atomic_op:
      op_id: send-traffic
      args:
        dst: "2001:db8:200::1"
    expected_result: 流量全部丢失 (无路由)
    trace_refs:
      - LC-FW-002 P3
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认BGP转发 | DUT1 | get-route-table prefix=2001:db8:200::/48 | FIB=BGP |
| 2 | 断开BGP邻居 | DUT1 | put-bgpconfig enable=false | BGP断开 |
| 3 | 验证路由黑洞 | DUT1 | get-route-table | 无该目的路由 |
| 4 | 验证流量丢包 | TG | send-traffic dst=2001:db8:200::1 | 流量全部丢失 |

### expected_results

1. 静态路由已失效, BGP路由为主要转发路径
2. BGP邻居断开后BGP路由被撤销
3. 无任何路由指向目的, 形成路由黑洞
4. 所有流量丢失
