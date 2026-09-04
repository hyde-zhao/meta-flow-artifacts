---
doc_type: FEATURE-TASKS
id: cr038-loopback-envfile-tasks
feature_id: F-CR038-E3
cr_id: CR-038
version: "0.1"
status: draft
created_at: "2026-08-15"
owner: meta-se（story-planning）
---

# Feature 任务清单 — 环回 env-file（F-CR038-E3）

## STORY-038-06（exporter，full-lld）

| TASK-ID | 任务 | 输出 |
|---|---|---|
| TASK-E3-06-01 | `_validate_mvp_topology` 放宽为 1 TG + 1 DUT + N SW | exporter.py |
| TASK-E3-06-02 | links 去重/自环/闭环校验 | exporter.py |
| TASK-E3-06-03 | `_build_nodes` interface_kind 透传 | exporter.py |
| TASK-E3-06-04 | `_resolve_ip_plan` pppoe 跳过 IP 校验 | exporter.py |
| TASK-E3-06-05 | 单测：POS-006、BND-004、NEG-005 | tests/ |

## STORY-038-07（case_runner，technical-note）

| TASK-ID | 任务 | 输出 |
|---|---|---|
| TASK-E3-07-01 | load_env_file 容忍 swN 节点 | case_runner.py |
| TASK-E3-07-02 | 单测：POS-007、tg1/dut1 校验不破坏 | tests/ |

## STORY-038-13（多实例 TRex + interface_kind，full-lld）

| TASK-ID | 任务 | 输出 |
|---|---|---|
| TASK-E3-13-01 | physical_pool trex_instances 块 + from_pool_data | physical_pool.py |
| TASK-E3-13-02 | pool_merge _merge_tg 按 name 归并 | pool_merge.py |
| TASK-E3-13-03 | exporter _resolve_tg_instance + 单实例回退 | exporter.py |
| TASK-E3-13-04 | topology_model InterfaceInfo.interface_kind / NodeInfo.trex_instance | topology_model.py |
| TASK-E3-13-05 | parser 解析 interface_kind / trex_instance | parser.py |
| TASK-E3-13-06 | 单测：POS-010/011、BND-003 | tests/ |
