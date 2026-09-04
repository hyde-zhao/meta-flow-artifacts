---
checkpoint_id: "CP6"
checkpoint_name: "编码完成检查"
type: "auto"
status: "PASS"
story_id: "CR037-S3"
story_slug: "pool-merge"
cr_id: "CR-037"
wave: 1
design_evidence_type: "full-lld"
executed_by: "meta-dev"
executed_at: "2026-08-05T00:00:00+00:00"
---

# CP6 编码完成检查 — CR037-S3 pool_merge.py

## 1. 实现对象清单

| 文件 | 操作 | 说明 |
|------|:---:|------|
| `skills/topo-planning/src/pool_merge.py` | 新建 | physical_pool.yaml × devices.yaml 归并为 UnifiedPool：DUT host 关联 + TG api_server 关联 + 台账三角映射 |
| `skills/topo-planning/tests/test_pool_merge.py` | 新建 | 21 个单元测试 |
| `skills/topo-planning/tests/fixtures/pool_merge/pool_sample.yaml` | 新建 | 7 节点简化物理池（FW1/FW2/SW1/SW2/PC1/PC2/TG1） |
| `skills/topo-planning/tests/fixtures/pool_merge/devices_sample.yaml` | 新建 | device_groups 为 list 的设备清单（S3-A） |
| `skills/topo-planning/tests/fixtures/pool_merge/ledger_sample.yaml` | 新建 | list + port_status 台账（P0-3） |
| `skills/topo-planning/tests/fixtures/pool_merge/pool_conflict.yaml` | 新建 | 同 host 不同 device_type 冲突池 |
| `skills/topo-planning/tests/fixtures/pool_merge/pool_full.yaml` | 新建 | 池全量承载降级数据 |

## 2. 设计契约映射

| LLD / Story 契约 | 实现 |
|------|------|
| `merge_pool(pool_path, devices_path, ledger_path=None) → UnifiedPool` | ✅ 最终池数据（S3-B） |
| DUT 节点：pool.management.host ↔ devices.device_groups[].firewall.host | ✅ `_find_device_group_by_host` |
| device_groups 为 list，遍历用 list 迭代（S3-A） | ✅ `for group in devices_data.device_groups`，无 `.items()`/`.values()` |
| TG 节点：api_server 关联（归一化后匹配，P1-4；api_server 主键 + host 兜底，S3-C） | ✅ `_find_device_group_by_api_server`：**两遍完整循环**（先 api_server 主键，再 host 兜底），决策表 Minor-1 |
| 关联失败不阻断，标记 unmatched_nodes[] | ✅ DUT/TG/SW/PC 关联失败均记 unmatched |
| 台账三角映射：ledger.model → pool.device_type → device-reference 平台（P0-3/P2-5/S3-D） | ✅ `_resolve_ledger` + `_resolve_hardware_platform`（conflicts 型号不决策，返回 None + warning，S3-D） |
| 型号冲突：同 host 且 pool.device_type ≠ devices → warning + devices 为准 | ✅ `_merge_dut` conflict_resolution 默认 devices_first |
| 池全量承载降级：仅校验 devices 存在性（备选 B） | ✅ `_check_devices_existence` + conflict_resolution='pool_first' |
| TG 节点归并：从 devices 补充 sub_type + hardware_platform | ✅ `_merge_tg` |
| `UnifiedPool.to_pool_data()` 导出 physical_pool.yaml schema（S3-B） | ✅ `to_pool_data()` + `UnifiedNode.to_pool_node()`；经 `PhysicalPool.load` 构造验证通过 |

## 3. 测试结果

- 测试文件：`skills/topo-planning/tests/test_pool_merge.py`
- 运行命令：`uv run --python 3.11 python -m pytest skills/topo-planning/tests/test_pool_merge.py`
- 结果：**21 passed**（0.15s）

覆盖：DUT host 关联、TG api_server 关联（归一化）、TG host 兜底、TG 无 key unmatched、TG host 兜底决策表（Minor-1 首组不误命中）、device_groups list 遍历、SW 未注册 unmatched、型号冲突（devices_first / pool_first）、全节点归并、台账三角映射、端口占用回填、台账缺失、池全量承载降级、缺文件错误、links 保留、to_pool_data roundtrip、多 TG、空池、未知 node_type。

## 4. 偏离 LLD 的设计点

| 偏离点 | 说明 |
|--------|------|
| `UnifiedPool.links` 保留 dict 形态 | LLD §5.1 注释写 `List[Dict]`，但 physical_pool.yaml 实际 schema 为 `links: {link1: {...}}`（dict）；实现保留 dict 以匹配 to_pool_data 输出的 physical_pool.yaml schema，测试 `test_unified_pool_links_preserved` 验证与原 pool links 一致 |
| `merge_pool` 增加 `conflict_resolution` / `platform_map_path` 可选参数 | 对齐 LLD §13 灰区3（devices_first / pool_first）；platform_map_path 供测试隔离（NFR3 独立测试） |
| `_merge_sw_pc` 尝试按 host 查找 switch/client device_group 补充信息 | LLD §7 SW/PC 分支的补充实现；未找到记 unmatched（非阻断） |
| 台账加载支持 `device_id` 字符串化索引 | 台账 device_id 可能是 int/str，`ledger_by_id` 用 `str(device_id)` 归一化，pool 侧查表同样 `str()` 转换 |

## 5. 验证入口

- meta-qa：`merge_pool` → `UnifiedPool.to_pool_data()` → `PhysicalPool.load()`（S3-B）完整链路
- TG 决策表边界：api_server 主键优先于首组 host 兜底（test_tg_host_fallback_decision_table）
- S10 将验证 `merge_pool → UnifiedPool → TopoMapper.match()` 全链路

## 6. 结论

**PASS** — CP6 编码完成检查通过。21 个单元测试全部通过，S3-A/B/C/D + P1-4/P1-5 + Minor-1 全部契约满足。

等待 host-orchestrator 推进 Story 状态并拉起 meta-qa。
