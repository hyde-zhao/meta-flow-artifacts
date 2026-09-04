---
doc_type: IMPLEMENTATION
story_id: STORY-038-06
story_slug: exporter-loopback-mvp
cr_id: CR-038
feature_id: F-CR038-E3
title: exporter 突破 MVP + SW nodes + 环回 links 校验 — 实现执行证据
design_evidence_type: full-lld
lld_ref: process/stories/STORY-038-06-exporter-loopback-mvp-LLD.md
source_hld: docs/design/HLD-CR-038.md §10.3
source_adr: docs/design/ARCHITECTURE-DECISION-CR-038.md §ADR-CR038-04 / §ADR-CR038-05
executed_by: meta-dev
executed_at: "2026-08-15"
---

# STORY-038-06 IMPLEMENTATION — exporter 突破 MVP + SW nodes + 环回 links 校验

## 实现前置检查

| 条目 | 状态 | 证据 |
|---|---|---|
| Story `status=dev-ready` 且 `dev_gate=cp5-approved` | PASS | `process/stories/STORY-038-06.md` frontmatter |
| full-lld 设计证据已确认（CP5 approved） | PASS | `process/checkpoints/CP5-CR038.md` + LLD `status=lld-ready-for-review` |
| 依赖门控（`depends_on: STORY-038-01/02` hard） | PASS | 上游 CP6 `STORY-038-01/02-CODING-DONE.md` 已产出，`DeviceMapping.role` / SW 归并可用 |
| 文件所有权无冲突（`exporter.py` shared，S13 merge_owner，本 Story 首次写入） | PASS | 未占用 S13 扩展点（`trex_instance`/`_resolve_tg_instance`/`trex_sync_port`/`trex_async_port`） |
| AI 可执行任务清单存在 | PASS | LLD §8 TASK-E3-06-01~05 |
| 不修改已 approve 对象 / `topology_model.py` / `parser.py` | PASS | 仅改 `exporter.py` + 新增 tests/fixtures |
| 安全约束（无凭据/真机下发/不穷举组合） | PASS | 沿用 `_CREDENTIAL_RE`；`_validate_mvp_topology` 仅按节点类型集合判定 |

## 实现对象清单

| 对象 | 文件 | 变更类型 | 职责 |
|---|---|---|---|
| `_validate_mvp_topology` | `skills/topo-planning/src/exporter.py` | 修改 | 放宽为 1 TG + 1 DUT + N SW，拒绝 Mock/PC/多 DUT/多 TG/UNKNOWN/MULTI |
| `_node_types` / `_sw_count` / `_locate_logical_tg_dut` / `_resolve_interface_kind` | 同上 | 新增 | 节点类型解析 / 环回判定 / TG-DUT 定位 / interface_kind duck-typing |
| `_validate_loopback_links` | 同上 | 新增 | 环回 links 去重/自环/同节点异口/连通分量/度数校验 |
| `_pair_loopback_arms` | 同上 | 新增 | TG 端口 → DUT 端口折叠臂配对（SW 中继透明） |
| `_build_port_mapping` | 同上 | 修改 | 单链路/环回双模式分派，返回 3 元组 |
| `_build_sw_nodes` | 同上 | 新增 | `swN` 节点段（host/brand/role/interfaces={}） |
| `_build_nodes` | 同上 | 修改 | tg1/dut1/swN + interface_kind 透传 |
| `_dut_iface_by_port` / `_dut_interface_obj` | 同上 | 新增 | 反查逻辑 DUT 接口（arm_map 优先 / port_by_link 兜底） |
| `_build_links` / `_build_loopback_links` | 同上 | 修改/新增 | 环回折叠臂 links 输出 |
| `_resolve_ip_plan` | 同上 | 修改 | pppoe DUT 接口跳过 IP 填充与完备性校验 |
| `test_exporter_loopback.py` | `skills/topo-planning/tests/` | 新增 | 13 个环回单测（T-06-01~T-06-13） |
| `fixtures/loopback_pool.yaml` | `skills/topo-planning/tests/fixtures/` | 新增 | 三角环测试池（tg-loop/dut-loop/sw-loop） |

## 设计契约映射

| Story 目标 | TASK-ID | 实现对象 | 测试 |
|---|---|---|---|
| R-F-001 `_validate_mvp_topology` 放宽（1TG+1DUT+N SW） | TASK-E3-06-01 | `_validate_mvp_topology` + `_node_types` | T-06-03~T-06-07 |
| R-F-003/004/005 links 去重/自环/闭环校验 | TASK-E3-06-02 | `_validate_loopback_links` + `_pair_loopback_arms` + `_build_port_mapping` + `_build_links` | T-06-01/T-06-08~T-06-11 |
| R-F-002 SW nodes 导出 + R-F-006 interface_kind 透传起点 | TASK-E3-06-03 | `_build_sw_nodes` + `_build_nodes` + `_resolve_interface_kind` + `_dut_iface_by_port`/`_dut_interface_obj` | T-06-01/T-06-02/T-06-12 |
| R-F-007 `_resolve_ip_plan` pppoe 跳过 | TASK-E3-06-04 | `_resolve_ip_plan` | T-06-13 |
| 单测 + fixture | TASK-E3-06-05 | `test_exporter_loopback.py` + `loopback_pool.yaml` | 全量 160 绿 |
| R-NF-001 回归无破坏 | 全部 | `_build_port_mapping` 单链路分支 `if sw_count == 0` 物理隔离 | 既有 `test_exporter.py` 全绿 |

- 上游接口契约：`DeviceMapping.role`（STORY-038-01）→ `swN.role`；SW 归并（STORY-038-02）→ `_is_node_type(dm,'SW')` 识别。
- 扩展点预留：`_resolve_interface_kind` 用 `getattr(iface, 'interface_kind', None) or 'physical'`，S13 加字段后零改动（ADR-CR038-05）。

## 单元测试与 Fixture 计划

- 测试文件：`skills/topo-planning/tests/test_exporter_loopback.py`（13 例）。
- 隔离策略（LLD §7.1）：手构 `ip_plan` + `setattr(node.interfaces[p], 'interface_kind', 'pppoe')` duck-typing 注入，绕开 `compute_ip_plan` 对 SW 拓扑的 `IP_PLAN_SW_VLAN_MISSING` 限制与 S13 parser/topology_model 未就绪。
- Fixture：`fixtures/loopback_pool.yaml`（三角环 `tg-loop`(IXIA-C, 701) ↔ `dut-loop`(HG3250, 702) ↔ `sw-loop`(ruijie-B, 703)，3 条直连电缆，每节点度 2）。
- 台账：测试内 `_fresh_ledger` 由 loopback_pool 节点动态生成（device_id → brand='test' + 全空闲 port_status），无需提交静态台账文件。
- 用例覆盖：POS-006（T-06-01/02）、BND-004（T-06-03~07）、NEG-005（T-06-08~11）、interface_kind（T-06-12）、pppoe 跳过 IP（T-06-13）、回归（T-06-14 由既有 test_exporter.py 隐式覆盖）。

## 最小实现切片

| 切片 | TASK-ID | 内容 | 完成准则 |
|---|---|---|---|
| S1 | TASK-E3-06-01 | `_validate_mvp_topology` 放宽 + `_node_types`/`_sw_count`/`_locate_logical_tg_dut` | T-06-03~07 |
| S2 | TASK-E3-06-02 | `_validate_loopback_links` + `_pair_loopback_arms` + `_build_port_mapping` 双模式 + `_build_loopback_links` | T-06-01/08~11 |
| S3 | TASK-E3-06-03 | `_build_sw_nodes` + `_build_nodes` + `_resolve_interface_kind` + `_dut_iface_by_port`/`_dut_interface_obj` | T-06-01/02/12 |
| S4 | TASK-E3-06-04 | `_resolve_ip_plan` pppoe 跳过 | T-06-13 |
| S5 | TASK-E3-06-05 | `test_exporter_loopback.py` + `loopback_pool.yaml` | 全量 160 绿 |

## 平台差异处理

N/A —— 纯 Python 导出器改动，无平台路径、安装结构、CLI 或真机协议差异。不涉及 `delivery/doc/PLATFORM-CONTRACTS.yaml` / `PLATFORM-INSTALL-SPEC.md`。

## 验证结果

- 运行命令：`uv run --python 3.11 pytest skills/topo-planning/tests/`
- 结果：**160 passed, 0 failed**（基线 147 + 新增环回 13，无回归）。
- 新增单测：`test_exporter_loopback.py` **13 passed**。
- 既有回归：`test_exporter.py` 全绿（单链路模式 `N SW == 0` 行为不变，R-NF-001）。
- dry-run 结构验证：`test_loopback_export_structure` 断言 env-file `nodes={tg1,dut1,sw1}`、`port_mapping={port1,port2}`（无 sw 键）、`sw1.interfaces=={}`、`links` 为 2 条折叠臂。

## 未覆盖项

- 环回完整 ip_plan 端到端生成（`compute_ip_plan` 对含 SW 拓扑抛 `IP_PLAN_SW_VLAN_MISSING`）——单测用手构 ip_plan 隔离（RA-06-01，OPEN-06-02，归属后续 Story/CR 供 vlan_assign）。
- pppoe 标记粒度语义（DUT pppoe-facing vs SW pppoe-server）最终由 S13 parser 注入——S06 只做通用跳过机制（OPEN-06-01，S13 闭环）。
- 真机 env-file 执行 / case-runner 端到端消费——不属本 Story（无凭据/真机下发约束）。
- 多环/分支拓扑（非单一简单环）——被 `LOOPBACK_CLOSED_LOOP` 显式拦截，不穷举（RA-038-004 首版）。

## 设计缺口反馈

1. **`_resolve_ip_plan` gateway 分支 pppoe 跳过需避免误跳端点循环**：LLD §3.7 只要求「pppoe DUT 接口反查命中也不写 ip」，但实现中 gateway 命中 pppoe 时若用 `continue` 会跳过整个广播域的 endpoints 循环，导致该域 TG 侧 ip 未填充、触发误报 `IP_PLAN_INCOMPLETE`。实现改为 `if not (node_key=='dut1' and pppoe): 写 ip`，使 pppoe 跳过只作用于 gateway 自身的 ip 写入、不影响 endpoints。该点已由 T-06-12 全链路覆盖。
2. **`_validate_loopback_links` 重复无向边采用静默去重而非报错**：R-F-003「只输出一次」+ LLD §5.3 步骤 1「同一无向对保留首次」明确为去重语义（非错误），与早期草案「重复边报错」不一致处已按 LLD 收敛为 `continue`（T-06-11 断言返回 3 条边、无异常）。

## 后续交接

- 验证入口：`uv run --python 3.11 pytest skills/topo-planning/tests/`。
- 关键场景：SCN-038-POS-006（环回三层导出）、SCN-038-BND-004（Mock/PC/多 DUT 拒绝）、SCN-038-NEG-005（自环/闭环结构化错误）。
- 风险提示：
  - `compute_ip_plan` 对 SW 拓扑的 `IP_PLAN_SW_VLAN_MISSING` 限制（单测手构 ip_plan 隔离，端到端待后续 Story）。
  - `interface_kind` 语义边界待 S13 闭环（OPEN-06-01）；`trex_instances`/`_resolve_tg_instance` 等 S13 扩展点本 Story 未占用。
- 设计 delta：本 Story 无长期 Feature DESIGN / ADR / HLD 回写（`design_delta_required=false`）。
- 文件所有权：`exporter.py` 为 shared，S13 为 merge_owner；本 Story 为首次写入方，后续 S13 合入需 merge_owner 协调。
