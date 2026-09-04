---
story_id: "CR037-S2"
story_slug: "limit-parser"
cr_id: "CR-037"
title: "limit_parser.py（NLP→结构化 topo-limit + 平台别名展开 + 端口兼容矩阵 + 交付 limit 语法规范文档）"
priority: "P0"
wave: 1
status: "lld-ready-for-review"
depends_on: []
dependency_type: "none"
assignee: null
dev_context: "新建 skills/topo-planning/src/limit_parser.py，实现 topo-limit 结构化解析。接收 Agent 转译后的结构化 limit（dict），展开平台别名（消费 manaul device-reference.md 13 硬件系列→型号映射，P1-2），校验端口兼容矩阵（GE=copper|fiber，TE/XTE/QTE=fiber-only）。speed_class 别名归一：NLP「10GE/万兆/TE」→TE、「QTE/40GE」→QTE、「TTE/25GE」→TTE（S2-A）。TG 平台约束分支：node_type=TG 时 platform 映射到 sub_type/hardware_platform（ixia-c/trex × EP/C236/J1900），不套 DUT 型号表（S2-B）。链路约束字段名统一为 link（S2-C）。nxp1043 等价类拆分 nxp1043-4g(→160) vs nxp1043-8g(→160-PRO/290/490/690)（S2-D）。交付语法规范文档 skills/topo-planning/docs/limit-syntax.md（含示例/反例集）。多节点作用域 all 语义：node_ref 空→作用于全部该 node_type 节点。空 limit 默认行为：跳过约束匹配。型号冲突消歧（conflicts）与 hardware_platform 统一按等价类取值（HG3250/JL-HG 等，P1-3）；消歧为「用户平台约束等价类 + devices.yaml 实际设备」联合决策，非首中（S2-E），且**接入 parse_limit 主流程**（R4：parse_limit 新增 pool_devices 入参，冲突型号展开后逐一调 _disambiguate_model，resolved_equiv_class 写入 DeviceConstraint 输出契约；缺 pool_devices 且无显式 platform 抛 AMBIGUOUS_MODEL）。平台别名优先级（Minor）：完整平台名优先精确匹配（海光3250→乐研海光3250），简写海光多命中抛 PLATFORM_AMBIGUOUS 或按 pool_devices 交集消歧，非首中。"
validation_context: "limit 解析覆盖：设备约束（含多节点 all 语义）+ 3 种链路约束模式（include-at-least-one/all/specific）+ 平台别名展开（≥5 别名含海光3250）+ speed_class 别名归一（10GE/万兆/40GE/25GE）+ TG sub_type/hardware_platform 分支 + 空 limit 默认行为 + 端口兼容矩阵校验（非法组合报 PARAM_INVALID）+ nxp1043 等价类拆分 + 冲突型号联合消歧（A1500-HU/2900/A1200-FU/A1280-FU）+ R4 主流程消歧（parse_limit(pool_devices=...) 输出 resolved_equiv_class）+ Minor 别名优先级（海光3250→乐研海光3250，海光歧义→PLATFORM_AMBIGUOUS/pool 消歧）。语法规范文档含完整示例集与反例集。"
acceptance_criteria:
  - "parse_limit(limit_str) → TopoLimit 结构化对象（device_constraints + link_constraints）"
  - "平台别名展开：'海光3250'→[DAS-TGFW-A1300-HU, DAS-TGFW-A1500-HU, DAS-TGFW-A1580-HU, DAS-TGFW-A1600-HU]"
  - "speed_class 别名归一：NLP「10GE/万兆」→TE、「QTE/40GE」→QTE、「TTE/25GE」→TTE，归一后做兼容矩阵校验（S2-A）"
  - "TG 平台约束分支：node_type=TG 时 platform 映射到 sub_type（ixia-c/trex）与 hardware_platform（EP/C236/J1900），不套 DUT 型号表（S2-B）"
  - "链路约束字段名统一为 link（弃用 link_id），specific 模式用 link 指定（S2-C）"
  - "device_constraints.node_ref 空 → 作用于全部该 node_type 节点（all 语义）"
  - "link_constraints 3 模式：include-at-least-one（至少一条满足）/ all（全部满足）/ specific（指定 link 满足）"
  - "端口兼容矩阵：speed_class=GE 可匹配 copper 或 fiber；TE/XTE/QTE 仅 fiber；非法组合报 PARAM_INVALID"
  - "空 limit（None/''/{}）→ 返回空约束，不阻断后续匹配"
  - "limit-syntax.md 包含完整示例（设备约束/链路约束/多节点/空 limit/TG 分支）+ 显式反例集（≥7 个非法输入，含 10GE+copper 归一反例）"
  - "型号冲突消歧：A1500-HU/2900/A1200-FU/A1280-FU 逐一按「用户平台约束等价类 + devices.yaml 实际设备」联合决策，非首中；多平台并存 → AMBIGUOUS_MODEL（S2-E）"
  - "R4 主流程接入：parse_limit 新增 pool_devices 入参，冲突型号展开后逐一调 _disambiguate_model，DeviceConstraint.resolved_equiv_class/disambiguation 输出消歧结果；缺 pool_devices 且无显式 platform → AMBIGUOUS_MODEL"
  - "Minor 别名优先级：完整平台名优先精确匹配（海光3250→乐研海光3250）；简写海光多命中抛 PLATFORM_AMBIGUOUS 或按 pool_devices 交集消歧，非首中"
  - "nxp1043 等价类拆分：DAS-TGFW-160 → nxp1043-4g-无盘，160-PRO/290/490/690 → nxp1043-8g（S2-D）"
output_files:
  - "skills/topo-planning/src/limit_parser.py"
  - "skills/topo-planning/docs/limit-syntax.md"
file_ownership:
  primary:
    - "skills/topo-planning/src/limit_parser.py"
    - "skills/topo-planning/docs/limit-syntax.md"
  shared: []
  merge_owner: null
  forbidden: []
feature_design_refs:
  - "F-CR037"
lld_policy:
  required_level: "full-lld"
  trigger_reasons:
    - "NLP 边界与平台别名展开需完整设计"
    - "语法规范是显式交付物（G5）"
  rationale: "约束语义复杂，需覆盖设备/链路 3 模式 + 冲突消歧"
dev_gate: "cp5-approved"
design_evidence_ref: "process/stories/STORY-CR037-S2-LLD.md"
---

# STORY-CR037-S2：limit_parser.py

## 目标

新建 `skills/topo-planning/src/limit_parser.py`，将 Agent 转译后的结构化 topo-limit 展开为设备约束与链路约束；消费 manaul `device-reference.md` 完整映射表（13 硬件系列→型号 + 型号冲突消歧，等价类语义，P1-2/P1-3）；校验端口兼容矩阵。同时交付显式语法规范文档 `limit-syntax.md`。

## 依赖

无。

## 输出文件

| 文件 | 说明 |
|------|------|
| `skills/topo-planning/src/limit_parser.py` | 核心解析/展开/校验模块 |
| `skills/topo-planning/docs/limit-syntax.md` | 语法规范文档（含示例/反例集） |

## AI 任务清单

- [ ] TASK-S2-01：定义 `TopoLimit` / `DeviceConstraint` / `LinkConstraint` 数据类（含 TG sub_type/hardware_platform 字段、equiv_class_map，S2-B/S2-D）
- [ ] TASK-S2-02：实现 `parse_limit(limit_input, platform_map_path=None, pool_devices=None)` 入口函数（R4：pool_devices 为可选入参，传入消歧上下文）
- [ ] TASK-S2-03：实现平台别名展开（`_expand_platform(alias) → [device_type]`，消费 device-reference.md；含 equiv_class_group 反查，S2-D）
- [ ] TASK-S2-04：实现型号冲突消歧（`_disambiguate_model(device_type, pool_devices, user_equiv_class) → platform`，联合决策非首中，S2-E）+ **接入 parse_limit 主流程**（R4：pool_devices 入参 + resolved_equiv_class 输出）
- [ ] TASK-S2-05：实现 device_constraints 处理（含 node_ref 空→all 语义 + node_type 过滤 + TG 分支 `_expand_tg_platform`，S2-B；冲突型号消歧调用，R4）
- [ ] TASK-S2-06：实现 link_constraints 3 模式（include-at-least-one / all / specific，字段名 link，S2-C）
- [ ] TASK-S2-07：实现端口兼容矩阵校验（`_validate_port_compat(speed_class, media_type) → bool`）
- [ ] TASK-S2-08：实现空 limit 默认行为（返回空 TopoLimit）
- [ ] TASK-S2-09：交付 `limit-syntax.md`（含完整示例集 + 显式反例集 ≥7，含 speed_class 别名表/TG 分支/别名歧义说明）
- [ ] TASK-S2-10：加载 device-reference.md 映射表（内联常量或 YAML 数据文件，含 tg_platforms/speed_class_aliases/conflicts）
- [ ] TASK-S2-11：实现 speed_class 别名归一（`_normalize_speed_class(raw)`，10GE/万兆→TE、QTE/40GE→QTE、TTE/25GE→TTE，S2-A）
- [ ] TASK-S2-12：平台别名优先级修正（Minor：完整平台名优先精确匹配"海光3250"→乐研海光3250；简写"海光"多命中 → PLATFORM_AMBIGUOUS 或 pool_devices 交集消歧，非首中）

## 设计证据

见 `process/stories/STORY-CR037-S2-LLD.md`（14 章节 full-lld）。
