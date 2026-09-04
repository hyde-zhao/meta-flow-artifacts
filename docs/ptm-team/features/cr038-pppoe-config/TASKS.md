---
doc_type: FEATURE-TASKS
id: cr038-pppoe-config-tasks
feature_id: F-CR038-E2
cr_id: CR-038
version: "0.1"
status: draft
created_at: "2026-08-15"
owner: meta-se（story-planning）
---

# Feature 任务清单 — PPPoE 配置（F-CR038-E2）

## STORY-038-03（commands + schema，full-lld）

| TASK-ID | 任务 | 输出 |
|---|---|---|
| TASK-E2-03-01 | physical_pool SW 节点新增 pppoe_server 块 schema | physical_pool.py + config/physical_pool.yaml |
| TASK-E2-03-02 | commands.py 三厂商 PPPoE 命令键 | commands.py |
| TASK-E2-03-03 | password_cipher 占位/密文校验 | physical_pool.py |
| TASK-E2-03-04 | 单测：POS-004、R-F-019 同源一致 | tests/ |

## STORY-038-04（switch_configurator，technical-note）

| TASK-ID | 任务 | 输出 |
|---|---|---|
| TASK-E2-04-01 | configure_pppoe_server（from_pool + 模板渲染 + dry-run） | switch_configurator.py |
| TASK-E2-04-02 | 幂等（先查再建） | switch_configurator.py |
| TASK-E2-04-03 | 单测：POS-003、REC-001 | tests/ |

## STORY-038-05（op_mapper，full-lld）

| TASK-ID | 任务 | 输出 |
|---|---|---|
| TASK-E2-05-01 | fw_config_pppoe_client 意图映射 | op_mapper.py |
| TASK-E2-05-02 | op 缺失三选一 fallback（默认 atomic-skip） | op_mapper.py |
| TASK-E2-05-03 | 单测：POS-005、PRE-001、RA-038-001 闭环 | tests/ |
