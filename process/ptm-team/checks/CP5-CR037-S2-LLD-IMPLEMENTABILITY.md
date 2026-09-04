---
doc_type: cp_check_result
id: CP5-CR037-S2-LLD-IMPLEMENTABILITY
story_id: "CR037-S2"
cr_id: "CR-037"
stage: CP5
checkpoint_type: auto
status: PASS
created_at: "2026-08-05"
owner: "meta-dev"
evidence_type: "full-lld"
evidence_path: "process/stories/STORY-CR037-S2-LLD.md"
---

# CP5-CR037-S2：设计证据可实现性自动预检

| 条目 | 结果 |
|------|:---:|
| lld_policy = full-lld 匹配 | PASS |
| 14 章节完整（0-15） | PASS |
| Goal 明确可量化 | PASS |
| Requirements 9 FR + 3 NFR 覆盖 | PASS |
| 模块拆分与职责清晰（parse_limit + 7 子函数 + platform_alias 数据层） | PASS |
| 代码结构与文件影响范围明确（3 新增 + 1 只读消费） | PASS |
| 数据模型定义完整（TopoLimit/DeviceConstraint/LinkConstraint + platform_alias.yaml + 端口兼容矩阵） | PASS |
| API/Interface 契约完整（parse_limit 3 输入态 + _expand_platform 3 层匹配 + _disambiguate_model） | PASS |
| 核心处理流程有流程图 + 约束叠加优先级（H4） + 多节点 all 语义 | PASS |
| 技术细节（别名展开 3 层算法 + 端口兼容矩阵 + 冲突消歧） | PASS |
| 安全与性能分析（JSON 校验 + 无凭据 + 懒加载 <10ms） | PASS |
| 测试设计（19 用例 + 覆盖平台别名/3 模式/兼容矩阵/空 limit/型号冲突） | PASS |
| 实施步骤 10 TASK + 13h 预估 | PASS |
| 风险 4 项 + 缓解 | PASS |
| 实现灰区 3 项 + 取舍记录（platform_alias 独立/多 DUT all 语义/NLP 转译边界） | PASS |
| 回滚与发布策略 | PASS |
| DoD 11 项可验证 | PASS |
| 输出文件与 DEVELOPMENT-PLAN 一致（limit_parser.py + limit-syntax.md） | PASS |
| 文件所有权无冲突 | PASS |
| 依赖 S2=[] 满足 | PASS |
| tier="A" 合理 | PASS |
| open_items 无 | PASS |
| HLD §4.2 对齐 | PASS |
| platform_alias.yaml 覆盖 13 硬件系列（P1-2）+ 等价类语义（P1-3）+ 主键规范系列名（P2-3） | PASS |
| limit-syntax.md 设计已明确（≥5 示例 + ≥5 反例） | PASS |
| lld-check: LLD Structure Check | OK |

## 开放项

无。

## 修订说明（2026-08-05 设计审查修订）

- **P1-2**：硬件系列数 14→13（device-reference.md 实际 13 个系列）；§5.2/§8.5/§11/§12/§15 同步。
- **P1-3**：`_disambiguate_model` 与 S3 `_resolve_hardware_platform` 统一 `hardware_platform` 取值语义为**等价类**（HG3250/JL-HG/HD-D2000/D2000/nxp1046-16g/C3758 等），`conflicts` 映射到等价类而非系列名；aliases 新增 `equiv_class` 字段。
- **P2-3**：`platform_alias.yaml` 主键改用规范系列名（如「乐研海光3250」），`also_known_as` 收别名（海光3250/HG3250）；`海光3250`/`海光5380` 不再是独立键。

## 修订说明（2026-08-05 CP5 综合审核 NEEDS_REWORK 打回，v0.4）

- **S2-A**：speed_class 别名归一——NLP「10GE / 万兆 / TE」→ 枚举 `TE`、「QTE / 40GE」→ `QTE`、「TTE / 25GE」→ `TTE`；新增 `speed_class_aliases` 区段 + `_normalize_speed_class`（§5.2/§6.5），语法规范 + 单测覆盖（§8.5/§10/§15）。修复「10GE 不在枚举」威胁核心场景的缺陷。
- **S2-B**：TG 平台约束独立分支——`node_type: TG` 时 platform 映射到 `sub_type`/`hardware_platform`（ixia-c/trex × EP/C236/J1900，6 组合），新增 `tg_platforms` 区段 + `_expand_tg_platform`（§3/§5.1/§6.4），不套 DUT 型号映射表。
- **S2-C**：字段名统一为 HLD 的 `link`（弃用 `link_id`），与 collection links 命名一致（§5.1/§6.1/§7/§8.5）。
- **S2-D**：nxp1043 等价类拆分——`nxp1043-4g`(→DAS-TGFW-160，4g-无盘) vs `nxp1043-8g`(→160-PRO/290/490/690)；`nxp1043` 家族键带 `equiv_class_group` 反查，`_expand_platform` 返回 `equiv_class_map`（§5.2/§6.2/§8.1）。
- **S2-E**：型号冲突消歧改为「用户平台约束等价类 + devices.yaml 实际设备」联合决策，非首中；conflicts（2900/A1200-FU/A1280-FU/A1500-HU）逐一给消歧策略（§6.3/§8.3）。
- **S2-F**：保持多节点 all 语义（H3）、空 limit 默认（O-a）、端口兼容矩阵（GE=copper|fiber，TE/XTE/QTE=fiber-only）。
- lld-check（v0.4）：**OK（PASS）**；单元测试 19→32 用例（新增 speed 别名/TG 分支/nxp1043 拆分/联合消歧）。

## 修订说明（2026-08-05 CP5 第二轮复审 R4 + Minor，v0.5）

- **R4（MAJOR）消歧算法未接入主流程**：`_disambiguate_model` 此前有设计但 `parse_limit` 未调用。修正：`parse_limit` 新增 `pool_devices` 入参（S3 归并后 devices.yaml 实际设备列表，可选）；`_parse_device_constraints` 展开后对 conflicts 中的 device_type 逐一调 `_disambiguate_model`，结果写入 `DeviceConstraint.resolved_equiv_class` + `disambiguation`，并更新 `equiv_class_map[device_type]`——输出契约含消歧后 device_type 集合。`pool_devices=None` 且用户未显式区分系列 → 抛 `AMBIGUOUS_MODEL`（非首中）。§1/§2（FR8 + NFR3 确定性依赖）/§3/§5.1（新增字段）/§6.1（签名 + PLATFORM_AMBIGUOUS 异常）/§6.3（主流程接入说明）/§7 流程图/§8.1b（调用链代码）/§10（`test_disambiguate_in_main_flow*`）/§11/§12/§13 灰区7/§15 DoD 全部同步。
- **Minor 平台别名优先级（"海光"简写首中杰伦海光）**：`_expand_platform` 修正为完整平台名优先精确匹配（"海光3250"→乐研海光3250），简写"海光"多命中（杰伦海光 JL-HG / 乐研海光3250 HG3250）抛 `PLATFORM_AMBIGUOUS` 或按 pool_devices 实际设备交集消歧，非首中。§5.2 注释/§6.2（查找优先级 2/3 步）/§8.1 算法/§10（`test_platform_alias_priority` / `test_platform_short_alias_ambiguous` / `test_platform_short_alias_disambiguated_by_pool`）/§12/§13 灰区8/§15 DoD 同步。
- 单元测试 32→40 用例（新增 R4 三例 + Minor 三例 + 1 例 `test_disambiguate_in_main_flow_no_pool`）；fixture 2→3（新增 `devices_pool_ambiguous.yaml`）。
- lld-check（v0.5）：**OK（PASS）**。
