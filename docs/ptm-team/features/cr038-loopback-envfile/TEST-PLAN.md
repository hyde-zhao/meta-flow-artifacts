---
doc_type: FEATURE-TEST-PLAN
id: cr038-loopback-envfile-testplan
feature_id: F-CR038-E3
cr_id: CR-038
version: "0.1"
status: draft
created_at: "2026-08-15"
owner: meta-se（story-planning）
---

# Feature 测试计划 — 环回 env-file（F-CR038-E3）

## 1. 测试对象

- `skills/topo-planning/src/exporter.py`（MVP 放宽 + links 校验 + interface_kind）
- `skills/topo-planning/src/pool_merge.py`（多实例归并）
- `skills/topo-config/src/physical_pool.py`（trex_instances）
- `skills/topo-collection/src/topology_model.py` / `parser.py`（数据模型）
- `skills/topo-collection/src/case_runner.py`（env-file 校验）

## 2. 测试场景（来源 SCENARIOS-CR-038.yaml）

| 场景 ID | 类别 | 覆盖点 |
|---|---|---|
| SCN-038-POS-006 | 正向 | 环回 env-file 生成（tg/dut/sw 三层） |
| SCN-038-POS-007 | 正向 | case_runner 容忍 swN |
| SCN-038-POS-010 | 正向 | 多实例 TRex env-file |
| SCN-038-POS-011 | 正向 | interface_kind=pppoe 无 ip、next_hop=null |
| SCN-038-BND-003 | 边界 | 单实例回退（tg1/dut1） |
| SCN-038-BND-004 | 边界 | Mock/PC/多 DUT 拒绝 |
| SCN-038-NEG-005 | 负向 | 自环/闭环 links → 结构化错误 |

## 3. 通过标准

- 全部正向/边界/负向场景 PASS。
- IP 计划完整性校验对 pppoe 接口跳过、physical 接口仍校验。
- env-file 键符合 ADR-09（9 占位不变，多实例扩展）。
