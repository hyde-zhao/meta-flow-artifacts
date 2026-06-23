# BGP4+ HA 非VRRP 开销控制 — 物理用例

> 生成时间: 2026-06-12 | 设计Skill: state-design
> 逻辑用例: LC-HA-002 | 目录: BGP-跨模块-HA场景

---

## PC-HA-002-01：非VRRP HA初始双机开销验证

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-HA-002-01 |
| `logic_case_id` | LC-HA-002 |
| `requirement_ids` | SR-007, SR-011 |
| `feature_tags` | BGP4+, HA, 非VRRP, 开销控制 |
| `case_title` | 验证非VRRP HA模式下主设备路由开销=normal, 备设备路由开销=max(4294967295) |
| `priority` | P0 |
| `preconditions` | DUT1(Master)/DUT2(Backup)分别独立建立BGP4+邻居; 无VRRP配置; 四种路由类型已配置 |
| `graph_ref` | SP-01 |
| `coverage_goal` | 主生命周期: 初始双机开销验证 |
| `trigger_data` | HA角色: DUT1=Master(开销=normal), DUT2=Backup(开销=max) |
| `topology_binding_refs` | node4_dut2_tg1_sw1_link6 |
| `trace_refs` | TP-F-M3-HA-002, LC-HA-002 P1 |
| `scenario_refs` | S20 |
| `scenario_chain_refs` | LC-HA-002 P1 |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 配置DUT1为Master角色并独立建立BGP邻居
    target: DUT1
    atomic_op:
      op_id: post-bgp6neighbor
      args:
        ip: "<TG_IPv6>"
        asNumber: 100
        interface: "GE0_1"
    expected_result: 邻居Established
    trace_refs:
      - TP-F-M3-HA-002

  - step_id: STEP-002
    step_name: 配置DUT2为Backup角色并独立建立BGP邻居
    target: DUT2
    atomic_op:
      op_id: post-bgp6neighbor
      args:
        ip: "<TG_IPv6>"
        asNumber: 100
    expected_result: 邻居Established
    trace_refs:
      - TP-F-M3-HA-002

  - step_id: STEP-003
    step_name: DUT1宣告网络路由+引入直连/静态/OSPFv3路由
    target: DUT1
    atomic_op:
      op_id: put-bgp6network-and-redist
      args:
        network: "2001:db8:100::/48"
        redist6Route:
          - protocol: "connected"
          - protocol: "static"
          - protocol: "ospfv3"
    expected_result: 四种路由类型通告成功, 开销=normal
    trace_refs:
      - LC-HA-002 P1

  - step_id: STEP-004
    step_name: 验证对端路由表优先选择DUT1 (开销低)
    target: TG (对端)
    atomic_op:
      op_id: get-bgp6route
      args: {}
    expected_result: 对端路由表选路指向DUT1 (开销=normal < DUT2开销=max)
    trace_refs:
      - LC-HA-002 P1

  - step_id: STEP-005
    step_name: 验证DUT2四种路由类型开销均为max(4294967295)
    target: TG (对端观察DUT2路由)
    atomic_op:
      op_id: get-bgp6route-detail
      args: {}
    expected_result: DUT2的MED/开销=4294967295 (四种路由类型均如此)
    trace_refs:
      - LC-HA-002 P1
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | DUT1建立BGP邻居 | DUT1 | post-bgp6neighbor | Established |
| 2 | DUT2建立BGP邻居 | DUT2 | post-bgp6neighbor | Established |
| 3 | DUT1宣告四种路由 | DUT1 | put-bgp6network-and-redist | 四种路由开销=normal |
| 4 | 对端选路验证 | TG | get-bgp6route | 优选DUT1 |
| 5 | DUT2开销验证 | TG | get-bgp6route-detail | 四种路由开销=4294967295 |

### expected_results

1. DUT1(Master)四种路由类型开销均为正常业务值(normal)
2. DUT2(Backup)四种路由类型开销均为最大值(4294967295)
3. 对端选路算法优先选择 DUT1
4. 四种路由类型: 网络宣告/直连引入/静态引入/OSPFv3引入均符合开销预期

---

## PC-HA-002-02：主降备 — 开销 normal→max

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-HA-002-02 |
| `logic_case_id` | LC-HA-002 |
| `requirement_ids` | SR-007, SR-011 |
| `feature_tags` | BGP4+, HA, 非VRRP, 开销控制, 主降备 |
| `case_title` | 验证DUT1 Master→Backup后四种路由类型开销全部变为最大值4294967295 |
| `priority` | P0 |
| `preconditions` | DUT1=Master(开销=normal), DUT2=Backup(开销=max), 四种路由类型已配置 |
| `graph_ref` | SP-02 |
| `coverage_goal` | 主降备: T1 (S0→S1) |
| `trigger_data` | HA角色切换: DUT1 Master→Backup |
| `topology_binding_refs` | node4_dut2_tg1_sw1_link6 |
| `trace_refs` | LC-HA-002 P2 |
| `scenario_refs` | S20 |
| `scenario_chain_refs` | LC-HA-002 P2 |
| `action_source_refs` | — |
| `confirmation_gap_refs` | GAP-HA2-001: 转换原子性; GAP-HA2-002: 收敛时间 |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认初始状态: DUT1=Master(开销normal)
    target: TG (对端)
    atomic_op:
      op_id: get-bgp6route-detail
      args: {}
    expected_result: DUT1四种路由类型开销=normal
    trace_refs:
      - LC-HA-002 P1

  - step_id: STEP-002
    step_name: 触发DUT1 Master→Backup角色切换
    target: DUT1
    atomic_op:
      op_id: set-ha-role
      args:
        role: "Backup"
    expected_result: DUT1 HA角色变更为Backup
    trace_refs:
      - LC-HA-002 P2

  - step_id: STEP-003
    step_name: 验证DUT1四种路由类型开销全部变为max
    target: TG (对端)
    atomic_op:
      op_id: get-bgp6route-detail
      args: {}
    expected_result: 网络宣告开销=4294967295, 直连引入开销=4294967295, 静态引入开销=4294967295, OSPFv3引入开销=4294967295
    trace_refs:
      - LC-HA-002 P2

  - step_id: STEP-004
    step_name: "[待确认] 验证四种路由类型开销同时完成转换"
    target: TG (对端)
    atomic_op:
      op_id: get-bgp6route-timing
      args: {}
    expected_result: "[待确认] 四种路由类型开销更新在同一个BGP UPDATE中 或 分批更新但时间差可接受"
    trace_refs:
      - LC-HA-002 P2
    confirmation_gap_refs:
      - GAP-HA2-001
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认初始状态 | TG | get-bgp6route-detail | DUT1四种路由开销=normal |
| 2 | 触发DUT1 Master→Backup | DUT1 | set-ha-role role=Backup | 角色变更 |
| 3 | 验证四种路由开销→max | TG | get-bgp6route-detail | 全部=4294967295 |
| 4 | [待确认] 验证转换原子性 | TG | get-bgp6route-timing | [待确认] |

### expected_results

1. DUT1角色变为Backup后, 四种路由类型开销全部变为4294967295
2. 对端收到BGP UPDATE消息更新MED/开销值
3. [待确认] 转换是否原子完成

---

## PC-HA-002-03：备升主 — 开销 max→normal

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-HA-002-03 |
| `logic_case_id` | LC-HA-002 |
| `requirement_ids` | SR-007, SR-011 |
| `feature_tags` | BGP4+, HA, 非VRRP, 开销控制, 备升主 |
| `case_title` | 验证DUT2 Backup→Master后四种路由类型开销全部恢复为正常值 |
| `priority` | P0 |
| `preconditions` | DUT1=Backup(开销=max), DUT2=Backup(开销=max) — 即主降备后场景 |
| `graph_ref` | SP-03 |
| `coverage_goal` | 备升主: T2 (S1→S0) |
| `trigger_data` | HA角色切换: DUT2 Backup→Master |
| `topology_binding_refs` | node4_dut2_tg1_sw1_link6 |
| `trace_refs` | LC-HA-002 P3 |
| `scenario_refs` | S20 |
| `scenario_chain_refs` | LC-HA-002 P3 |
| `action_source_refs` | — |
| `confirmation_gap_refs` | GAP-HA2-001, GAP-HA2-002 |
| `fact_status` | needs-confirmation |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: 确认DUT1已降备(开销=max), DUT2仍为Backup(开销=max)
    target: TG (对端)
    atomic_op:
      op_id: get-bgp6route-detail
      args: {}
    expected_result: DUT1开销=max, DUT2开销=max
    trace_refs:
      - LC-HA-002 P2

  - step_id: STEP-002
    step_name: 触发DUT2 Backup→Master角色切换
    target: DUT2
    atomic_op:
      op_id: set-ha-role
      args:
        role: "Master"
    expected_result: DUT2 HA角色变更为Master
    trace_refs:
      - LC-HA-002 P3

  - step_id: STEP-003
    step_name: 验证DUT2四种路由类型开销全部恢复为normal
    target: TG (对端)
    atomic_op:
      op_id: get-bgp6route-detail
      args: {}
    expected_result: 网络宣告/直连引入/静态引入/OSPFv3引入开销全部=normal
    trace_refs:
      - LC-HA-002 P3

  - step_id: STEP-004
    step_name: 验证对端选路自动切换到DUT2
    target: TG (对端)
    atomic_op:
      op_id: get-bgp6route
      args: {}
    expected_result: "[待确认] 对端路由表优选DUT2 (开销normal), 收敛时间[待确认]"
    trace_refs:
      - LC-HA-002 P3
    confirmation_gap_refs:
      - GAP-HA2-002
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | 确认DUT1/DUT2均为Backup | TG | get-bgp6route-detail | 两端开销=max |
| 2 | 触发DUT2升为Master | DUT2 | set-ha-role role=Master | 角色变更 |
| 3 | 验证四种路由开销→normal | TG | get-bgp6route-detail | 全部=normal |
| 4 | 验证对端选路切换 | TG | get-bgp6route | [待确认] 优选DUT2 |

### expected_results

1. DUT2角色变为Master后, 四种路由开销全部恢复为normal
2. 对端收到BGP UPDATE, 选路自动从(可能已无normal路由)切换到DUT2
3. [待确认] 收敛时间和选路切换行为

---

## PC-HA-002-05：路由类型覆盖 — 网络宣告路由

| 字段 | 值 |
|------|-----|
| `physical_case_id` | PC-HA-002-05 |
| `logic_case_id` | LC-HA-002 |
| `requirement_ids` | SR-008, SR-011 |
| `feature_tags` | BGP4+, HA, 网络宣告, 开销控制 |
| `case_title` | 验证网络宣告路由在主备角色切换时开销值正确变化 |
| `priority` | P1 |
| `preconditions` | DUT1=Master, 已宣告网络路由 2001:db8:100::/48 |
| `graph_ref` | SP-05 |
| `coverage_goal` | 路由类型覆盖: 网络宣告路由 |
| `trigger_data` | 网络宣告路由: network="2001:db8:100::/48" |
| `topology_binding_refs` | node4_dut2_tg1_sw1_link6 |
| `trace_refs` | LC-HA-002 P4 网络宣告 |
| `scenario_refs` | S20 |
| `scenario_chain_refs` | LC-HA-002 P4 |
| `action_source_refs` | — |
| `confirmation_gap_refs` | — |
| `fact_status` | confirmed |

### case_steps

```yaml
case_steps:
  - step_id: STEP-001
    step_name: DUT1(Master)宣告网络路由
    target: DUT1
    atomic_op:
      op_id: post-bgp6network
      args:
        network: "2001:db8:100::/48"
    expected_result: API返回success
    trace_refs:
      - LC-HA-002 P4

  - step_id: STEP-002
    step_name: 验证对端看到的网络宣告路由开销=normal
    target: TG (对端)
    atomic_op:
      op_id: get-bgp6route-detail
      args:
        prefix: "2001:db8:100::/48"
    expected_result: 开销=normal (非4294967295)
    trace_refs:
      - LC-HA-002 P4

  - step_id: STEP-003
    step_name: 触发主降备并验证网络宣告开销→max
    target: DUT1→TG
    atomic_op:
      op_id: set-ha-role-and-verify
      args:
        role: "Backup"
        prefix: "2001:db8:100::/48"
    expected_result: 网络宣告路由开销=4294967295
    trace_refs:
      - LC-HA-002 P4

  - step_id: STEP-004
    step_name: 触发备升主并验证网络宣告开销→normal
    target: DUT2→TG
    atomic_op:
      op_id: set-ha-role-and-verify
      args:
        role: "Master"
        prefix: "2001:db8:100::/48"
    expected_result: 网络宣告路由开销=normal
    trace_refs:
      - LC-HA-002 P4
```

### test_steps

| 步骤 | 操作描述 | 执行对象 | 原子操作 | 预期结果 |
|------|---------|---------|---------|---------|
| 1 | DUT1宣告网络路由 | DUT1 | post-bgp6network network=2001:db8:100::/48 | API success |
| 2 | 验证开销=normal (Master) | TG | get-bgp6route-detail prefix=2001:db8:100::/48 | 开销=normal |
| 3 | 主降备验证开销→max | DUT1→TG | set-ha-role role=Backup | 开销=4294967295 |
| 4 | 备升主验证开销→normal | DUT2→TG | set-ha-role role=Master | 开销=normal |

### expected_results

1. Master状态网络宣告路由开销为正常值
2. Backup状态网络宣告路由开销为最大值4294967295
3. 角色切换后网络宣告路由开销正确跟随变化
