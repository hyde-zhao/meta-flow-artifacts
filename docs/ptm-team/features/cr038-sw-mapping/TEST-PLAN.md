---
doc_type: FEATURE-TEST-PLAN
id: cr038-sw-mapping-testplan
feature_id: F-CR038-E1
cr_id: CR-038
version: "0.1"
status: draft
created_at: "2026-08-15"
owner: meta-se（story-planning）
---

# Feature 测试计划 — SW 映射与台账（F-CR038-E1）

## 1. 测试对象

- `skills/topo-config/src/topo_mapper.py`（显式 SW 匹配 + reserved 互斥 + role）
- `skills/topo-planning/src/pool_merge.py`（SW 归并 + 台账映射）

## 2. 测试场景（来源 SCENARIOS-CR-038.yaml）

| 场景 ID | 类别 | 覆盖点 |
|---|---|---|
| SCN-038-POS-001 | 正向 | SW 显式映射 status=matched，link_mapping 端点含 SW |
| SCN-038-POS-002 | 正向 | role=pppoe-server 保留 |
| SCN-038-NEG-001 | 负向 | SW 无候选 → 结构化 reason |
| SCN-038-NEG-002 | 负向 | SW 接口不匹配 → 结构化 reason |
| SCN-038-BND-002 | 边界 | Mock 仍 virtual，含 Mock 端点 link 标 unmapped |

## 3. 专项单测（R-F-018 地基）

| 用例 | 断言 |
|---|---|
| 显式 SW + 自动透传并存 | `_try_l2_pass_through` 永不占用 explicit_sw_reserved 中 SW |
| 回溯顺序无关性 | 交换节点处理顺序，reserved 集互斥不变量不变 |
| 台账互斥 | 显式 SW 映射成功后 allocated_to 写回；自动透传 SW 端口同样占用 |
| role 传递 | DeviceMapping.role == 逻辑 node.role（pppoe-server） |

## 4. 通过标准

- 全部正向/负向/边界场景 PASS。
- R-F-018 互斥单测通过（顺序无关）。
- CR-037 node2 场景回归无破坏（R-NF-004）。
