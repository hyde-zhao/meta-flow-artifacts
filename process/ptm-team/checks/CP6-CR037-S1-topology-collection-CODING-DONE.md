---
checkpoint_id: "CP6"
checkpoint_name: "编码完成检查"
type: "auto"
status: "PASS"
story_id: "CR037-S1"
story_slug: "topology-collection"
cr_id: "CR-037"
wave: 1
design_evidence_type: "full-lld"
executed_by: "meta-dev"
executed_at: "2026-08-05T00:00:00+00:00"
---

# CP6 编码完成检查 — CR037-S1 topology_collection.py

## 1. 实现对象清单

| 文件 | 操作 | 说明 |
|------|:---:|------|
| `skills/topo-planning/src/topology_collection.py` | 新建 | 解析 collection.md 提取 12 个拓扑 YAML，提供 `find_topology_file` / `list_topologies` / `derive_min_ports` 等 |
| `skills/topo-planning/src/__init__.py` | 新建 | src 包标记 |
| `skills/topo-planning/tests/test_topology_collection.py` | 新建 | 14 个单元测试 |
| `skills/topo-planning/tests/fixtures/fixture_collection_broken.md` | 新建 | YAML 语法错误 fixture |
| `skills/topo-planning/tests/fixtures/fixture_index_no_ptm_te.yaml` | 新建 | compatible_consumers 不含 ptm-te fixture |
| `skills/topo-planning/tests/fixtures/fixture_index_missing_collection.yaml` | 新建 | 缺 tgfw-topo-collection 条目 fixture |

## 2. 设计契约映射

| LLD / Story 契约 | 实现 |
|------|------|
| `find_topology_file(topo_id) → str`（返回 YAML 文件路径，P0-1/P1-1） | ✅ 写入标准缓存路径（`PTM_TOPOLOGY_CACHE_DIR` 可覆盖），12 个拓扑命中率 100% |
| `_parse_collection_md` 状态机（`## node*` → 首个 ```yaml 块，S1-B） | ✅ 正则白名单 `^##\s+(node\d+_.+)$`，仅 node2_dut1_tg1_link2 含 speed_class/media_type |
| `_load_index` 校验 tgfw-topo-collection + compatible_consumers 含 ptm-te（P0-2） | ✅ 不要求 node* 顶层条目 |
| `derive_min_ports`（interfaces→min_ports，显式声明优先） | ✅ 注入 topo_dict.nodes 不覆盖显式值 |
| `list_topologies`（候选集 = _parse_collection_md key 集合，S1-A） | ✅ 返回 12 个 node* topo_id，与 index.yaml 顶层无关 |
| 模糊搜索（编辑距离 ≤2，候选集来自 collection，R3） | ✅ `node2_dut1_tg1_link4` → {link3, link2, link5}，不含不存在的 node4_dut1_tg1_link4 |
| parser.py `from topology_collection import find_topology_file` 可 import | ✅ 测试验证模块可 import + 返回路径可被 `detect_topology_format` + `_parse_yaml_topology` 消费（P0-1） |
| 错误处理（5 种错误路径） | ✅ FileNotFoundError（collection 缺失 / topo_id 不存在）+ ValueError（index 缺条目 / 不含 ptm-te / YAML 解析失败带行号） |
| mtime 感知缓存 | ✅ 模块级 `_collection_cache`，二次调用不重读（mock 验证） |

## 3. 测试结果

- 测试文件：`skills/topo-planning/tests/test_topology_collection.py`
- 运行命令：`uv run --python 3.11 python -m pytest skills/topo-planning/tests/test_topology_collection.py`
- 结果：**14 passed**（0.55s，无警告）

覆盖：正常路径（find/derive/all_12/list/data_shape/cache_hit）、错误路径（nonexistent/missing_md/missing_entry/compat/yaml_error）、模糊搜索（suggestion_in_find）。

## 4. 偏离 LLD 的设计点

| 偏离点 | 说明 |
|--------|------|
| `_locate_collection_md` 显式路径严格使用 | 显式传入 `collection_path` 时不再回退默认查找路径，路径不存在直接 `FileNotFoundError`（保证显式参数精确语义；默认路径查找不受影响） |
| 路径查找增加开发仓库形态 | 除 LLD 规定的 `{SKILL_ROOT}/resource/...` 与 `~/.ptm-team/...` 外，增加"从模块文件向上回溯到仓库根 `resource/network-topology/`"，使开发仓库与安装形态均可用 |
| 缓存目录支持 `PTM_TOPOLOGY_CACHE_DIR` 环境变量 | LLD 固定 `~/.ptm-team/cache/topology/`，实现允许环境变量覆盖以便测试隔离，默认行为不变 |
| parser.py 未修改 | LLD §6.4 的 `sys.path` 过渡降级由 S8 安装器统一 PYTHONPATH 承担；S1 保证模块独立可用 + 返回路径契约正确，避免越权修改 topo-config（file_ownership 仅含 topology_collection.py） |

## 5. 验证入口

- meta-qa：`find_topology_file("node2_dut1_tg1_link3")` 返回路径 → `detect_topology_format` → `_parse_yaml_topology` 完整链路
- 对全部 12 个拓扑运行 dry-run（`python skills/topo-planning/src/topology_collection.py`）
- S10 将验证 `parser.parse_topology(topo_id=...)` 完整链路

## 6. 结论

**PASS** — CP6 编码完成检查通过。14 个单元测试全部通过，P0-1/P0-2/P1-1/S1-A/S1-B/R3 契约全部满足。

等待 host-orchestrator 推进 Story 状态并拉起 meta-qa。
