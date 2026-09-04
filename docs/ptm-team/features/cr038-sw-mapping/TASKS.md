---
doc_type: FEATURE-TASKS
id: cr038-sw-mapping-tasks
feature_id: F-CR038-E1
cr_id: CR-038
version: "0.1"
status: draft
created_at: "2026-08-15"
owner: meta-se（story-planning）
---

# Feature 任务清单 — SW 映射与台账（F-CR038-E1）

## STORY-038-01（topo_mapper，full-lld）

| TASK-ID | 任务 | 输出 |
|---|---|---|
| TASK-E1-01-01 | 预计算 explicit_sw_reserved（回溯前） | topo_mapper.py |
| TASK-E1-01-02 | `_try_l2_pass_through` 增加 reserved 集跳过条件 | topo_mapper.py |
| TASK-E1-01-03 | `_build_result` 确保 SW role 传递 | topo_mapper.py |
| TASK-E1-01-04 | 结构化失败 reason（SW 无候选/接口不匹配） | topo_mapper.py |
| TASK-E1-01-05 | 单测：POS-001/002、NEG-001/002、BND-002、R-F-018 互斥 | tests/ |

## STORY-038-02（pool_merge，technical-note）

| TASK-ID | 任务 | 输出 |
|---|---|---|
| TASK-E1-02-01 | 确认 SW 节点归并 + switch 块补充 device_type/hardware_platform | pool_merge.py |
| TASK-E1-02-02 | SW 台账映射（_resolve_ledger 覆盖 SW） | pool_merge.py |
| TASK-E1-02-03 | 单测：SW 归并台账映射无冲突（R-F-011） | tests/ |
