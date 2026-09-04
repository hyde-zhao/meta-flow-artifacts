---
story_id: STORY-038-06
story_slug: exporter-loopback-mvp
cr_id: CR-038
checkpoint: CP7
result: PASS_WITH_RISK
validation_mode: mixed
created_at: "2026-08-15"
author: meta-qa
design_evidence_type: full-lld
---

# CP7-STORY-038-06-exporter-loopback-mvp-VERIFICATION-DONE

## 结论

| 项目 | 内容 |
|---|---|
| 阶段决策 | PASS_WITH_RISK |
| validation_mode | mixed（pytest + dry-run 等价验证） |
| 路由 | none（verified；OPEN-06-02 归属后续 Story/CR） |

## 验证对象清单

| 文件 | 操作 | 验证结果 |
|---|---|---|
| `skills/topo-planning/src/exporter.py` | 修改（MVP 放宽 + `_validate_loopback_links`/`_pair_loopback_arms`/`_build_sw_nodes`/`_resolve_interface_kind`/`_dut_iface_by_port`/`_dut_interface_obj`/`_build_loopback_links`） | PASS |
| `skills/topo-planning/tests/test_exporter_loopback.py` | 新增（13 用例） | PASS |
| `skills/topo-planning/tests/fixtures/loopback_pool.yaml` | 新增 | PASS |

## 验证追踪矩阵

| 场景/契约 | 测试 | 结果 |
|---|---|---|
| SCN-038-POS-006（环回三层导出） | `test_loopback_export_structure` / `test_loopback_sw_node_fields` | PASS |
| SCN-038-BND-004（Mock/PC/多 DUT 拒绝 + SW 放行） | `test_mvp_{mock,pc,multi_dut}_rejected` / `test_mvp_{sw,multi_sw}_allowed` | PASS |
| SCN-038-NEG-005（自环/同节点/断链/去重） | `test_loopback_{self_loop,same_node_diff_port,disconnected,dedup}` | PASS |
| R-F-006 interface_kind 透传 | `test_interface_kind_passthrough` | PASS |
| R-F-007 pppoe 跳过 IP | `test_resolve_ip_plan_pppoe_skip` | PASS |

## 设计契约验证

- R-F-001：`_validate_mvp_topology` 放宽为 1 TG + 1 DUT + N SW，拒绝 Mock/PC/多 DUT/多 TG/UNKNOWN/MULTI
- R-F-003/004/005：`_validate_loopback_links` 去重/自环/闭环结构化错误（`LOOPBACK_SELF_LOOP`/`LOOPBACK_CLOSED_LOOP`）
- R-F-002 SW nodes 导出（进 nodes 不进 port_mapping）；R-F-006 interface_kind duck-typing 透传起点
- R-F-007 `_resolve_ip_plan` pppoe DUT 接口跳过 IP
- 设计缺口：`_resolve_ip_plan` pppoe 跳过从 `continue` 改为条件写 ip（避免误跳 endpoints 循环），T-06-12 覆盖，已留痕

## 分层验证计划

- 层A：`test_exporter_loopback.py` 独立复跑 13 passed；全量 340 passed / 0 failed
- 层B：N/A（无真机 env-file 执行 / case-runner 消费）

## 8 维度验收

| # | 维度 | 结果 |
|---|---|---|
| 1 | 完整性（BLOCKING） | PASS（3 产物 ≥ 验收标准） |
| 2 | 平台适配（BLOCKING） | PASS（N/A：纯 Python 导出） |
| 3 | 验收标准覆盖（BLOCKING） | PASS（POS-006/007、NEG-005、BND-004 全覆盖） |
| 4 | 安全合规（BLOCKING） | PASS（沿用 `_CREDENTIAL_RE`，无真机下发） |
| 5 | 命名规范（REQUIRED） | PASS |
| 6 | Frontmatter（REQUIRED） | PASS |
| 7 | 可安装性（REQUIRED） | PASS（N/A：纯代码） |
| 8 | 文档覆盖（OPTIONAL） | 文档阶段检查 |

## 测试证据

- `uv run --python 3.11 pytest skills/topo-planning/tests/test_exporter_loopback.py -q` → 13 passed
- 全量 → 340 passed, 0 failed

## 问题与缺陷

| # | 严重度 | 问题 | 状态 |
|---|---|---|---|
| Q1 | Info（范围） | OPEN-06-02：环回完整 ip_plan 端到端受 `IP_PLAN_SW_VLAN_MISSING` 限制 | 归属后续 Story/CR |

## 剩余风险

- OPEN-06-02（环回完整 ip_plan 端到端未闭环）
- 臂配对仅适用单一简单环，多环/分支拓扑被 `LOOPBACK_CLOSED_LOOP` 拦截（RA-038-004 首版）
- 环回折叠臂 links name 语义与单链路不同（RA-06-04）

## 阶段决策

PASS_WITH_RISK → verified；风险（OPEN-06-02 + 臂配对边界）汇入 CP8。
