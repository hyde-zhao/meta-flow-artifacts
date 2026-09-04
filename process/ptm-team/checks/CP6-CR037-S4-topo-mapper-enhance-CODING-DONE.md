---
checkpoint_id: "CP6"
checkpoint_name: "编码完成检查"
type: "auto"
status: "PASS"
story_id: "CR037-S4"
story_slug: "topo-mapper-enhance"
cr_id: "CR-037"
wave: 2
design_evidence_type: "full-lld"
executed_by: "meta-dev"
executed_at: "2026-08-05T00:00:00+00:00"
---

# CP6 编码完成检查 — CR037-S4 topo_mapper.py 增强

## 1. 实现对象清单

| 文件 | 操作 | 说明 |
|------|:---:|------|
| `skills/topo-config/src/topo_mapper.py` | 修改 | per-link port_requirement 三态（None/全局/per-link）+ 回溯剪枝按 link_id + hardware_platform 等价类 / device_types 候选过滤 + 端口兼容矩阵（PORT_COMPAT_MATRIX）+ H4 约束叠加（S4-A：TE 系强制 fiber）+ include-at-least-one 软约束（S4-B）+ H1 ledger 占用闭环（_commit_allocation / release_env）+ MappingResult.per_link_requirements / allocated_env / include_at_least_one 字段 |
| `skills/topo-config/src/physical_pool.py` | 修改（仅新增方法） | `PhysicalPool.release(env_name)`（P1-7）+ `PhysicalPool.from_pool_data(pool_data, ledger_path)`（S3-B）；台账 schema 不变（P0-3/S4-C） |
| `skills/topo-config/src/mapping_validator.py` | 修改 | R2 per-link 聚合（P1-6/S4-D）：validate 增加 topo_limit 参数；按 PortMapping.via_link 逐端口校验，无约束端口标注 unconstrained 不静默跳过；include-at-least-one 按 ≥1 聚合判定 |
| `skills/topo-config/src/topology_model.py` | 检查（无需改动） | MappingResult 定义于 topo_mapper.py；LLD §4 output_files 中 topology_model.py 的字段变更实际落在 topo_mapper.py 的 MappingResult（见 §4 偏离说明） |
| `skills/topo-config/tests/test_topo_mapper_s4.py` | 新建 | 26 个 pytest 单元测试 |
| `skills/topo-config/tests/fixtures/pool_te.sample.yaml` | 新建 | TE 可用池（tg1 含 TE + dut1 含 TE + dut2 无 TE + 直连 cables） |
| `skills/topo-config/tests/fixtures/ledger_te.sample.yaml` | 新建 | pool_te 对应台账（P0-3 list + port_status） |
| `skills/topo-config/tests/fixtures/ledger_allocated.sample.yaml` | 新建 | 预置 allocated_to=case-000 台账（H1 跨用例占用测试） |
| `skills/topo-config/tests/fixtures/pool_no_te.sample.yaml` | 新建 | 无 TE 端口池（失败路径测试） |

## 2. 设计契约映射

| LLD / Story 契约 | 实现 |
|------|------|
| `match(logical_topo, topo_limit=None, port_requirement=None, env_name=None)`（FR1） | ✅ 三态归一化：`_normalize_port_requirement`（None→{}；全局含 speed_class/media_type→应用全部 link；per-link→原样） |
| 回溯剪枝按 link_id（FR2/FR6，AG-2 推荐 A） | ✅ `_check_connectivity` 对每条 link_id 调 `_resolve_per_link_requirement(lid)`，`_port_meets_requirement(dev, port, req, strict)` |
| hardware_platform 等价类 + device_types 候选过滤（FR3，P1-3） | ✅ `_build_candidates` 调 `_device_constraint_for` + `_device_meets_constraint`（device_types 命中或 equiv_class 命中 hardware_platform；约束非空未命中→False） |
| 端口兼容矩阵（FR4）：GE=copper\|fiber，TE/XTE/QTE/TTE=fiber-only，FE=copper\|fiber | ✅ `PORT_COMPAT_MATRIX` 常量 + `_port_meets_requirement` 矩阵判定（与 S2 limit_parser 同源语义） |
| H1 ledger 占用闭环（FR5）：_is_port_busy 消费 allocated_to + 映射成功写回 + release | ✅ `_is_port_busy`（既有，消费 allocated_to）+ `_commit_allocation`（allocate + save_ledger）+ `PhysicalPool.release(env_name)`（仅内存台账）+ `TopoMapper.release_env`（薄封装） |
| H4 约束叠加（FR7，S4-A）：用户 > 逻辑 topo > 默认；TE 系强制 fiber、丢弃逻辑 copper | ✅ `_resolve_per_link_requirement`：用户字段优先 + 逻辑下限补齐；`speed_class in TE_SPEED_CLASSES → media_type='fiber'` |
| include-at-least-one 软约束（FR8，S4-B）：not strict 分支位于硬剪枝 return False 之前 + post-match ≥1 | ✅ `_user_link_requirement` 对 include-at-least-one 返回 strict=False；`_port_meets_requirement` 中 `if not strict: return True` 位于硬剪枝前；`_build_result` 中 `_count_satisfied_links` post-match 计数，0 条→failed + 结构化明细 |
| MappingResult 新增 per_link_requirements / allocated_env（FR10） | ✅ `MappingResult.per_link_requirements: Dict[str, dict]` + `allocated_env` + `include_at_least_one` |
| mapping_validator R2 per-link 聚合不静默跳过（FR9，P1-6/S4-D） | ✅ `MappingValidator.validate(..., topo_limit=None)`；R2 优先读 `MappingResult.per_link_requirements`，无约束端口计入 unconstrained 标注 |
| S3-B：TopoMapper 输入 raw PhysicalPool（from_pool_data 构造） | ✅ `PhysicalPool.from_pool_data(unified.to_pool_data(), unified.ledger_path)`；测试 `test_from_pool_data_construction` |
| 空 limit 向后兼容（FR11/NFR1）：`match(topo)` 行为等价 | ✅ `port_requirement=None → port_requirements={}`，`_port_meets_requirement` 对空 req 返回 True |
| 台账 schema 不变（NFR2，P0-3/S4-C） | ✅ `release`/`from_pool_data`/`_commit_allocation` 均基于 list + port_status schema，未新增字段 |

## 3. 测试结果

- 测试文件：`skills/topo-config/tests/test_topo_mapper_s4.py`
- 运行命令：`uv run --python 3.11 python -m pytest skills/topo-config/tests/test_topo_mapper_s4.py`
- 结果：**26 passed**（0.46s）

覆盖：per-link TE 约束匹配/无 TE 失败、全局约束向后兼容、hardware_platform 等价类（命中/无候选）、device_types 列表、端口兼容矩阵常量/GE-fiber/TE-fiber-only、**H4 TE-copper unsatisfiable（S4-A）**、H4 逻辑下限、**include-at-least-one 软约束（S4-B）**/0 条失败/soft-not-hardcut、from_pool_data、ledger 写回/cross-use busy/release/persist、release_env、空 limit 向后兼容、per_link_requirements 回填、validator R2 per-link/unconstrained/include-at-least-one。

## 4. 既有测试回归情况

- 基线（改动前）：`skills/topo-config/tests/` = 28 failed / 28 passed（失败均为 config/physical_pool.yaml 模板占位符导致的既有失败）。
- 改动后：**28 failed / 54 passed**——失败清单与基线**逐条一致**（`diff` 验证无新增失败），新增 26 个 S4 测试全通过。
- 无回归。失败项与本次改动无关（模板占位符 `${...}` 无法被 PhysicalPool.get_devices_by_type 匹配）。

## 5. 偏离 LLD 的设计点

| 偏离点 | 说明 |
|--------|------|
| `MappingResult` 字段落在 `topo_mapper.py` 而非 `topology_model.py` | LLD §4 output_files 将 `topology_model.py` 列为修改对象，但 MappingResult 数据类实际定义于 `topo_mapper.py`（既有实现）；`topology_model.py` 只含 Topology/NodeInfo 等逻辑层模型。字段按实际定义位置添加，文件所有权在 CP5 声明覆盖 `topo_mapper.py`；`topology_model.py` 经检查无需改动 |
| `MappingResult` 额外新增 `include_at_least_one: List[dict]` 字段 | LLD §5.3 仅定义 per_link_requirements/allocated_env；为支撑 S4-B post-match 结构化明细（缺 TE 端口类），追加 include_at_least_one 记录约束满足数，供 S5 build_failure_report 消费 |
| `_check_connectivity` 直连 cable 检查改为 `busy = used_ports | ports_needed` | 修复同设备对多 link 复用同一 cable 的既有缺陷（原实现只查 used_ports）；保证每条逻辑 link 占用独立端口，per-link 约束语义正确 |
| `_build_result` 直连 link 选 cable 重放 per-link 约束 | 原实现在结果构建阶段不校验端口类型（回溯校验会被结果构建绕过）；S4 增加 `_port_meets_requirement(phys_a/b, port, req, strict)` 重放，保证 link_mappings 反映约束 |
| `_resolve_per_link_requirement` 增加 port_requirement 融合 | LLD §8.3 `_user_link_requirement` 只读 topo_limit.link_constraints；实现优先读 `self.port_requirements`（三态归一化结果），再回退 topo_limit，保证 port_requirement 参数直接生效 |
| `from_pool_data` 台账缺失降级空台账 | S3 merge 可产出 ledger_path=None；实现用空台账（list 空）构造，与 pool_merge.load_ledger 降级语义一致 |
