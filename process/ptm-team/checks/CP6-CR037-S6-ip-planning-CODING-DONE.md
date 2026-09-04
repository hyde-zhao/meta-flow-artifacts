---
checkpoint_id: "CP6"
checkpoint_name: "编码完成检查"
type: "auto"
status: "PASS"
story_id: "CR037-S6"
story_slug: "ip-planning"
cr_id: "CR-037"
wave: 2
design_evidence_type: "technical-note"
executed_by: "meta-dev"
executed_at: "2026-08-05T00:00:00+00:00"
---

# CP6 编码完成检查 — CR037-S6 IP 规划桥接（exporter.py#IP 规划段）

## 1. 实现对象清单

| 文件 | 操作 | 说明 |
|------|:---:|------|
| `skills/topo-planning/src/exporter.py#IP 规划段` | 修改（S5 之上追加） | 新增 `compute_ip_plan(mapping_result, logical_topo, pool, vlan_assign=None, preset_subnets=None) -> dict` + `_supplement_direct_domains` + `_topology_has_sw`；复用 `ipam.IPAM` + `ipam.ip_plan_to_dict` + `topology_exporter._compute_broadcast_domains` |
| `skills/topo-planning/tests/test_exporter.py`（S6 段） | 新建 | compute_ip_plan 专项测试：自动 / preset / 容量不足 / SW vlan 缺失 / SW L2 域缺失 / 权威 schema |
| `skills/topo-planning/tests/fixtures/pool_sw.sample.yaml` | 新建 | 单 SW 双端口连接 TG/DUT 池（SW L2 域测试） |

不修改：`skills/topo-config/src/ipam.py`、`topology_exporter.py`、`vlan_isolator.py`（只读复用）。

## 2. 设计契约映射

| Story 技术说明契约 | 实现 |
|------|------|
| `compute_ip_plan(mapping_result, logical_topo, pool, vlan_assign=None, preset_subnets=None) -> dict`（TASK-S6-01） | ✅ |
| 复用 `topology_exporter._compute_broadcast_domains`（含 DUT-DUT 直连 L3 域，TASK-S6-02） | ✅ `_compute_broadcast_domains(mapping, topo, vlan_assign, pool)` |
| 复用 `ipam.IPAM`（interactive=False 自动 / preset 指定，容量校验，TASK-S6-03） | ✅ `IPAM().plan(domains, interactive=False, preset_subnets=preset)` |
| `ip_plan` 结构对齐 `ipam.ip_plan_to_dict` 权威 schema（endpoints 不含 gateway 设备，TASK-S6-04） | ✅ `ip_plan_to_dict(assignments)`；测试 `test_compute_ip_plan_authoritative_schema` |
| 接入 S5 `_resolve_ip_plan`（nodes.*.interfaces.*.ip/gw + dut.next_hop，TASK-S6-05） | ✅ S5 `_resolve_ip_plan` 消费 compute_ip_plan 输出；测试 `test_export_ip_plan_rendered` |
| SW（L2）拓扑 + `vlan_assign=None` → `IP_PLAN_SW_VLAN_MISSING`（S6-A 强制） | ✅ `_topology_has_sw` + 前置校验；测试 `test_compute_ip_plan_sw_vlan_missing` |
| 含 SW 但广播域无 switch_ports 非空 L2 域 → `IP_PLAN_SW_L2_DOMAIN_MISSING` | ✅ 广播域结果校验；测试 `test_compute_ip_plan_sw_l2_domain_missing` |
| 容量不足（端点+网关 > 可用 IP）→ 结构化错误 | ✅ `ValueError` 转 `ExporterError('IP_CAPACITY_EXCEEDED')`；测试 `test_compute_ip_plan_capacity_exceeded` |
| preset 子网非法/重复 → 报错 | ✅ IPAM.plan 对 preset 校验 ValueError → ExporterError |
| 自动模式 198.18.0.0/15 按序 /24 | ✅ IPAM 默认池；测试 `test_compute_ip_plan_auto` |
| 广播域为空 → 空 ip_plan；S5 缺 IP 默认 IP_PLAN_REQUIRED | ✅ compute_ip_plan 返回空 dict；S5 `ip_plan={}` 时 `_resolve_ip_plan` 返回，接口缺 IP → IP_PLAN_INCOMPLETE（或 ip_plan=None → IP_PLAN_REQUIRED） |

## 3. 测试结果

- 测试文件：`skills/topo-planning/tests/test_exporter.py`（S6 段 9 个专项 + 集成 27 个）
- 运行命令：`uv run --python 3.11 python -m pytest skills/topo-planning/tests/test_exporter.py`
- 结果：**36 passed**（0.54s），其中 S6 compute_ip_plan 专项：`test_compute_ip_plan_auto` / `_preset` / `_capacity_exceeded` / `_sw_vlan_missing` / `_sw_vlan_provided` / `_sw_l2_domain_missing` / `_authoritative_schema` + `test_export_ip_plan_rendered` / `test_export_ip_plan_authoritative_schema`

## 4. 既有测试回归情况

- 不修改 ipam.py / topology_exporter.py / vlan_isolator.py（只读复用）；topo-config 既有失败清单与基线逐条一致，无新增回归。
- topo-planning 全部 112 passed。

## 5. 偏离 LLD / Story 的设计点

| 偏离点 | 说明 |
|--------|------|
| `_supplement_direct_domains` 补充 TG/PC↔DUT 直连 p2p 域 | Story 技术说明「广播域计算复用 `_compute_broadcast_domains`」——但该函数仅覆盖 DUT-DUT 直连 L3 域（`_compute_l2_broadcast_domains` 的「仅处理 DUT↔DUT」分支），单 TG + 单 DUT 直连 MVP（node2_dut1_tg1_linkN）无任何广播域。为满足「S6 交付后由 S7/S8 传入 ip_plan」的完整 ip_plan 产出，在复用基础上补充 TG/PC↔DUT 直连域（DUT 为网关、TG/PC 为 endpoint）。该适配层对应 S6 技术说明 §风险「输入契约不完全匹配 → 需要适配层」；S5 `_resolve_ip_plan` 因此可正常渲染 MVP |
| 广播域 gateway=None 清洗为 {} | `_compute_l2_broadcast_domains` 对无 L3 网关的 SW L2 域置 gateway=None，`ipam.IPAM.plan` 内部 `gw.get('device')` 对 None 崩溃；compute_ip_plan 清洗（与 S6 适配层同源） |
| `compute_ip_plan` 允许 `mapping_result=None`（仅 SW vlan 缺失前置校验场景） | 仅用于前置校验（SW 拓扑 + vlan_assign=None 即报错，无需 mapping）；正常路径必须传 mapping_result |
