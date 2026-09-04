---
checkpoint_id: "CP6"
checkpoint_name: "编码完成检查"
type: "auto"
status: "PASS"
story_id: "CR037-S2"
story_slug: "limit-parser"
cr_id: "CR-037"
wave: 1
design_evidence_type: "full-lld"
executed_by: "meta-dev"
executed_at: "2026-08-05T00:00:00+00:00"
---

# CP6 编码完成检查 — CR037-S2 limit_parser.py

## 1. 实现对象清单

| 文件 | 操作 | 说明 |
|------|:---:|------|
| `skills/topo-planning/src/limit_parser.py` | 新建 | topo-limit 结构化解析：平台别名展开 + speed_class 归一 + TG 分支 + 端口兼容矩阵 + R4 冲突消歧 |
| `skills/topo-planning/config/platform_alias.yaml` | 新建 | 平台别名映射数据（13 硬件系列 + nxp1043-4g/8g 拆分 + tg_platforms 6 组合 + speed_class_aliases + conflicts） |
| `skills/topo-planning/docs/limit-syntax.md` | 新建 | 语法规范文档（≥5 示例 + ≥7 反例 + 别名表 + TG 分支 + 消歧说明 + nxp1043 拆分说明，G5 交付物） |
| `skills/topo-planning/tests/test_limit_parser.py` | 新建 | 41 个单元测试 |
| `skills/topo-planning/tests/fixtures/platform_alias_test.yaml` | 新建 | 精简映射表（测试自包含） |
| `skills/topo-planning/tests/fixtures/devices_pool_test.yaml` | 新建 | 消歧测试设备池（HG3250 + C3758 设备） |
| `skills/topo-planning/tests/fixtures/devices_pool_ambiguous.yaml` | 新建 | 多平台并存设备池（AMBIGUOUS_MODEL 测试） |

## 2. 设计契约映射

| LLD / Story 契约 | 实现 |
|------|------|
| `parse_limit(limit_input, platform_map_path=None, pool_devices=None) → TopoLimit` | ✅ dict/JSON str/None 统一处理 |
| 平台别名展开：海光3250 → [A1300-HU, A1500-HU, A1580-HU, A1600-HU] | ✅ 主映射 + fixture 均通过 |
| speed_class 别名归一（10GE/万兆→TE、QTE/40GE→QTE、TTE/25GE→TTE，S2-A） | ✅ 归一后做兼容矩阵校验 |
| TG 平台约束分支（S2-B：sub_type/hardware_platform，不套 DUT 型号表） | ✅ `_expand_tg_platform` 6 组合校验 |
| 链路约束字段名统一为 link（S2-C） | ✅ specific 模式用 `link`，缺 link 报 PARAM_INVALID |
| device_constraints.node_ref 空 → all 语义 | ✅ scope="all" |
| link_constraints 3 模式（include-at-least-one/all/specific） | ✅ mode 校验 |
| 端口兼容矩阵（GE=copper|fiber，TE/XTE/QTE/TTE=fiber-only） | ✅ `_validate_port_compat` |
| 空 limit（None/''/{}）→ 空约束 | ✅ is_empty=True |
| limit-syntax.md 含完整示例/反例集（≥5+≥7） | ✅ 7 示例 + 10 反例 |
| 型号冲突消歧联合决策（S2-E）+ R4 主流程接入 | ✅ `_disambiguate_model` 在 `_parse_device_constraints` 中对 conflicts 逐一调用，`resolved_equiv_class`/`disambiguation` 写入输出契约；缺 pool_devices 且无显式 platform → AMBIGUOUS_MODEL |
| 平台别名优先级（完整名优先"海光3250"→乐研海光3250；简写"海光"歧义非首中，Minor） | ✅ `_expand_platform` 主键→aka 精确→歧义消歧→子串；PLATFORM_AMBIGUOUS / pool_devices 交集消歧 |
| nxp1043 等价类拆分（nxp1043-4g→160，nxp1043-8g→160-PRO/290/490/690，S2-D） | ✅ `equiv_class_map` 逐型号反查 |

## 3. 测试结果

- 测试文件：`skills/topo-planning/tests/test_limit_parser.py`
- 运行命令：`uv run --python 3.11 python -m pytest skills/topo-planning/tests/test_limit_parser.py`
- 结果：**41 passed**（0.12s）

覆盖：设备约束（platform/aka/all_scope/specific_node）、R4 主流程消歧（有/无 pool、用户优先）、平台别名优先级（完整名/简写歧义/池消歧）、TG 分支（sub_type/hw/combo/unknown）、链路 3 模式、speed 别名（10GE/万兆/40GE/25GE/10ge+copper/unknown）、端口兼容矩阵、空 limit、平台未知、冲突消歧（A1500-HU 三态/2900/多歧义）、nxp1043 拆分、复合、JSON 输入、非法 mode。

## 4. 偏离 LLD 的设计点

| 偏离点 | 说明 |
|--------|------|
| 支持 `device_type` 直接约束 | LLD §2 FR3 声明 device_constraints 含 device_type 字段但 §8.1b 伪代码仅处理 platform；实现支持 `device_type`/`device_types` 直接给定，使 R4 的"缺 pool_devices 且无显式 platform → AMBIGUOUS_MODEL"测试场景可构造 |
| `_disambiguate_alias_by_pool` 优先 hardware_platform 等价类交集 | LLD 伪代码用 `(dt_set & actual_dts) or (ec and ec in actual_hps)` OR 逻辑，会导致冲突型号（A1500-HU 同时属于两个系列）双命中；实现改为**优先 hardware_platform 等价类交集**，无 hw 字段时才退化为 device_type 交集，保证"海光"+pool(JL-HG)→杰伦海光唯一命中 |
| TG `_expand_tg_platform` 支持 `sub_type`/`hardware_platform` 字段直传 | 与 LLD §6.1 输入格式一致（`raw.get('platform') or raw.get('sub_type') or raw.get('hardware_platform')`） |
| `platform_alias.yaml` 落盘于 `config/`（LLD §4 新文件） | ✅ 已创建，主映射 15 个 alias 键（13 系列中 nxp1043 拆 3 键） |

## 5. 验证入口

- meta-qa：用 `limit-syntax.md` 全部示例/反例对 `parse_limit` 跑回归（示例应通过、反例应抛对应 code）
- 主映射 smoke：`parse_limit({"device_constraints":[{"platform":"海光3250"}]})` → 4 型号 HG3250
- S10 将审核 `limit-syntax.md` 示例/反例集完整性

## 6. 结论

**PASS** — CP6 编码完成检查通过。41 个单元测试全部通过，S2-A/B/C/D/E + R4 + Minor 全部契约满足。

等待 host-orchestrator 推进 Story 状态并拉起 meta-qa。
